# Design Decisions for mailfruit

Also check out [some threat models](threat_models.md) this package attempts to address.

This installation will not accept email from unencrypted sources (sending servers must issue STARTTLS.) In the age of LetsEncrypt there is little reason for a sending server not to have a TLS certificate (though funnily enough this does actually block a local bank's mail server from sending email.)

This installation uses the Ansible keyword "latest" to manage package versions. It also installs the `unattended-upgrades` package to keep software up to date. This is a security measure to help mitigate the constant stream of CVEs. This means that software can and will break from time to time when upstream (Debian) bumps versions. We rely on upstream for reasonable upgrade paths and trust their judgement (love you guys.) The administrator is responsible for preventing or fixing broken services; send a PR if you have bugfixes you would like to merge! This potential for downtime is a compelling reason to implement monitoring in the future.

This installation can download software over Tor. This includes nearly every package from apt (with the exception of initial package upgrades, apt-transport-tor and tor itself) but in the future will include SUBMISSION, IMAP and SSH protocol hidden services. This is both to enable very secure communications with the server as well as prevent fingerprinting of this as a server running TREES and this Mailfruit installation. Initial provisioning using Tor takes a nontrivial amount of time (hours if you're using a bad circuit), though subsequent runs should be much faster. The default debug configuration does not use Tor.

*If it is not safe to use Tor in your country, please modify your `host_vars`! Set the `use_tor` variable to 'false', or point `tor_proxy_url` to a custom value and set `manage_tor` to 'false'!* To use a [bridge](https://bridges.torproject.org), set `tor_proxy_url` to point to a configured Tor server, and set `manage_tor` to 'false' to prevent managing Tor on the mailserver box.

This installation targets Debian because it's much easier to target one distribution. We also believe in free software. Feel free to fork and modify for commericial distros.

This installation does not yet split out the MTA and MDA roles, which would enable higher availability and better performance. There are multiple things involved in configuring that - using a mail delivery mechanism that is not a local client and setting up proper user querying over network between Postfix & Dovecot are the big ones. This is a work in progress, though the level of effort here is high. This installation targets small to medium sized mail servers; benchmarking has not been done, but it's suspected the encryption used is the biggest bottleneck, using a significant amount of CPU; a second guess would be the user database living in SQLite. PRs are welcome here.
