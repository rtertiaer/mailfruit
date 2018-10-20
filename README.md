# mailfruit

This playbook deploys mailfruit, which aims to simplify deployment of a small secure mail server by systems administrators. This Ansible playbook installs and configures:
- [TREES from riseuplabs](https://0xacab.org/riseuplabs/trees)
- [Dovecot](https://en.wikipedia.org/wiki/Dovecot_(software))
- [Postfix](https://en.wikipedia.org/wiki/Postfix_(software))
- [sqlite3](https://en.wikipedia.org/wiki/SQLite) user database
- [Fail2ban](https://www.fail2ban.org/wiki/index.php/Main_Page)
- SSL certs from [Let's Encrypt](https://letsencrypt.org)
- Local backups with [duplicity](http://duplicity.nongnu.org)
on a single host.

## How to use me
You can run temporary test instances of this codebase with Vagrant; simply install Vagrant and run `vagrant up`.

You can also install this on a standalone server for production use:
- Start a new Debian 9 (Stretch) server (tested most recently on 9.5.)
  - We recommend setting up an encrypted root volume.
  - If you're paranoid, we recommend avoiding rented servers from cloud providers like AWS and DigitalOcean, who have the ability to peer into your server's RAM and negate all the following security measures. If you must use a third party to host your internet facing ports, consider using something like HAProxy to forward your traffic through a VPN or Tor to hardware you do control.
- Set up DNS records for your host: an A record for its hostname, and MX records for all domains for which you want to receive mail.
- Set up your host in `hosts` as an mta_server and mda_server. An example `hosts` file (replace 127.0.0.1 with your server's IP or DNS name):
```
[mta_servers]
127.0.0.1

[mda_servers]
127.0.0.1

[all:children]
mta_servers
mda_servers
```
- Copy the file [host_vars/defaults](/host_vars/defaults) to a new `host_vars/$hostname` and modify its variables.
  - `server_hostname` is the actual hostname of your new Debian 9 server. This is used to request a LetsEncrypt certificate for non-`debug` installations as well as configure mail services.
  - `mail_domain` is the default domain for which this server will accept email; this value shows up in, for example, SMTP and maybe IMAP connections. It does not actually modify what domains for which the server accepts mail. This installation accepts mail for any user in the user database; for more details, inspect the contents of /usr/local/bin/add_user.sh. This could alternatively be defined in `group_vars/all`.
  - `admin_email` is the email address of the admin. This is used to send informational messages to the administrator about server health (right now, just Fail2Ban.) It is most sensible to have this email address located on another mail service, but I won't judge you if you use an email on $localhost. This could alternatively be defined in `group_vars/all`.
  - `use_tor` can take values 'true' or 'false' and toggles the use of Tor services for Git, apt and hidden services. Using Tor for these takes a nontrivial amount of time - during the initial provision and with a poor circuit, this can take hours, though subsequent runs should be much quicker.
  - `manage_tor` can take values 'true' or 'false' and toggles whether this playbook should attempt to install and manage tor and apt-transport-tor. If this is set to 'false', you can still use Tor by manually configuring a server and setting the `tor_proxy_url` variable.
  - `tor_proxy_url` is a URL of the SOCKS proxy to use, like `socks5h://127.0.0.1:9050`. Setting this to a custom value and setting `manage_tor` to 'false' allows you to configure bridges or exit the network through a particular host. Right now, if this is not set to `socks5h://127.0.0.1:9050` mailfruit will not use Tor for downloading packages with apt - we must configure apt to use this proxy as well and it defaults to localhost.
- Run this playbook against your new Debian 9 server. Example: `ansible-playbook -u root site.yml`
- Run the helper script `/usr/local/bin/add_user.sh` on the mail server to add users.
  - Better tooling here is necessary.
  - Note that this may leave a user's password in cleartext in `.bash_history`; consider running this script on something other than the Dovecot server.
- Set up a cronjob to rsync the /var/backups folder to a secure offsite location.

## Design decisions
This installation will not accept email from unencrypted sources (sending servers must issue STARTTLS.) In the age of LetsEncrypt there is little reason for a sending server not to have a TLS certificate (though funnily enough this does actually block a local bank's mail server from sending email.)

This installation uses the Ansible keyword "latest" to manage package versions. It also installs the `unattended-upgrades` package to keep software up to date. This is a security measure to help mitigate the constant stream of CVEs. This means that software can and will break from time to time when upstream (Debian) bumps versions. We rely on upstream for reasonable upgrade paths and trust their judgement (love you guys.) The administrator is responsible for preventing or fixing broken services; send a PR if you have bugfixes you would like to merge! This potential for downtime is a compelling reason to implement monitoring in the future.

This installation can download software over Tor. This includes nearly every package from apt (with the exception of initial package upgrades, apt-transport-tor and tor itself) but in the future will include SUBMISSION, IMAP and SSH protocol hidden services. This is both to enable very secure communications with the server as well as prevent fingerprinting of this as a server running TREES and this Mailfruit installation. Initial provisioning using Tor takes a nontrivial amount of time (hours if you're using a bad circuit), though subsequent runs should be much faster. The default debug configuration does not use Tor.

*If it is not safe to use Tor in your country, please modify your `host_vars`! Set the `use_tor` variable to 'false', or point `tor_proxy_url` to a custom value and set `manage_tor` to 'false'!* To use a [bridge](https://bridges.torproject.org), set `tor_proxy_url` to point to a configured Tor server, and set `manage_tor` to 'false' to prevent managing Tor on the mailserver box.

This installation targets Debian because it's much easier to target one distribution. We also believe in free software. Feel free to fork and modify for commericial distros.

This installation does not yet split out the MTA and MDA roles, which would enable higher availability and better performance. There are multiple things involved in configuring that - using a mail delivery mechanism that is not a local client and setting up proper user querying over network between Postfix & Dovecot are the big ones. This is a work in progress, though the level of effort here is high. This installation targets small to medium sized mail servers; benchmarking has not been done, but it's suspected the encryption used is the biggest bottleneck, using a significant amount of CPU; a second guess would be the user database living in SQLite. PRs are welcome here.

## TODO
This is a work in progress. Roadmap:
- (more) Tor hidden service integrations (IMAP, SUBMISSION, SSH)
- AppArmor profiles
- Harden SSH according to [these recommendations](https://github.com/stribika/stribika.github.io/blob/master/_posts/2015-01-04-secure-secure-shell.md)
- Monitoring
- Split out mta & mda roles for scalability
- Webmail (preferrably without Javascript; how can we do this while treating the password as super sensitive?)
- Encrypted volume (is this outside scope for this playbook?)
- Encrypted backups - GPG key + PAR file integrations with duplicity
- HA - multiple servers of each role, craft each box to be ephemeral
- Better tooling for adding users (very simple script now)
- Optional RBL (pyzor?)
