# mailfruit

This role deploys a reasonably secure small mail server stack. This Ansible playbook installs and configures:
- [TREES from riseuplabs](https://0xacab.org/liberate/trees)
- [Dovecot](https://en.wikipedia.org/wiki/Dovecot_(software))
- [Postfix](https://en.wikipedia.org/wiki/Postfix_(software))
- [OpenDKIM](https://www.opendkim.org/)
- [sqlite3](https://en.wikipedia.org/wiki/SQLite) user database
- SSL certs from [Let's Encrypt](https://letsencrypt.org)
on a single host.

## Requirements
* Debian 12+
* The hostname in `mailfruit_server_hostname` pointing to the server

## Role Variables
* `mailfruit_server_hostname` - *required*
* `mailfruit_mail_domains` - *required* a list of domains this server can accept mail for. The first item in this list is used as a default for various things.
* `mailfruit_admin_email` - *required*
* `mailfruit_certbot_authenticator` - _optional_, default: `standalone`
* `mailfruit_trees_git_ref` - _optional_, default: `master`

## Example Playbook

```
- hosts: mail_servers
  roles:
    - { role: rtasson.mailfruit, mailfruit_server_hostname: "big.blah.com" }
```

## Some notes
I've opted to require TLS at every step with pre-wrapped ports - ie, using port 993 for IMAPS instead of 143, where TLS is negotiated within a cleartext connection. Not all clients like this, but it can prevent attacks where the TLS negotiation is stripped out of the cleartext.

## License
GPLv3

## Post-deployment
*Important*: For each server you deploy this to, you _must_ host the DKIM `TXT` record for each domain. This record can be found on each server at `/etc/opendkim/keys/mail.txt`. You must also configure your SPF/DMARC records. If you do not do this, you will have very poor email deliverability.

You should probably use something like fail2ban to prevent account harvesting & break-in attempts. You should almost certainly harden your SSH install. You should definitely take backups of this server, particularly the user database; without it, the mail files become unreadable. You should implement external monitoring of this server; in particular, if this monitoring sends emails to alert you, those email addresses shouldn't reside on this server ;)
