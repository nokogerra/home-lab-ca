# Home lab root CA
> **Do not change the ansible inventory structure, group names must remain the same. Groups are used for delegate_to in some tasks**. Actually, inventory group names are used only in "sign_certificate.yml" and "generate_root_ca_crl.yml" tasks of "sub_ca" role and, obviously, in playbooks. So, you can change the inventory group names, but in that case you have to change those tasks accordingly.
> Just put root CA FQDN or IP into root_ca group.

This is the first role of the home-lab-ca bundle and it installs root CA based on openssl.<br />
There must be a user **with primary group** on a target OS, which will be a CA administrator, **it should be used as an ansible user**, it also must be a **sudoer**. The play, which installs root CA, must be executed with sudo (root) privileges, all file/dir permissions are configured appropriately for a CA administrator during execution.<br />
**It is assumed all the further CA operations, like certificate signing/revocation operations, CRL generation, etc, should be executed by a CA administrator without root privileges, it means "become:false" in ansible.**<br />
Example:
```
root@s1-root-ca-01:~# cat /etc/passwd | grep "^ca"
ca:x:1001:1001:ca,,,:/home/ca:/bin/bash

root@s1-root-ca-01:~$ cat /etc/group | grep "^ca"
ca:x:1001:

root@s1-root-ca-01:~# cat /etc/sudoers.d/90-cloud-init-users | grep "^ca"
ca ALL=(ALL) NOPASSWD:ALL
```
I use ssh key pair to authenticate as a "ca" user on a remote host.
### Process overview
1. Make root CA directory structure (/ca/certs, /ca/db/, /ca/private, /ca/csrs) with ca_user owner and appropriate permissions;
2. Make crlnumber, serial and index files in /ca/db;
3. Make openssl.cnf based on openssl.cnf.j2;
4. Generate root CA CSR and private key files with ca_user owner and appropriate permissions;
5. Selfsign (openssl ca selfsign) a root CA certificate and fix permissions for the file;
6. Generate a CRL ca_user owner and appropriate permissions;
7. Install apache2, root CA CRL and CRT will be published via this web-server;
8. Add ca_user to www-data group, so he will be able to publish CRT and CRL in /var/www/html (via symlinks); add www-data user to ca_group, so this user will be able to read /ca/root_ca.crl and /ca/certs/root_ca.crt; restart apache2 service;
9. Make a cron job to generate CRL periodically.
### Openssl policy
Make sure you use the same **organizationName** and **countryName** in all further certificate operations, because these options are enforced:
```
[policy_c_o_match]
countryName = match
stateOrProvinceName = optional
organizationName = match
organizationalUnitName = optional
commonName = supplied
emailAddress = optional
```
Otherwise, change them to 'optional' in openssl.cnf.j2.
### Variables
```
# root CA administrator and its primary group:
root_ca_user: "ca"
root_ca_group: "ca"

# root CA name (machine hostname)
root_ca_name: "s1-root-ca-01"

# root CA domain suffix, must be your domain
root_ca_domain: "nokogerra.lab"

# CRT and CRL will be published at 
# http://{{ root_ca_name }}.{{ root_ca_domain }}/{{ root_ca_name }}.crt
# and
# http://{{ root_ca_name }}.{{ root_ca_domain }}/{{ root_ca_name }}.crt
# it is configured accordingly in the openssl.cnf
# so, the name must be resolvable (s1-root-ca-01.nokogerra.lab in my case)

# CA country
root_ca_country: "RU"

# CA organization 
root_ca_organization: "NOKOGERRA-LAB"

# root CA certificate common name
root_ca_cn: "s1-root-ca-01"

# root CA certificate validity period
root_ca_cert_days: "3650"

# root CA CRL validity period
root_ca_crl_days: "30"

# root CA private key length
root_ca_key_bits: "4096"

# root CA private key encryption, valid values: "yes" or "no"
# in case "yes" is specified, you will be prompted
# to enter a passphrase during the playbook execution,
# the passphrase will be shown in an ansible log
root_ca_encrypt_key: "yes"

# name constraints (soft!), means for which names and IPs
# this root CA will be able to sign certificates (basically sub CA names)
# your home lab domain is recommended
root_ca_certs_name_constraints:
  permitted:
    - "DNS.0=nokogerra.lab"
    - "DNS.1=nokogerra2.lab"
  excluded:
    - "IP.0=0.0.0.0/0.0.0.0"
    - "IP.1=0:0:0:0:0:0:0:0/0:0:0:0:0:0:0:0"

# Generate CRL cron job scheduler (runs once a day in this example):
root_ca_gen_crl_scheduler:
  minute: "0"
  hour: "1"
  day: "*"
  month: "*"
```
Take a look at **tests**.
