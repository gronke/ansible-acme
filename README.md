[![Ansible Galaxy](https://img.shields.io/badge/ansible--galaxy-letsencrypt-blue.svg?style=flat-square)](https://galaxy.ansible.com/jaywink/letsencrypt)
[![License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](https://tldrlegal.com/license/mit-license)

## Ansible LetsEncrypt

A role to automate LetsEncrypt certificates.

Stability: beta.

### What does it do?

This role will pull in the [simp_le](https://github.com/kuba/simp_le) that is recommended from LetsEncrypt for automation. After setting up the client the role will try to obtain a certificate for the provided domain.

Functionality as follows:
* One domain per role include only
* Runs the client as unprivileged user
* Certificates are only renewed when they expire within a definable threshold

PR's are welcome to include more functionality.

#### More detail

* The client will be installed in `/opt/letsencrypt` as root
* A list of services to be stopped before and (re-)started after obtaining a new certificate can be configured using the variable `letsencrypt_pause_services`.
* `certonly` mode is used, which means no automatic web server installation
* After cert issuing, you can find it in `/etc/letsencrypt/live/<domainname>`
   * Tip, use this in your Apache2 config, for example, in your main role. Just make sure not to try and start Apache2 with the virtualhost active without the LetsEncrypt role running first!

       ```
       SSLCertificateFile /etc/letsencrypt/live/{{ hostname }}/cert.pem
       SSLCertificateKeyFile /etc/letsencrypt/live/{{ hostname }}/privkey.pem
       SSLCertificateChainFile /etc/letsencrypt/live/{{ hostname }}/chain.pem
       ```

* If the cert has been requested before, this role will automatically try to renew it, if possible. Disable this functionality by setting `letsencrypt_force_renew` to `false`. No renewal will be attempted in this case if cert is not due for renewal.
* A `www.` subdomain will automatically be requested along with the certificate.
    * To disable this behaviour, set `letsencrypt_request_www` to `false` in your vars.

### Requirements

Tested with

* Ubuntu 14.04 and Debian 8
* Apache2 and Nginx
* Ansible 1.9 / 2.x

### Role Variables

#### Required

* `letsencrypt_domain` - Domain the certificate is for.
* `letsencrypt_email` - Your email as certificate owner.
* `letsencrypt_tos_sha256` - Provide the SHA256 hash of the latest Terms-of-Service you agreed to

#### Optional

* `letsencrypt_args` - Additional command line args to simp_le LetsEncrypt client.
* `letsencrypt_force_renew` - Whether to attempt renewal always, default to `true`.
* `letsencrypt_pause_services` - List of services to stop/start while calling Certbot.
* `letsencrypt_request_www` - Request `www.` automatically (default `true`).

### Terms of Service

On the [Let's Encrypt: Policy and Legal Repository](https://letsencrypt.org/repository/) page the latest "Terms of Service" PDF document can be downloaded. Make sure you agree to this document before you calculate the checksum and provide it as role input variable.

```bash
export TOS_DOCUMENT=LE-SA-v1.1.1-August-1-2016.pdf
wget https://letsencrypt.org/documents/$TOS_DOCUMENT
shasum -a 256 $TOS_DOCUMENT
```

At writing time the latest tos_sha256 is `6373439b9f29d67a5cd4d18cbc7f264809342dbf21cb2ba2fc7588df987a6221`

### Example Playbook

This role works best when included just before your main site role, for example. Or it can be used in an individual playbook, for example as below.

If not manually specified the role will create system user and group "letsencrypt" that executes the client

    ---
    - hosts: myhost
      roles:
        - role: ansible-letsencrypt
          letsencrypt_user: "letsencrypt"
          letsencrypt_email: email@example.com
          letsencrypt_domain: example.com
          letsencrypt_pause_services:
            - apache2

### License

MIT

### Author Information

Jason Robinson (@jaywink) - mail@jasonrobinson.me - https://iliketoast.net/u/jaywink - https://twitter.com/jaywink

Special thanks to Stefan Grönke (@gronke) for his work on expanding this role.

See CONTRIBUTORS for a full list of contributors.
