# Ansible ACME Challenge

This role was forked from [jaywink.letsencrypt](https://github.com/jaywink/letsencrypt) with the purpose to use native Ansible features rather than leaving the certification steps up to the (official LetsEncrypt) Certbot ACME client.

Ansible version required: 2.x

## What does it do?

This role provides the ability to maintain and generate SSL certificates that were automatically be signed by a third party certificate authority service (like LetsEncrypt) that validates domains via the ACME-Challenge-Response method.

The current implementation is using [simp_le](https://github.com/kuba/simp_le) as ACME client, as it is not intended as all-in-one solution like the LetsEncrypt certbot client. Removing the dependency on any third-party client is still an important milestone of this project.


## System Requirements

### Operating System
- Ubuntu >= 14.04
- Debian >= 7 (Jessie)
- FreeBSD >= 10

### Service Integration
- Nginx
- Apache 1.9 / 2

## How does the certification work

ACME specifies several methods to validate the ownership of a domain. The most common one (that is also supported by the certbot client) is HTTP.

After generating a private key, a SSL certificate and a CSR (certificate signing request), the ACME client negotiates a challenge response with the CA (e.g. LetsEncrypt). This response needs to be served from the path `/.well-known/acme-challenge` on the same vhost as the certificate to be signed.

## Role Variables

| Name | Required | Description |
| ---- | -------- | ----------- |
| acme_email | **yes** | This is the certificate admins email - make sure to set it to yours! |
| acme_domain | **yes** | The domain name that is going to be signed |
| acme_tos_sha256 | **yes** | SHA256 Hash of the LetsEncrypt Terms of Service |
| acme_staging | no | Boolean value that enables generating fake certificates for testing purposes (Default: `false`) |
| acme_verbose | no | Turns on verbose logging (Default: `false`) |
| acme_pause_services | no | List of services that need to be shut down while  |
| acme_force_renew | no | This boolean can enforce re-certification of existing certificates independent of their expiry date |
| acme_expiration_threshold | no | Amount of seconds to expiration before a certificate gets renewed (Default: `30 days`) |
| acme_user | no | The user that generates the certificates (Default: `acme`) |
| acme_group | no | The primary group of the user generating the certificates analog acme_user (Default: `{{acme_user}}`) |
| acme_export_dir | no | The storage location of the key and certificate maintained by this role. Defaults to the same default directory as certbot (Default: `/etc/letsencrypt/live/{{acme_domain}}`) |
| acme_challenge_url | no | The service URL to trigger the ACME challenge. Overrides settings from `acme_staging` (Default: `https://acme-v01.api.letsencrypt.org/directory`)

### Role Variables

#### Required

* `letsencrypt_domain` - Domain the certificate is for.
* `letsencrypt_email` - Your email as certificate owner.

#### Optional

* `letsencrypt_certbot_args` - Additional command line args to be passed to Certbot-- will be combined with `letsencrypt_certbot_default_args`. See [the Certbot docs](https://certbot.eff.org/docs/using.html) for arguments you may pass.
* `letsencrypt_certbot_default_args` - Please see `defaults/main.yml` what the default arguments are. Also, you could add To override all the arguments to Certbot, for example to use another plugin, set them using this variable.
* `letsencrypt_certbot_verbose` - Make Certbot output to console (default `true`).
* `letsencrypt_certbot_version` - Set specific Certbot version, for example a git tag or branch. Note that the lowest version of Certbot we support is 0.6.0.
* `letsencrypt_force_renew` - Whether to attempt renewal always, default to `true`.
* `letsencrypt_paus`e_services` - List of services to stop/start while calling Certbot.
* `letsencrypt_request_www` - Request `www.` automatically (default `true`).

### Example Playbook

This role works best when included just before your main site role, for example. Or it can be used in an individual playbook, for example as below.

This role should become root on the target host.

```yaml
---
- hosts: myhost
  become: yes
  become_user: root
  roles:
    - role: gronke.acme
      acme_email: email@example.com
      acme_domain: example.com
      acme_pause_services:
        - apache2
```

The according Apache2 SSL configuration would look like:

```
SSLCertificateFile /etc/letsencrypt/live/{{ letsencrypt_domain }}/cert.pem
SSLCertificateKeyFile /etc/letsencrypt/live/{{ letsencrypt_domain }}/privkey.pem
SSLCertificateChainFile /etc/letsencrypt/live/{{ letsencrypt_domain }}/chain.pem
```

### Tested on

* Ubuntu 14.04 and Debian 8
* Apache2 and Nginx
* Ansible 2.x

### License

MIT

### Author Information

The project was created by Jason Robinson (@jaywink) to generate and maintain LetsEncrypt certificates by utilizing the official certbot client. This fork aims to work without dependency to third-party ACME clients and follows the principle of priviledge separation. After contributing on the original repository, Stefan Grönke (@gronke) forked the project to coexist with the certbot-driven approach.

See CONTRIBUTORS for a full list of contributors.
