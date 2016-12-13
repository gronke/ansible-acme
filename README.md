# Ansible ACME Challenge

This role was forked from [jaywink.letsencrypt](https://github.com/jaywink/letsencrypt) with the purpose to use native Ansible features rather than leaving the certification steps up to the (official LetsEncrypt) Certbot ACME client.

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


### License

MIT

### Author Information

The project was created by Jason Robinson (@jaywink) to generate and maintain LetsEncrypt certificates by utilizing the official certbot client. This fork aims to work without dependency to third-party ACME clients and follows the principle of priviledge separation. After contributing on the original repository, Stefan Grönke (@gronke) forked the project to coexist with the certbot-driven approach.

See CONTRIBUTORS for a full list of contributors.
