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

## More reading
* [Threat models](docs/threat_models.md)
* [Design decisions](docs/design_decisions.md)

## TODO
This is a work in progress. Roadmap:
- Make this into a regular Ansible role, ie suitable for the Galaxy
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
