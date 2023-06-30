# Home lab sub CA
> **Do not change the ansible inventory structure, group names must remain the same. Groups are used for "delegate_to" in some tasks**
> Just put root CA and sub CA FQDNs or IPs into root_ca and sub_ca groups accodringly.
> Otherwise, you should search/replace all entries of the ansible group names in all home-lab-ca roles.<br />
This is the second role of the home-lab-ca bundle and it installs sub CA based on openssl.<br />
There must be a user **with primary group** on a target OS, which will be a CA administrator, **it should be be used as an ansible user**, it also must be a **sudoer**. The play, which installs sub CA, must be executed with sudo (root) privileges, all file/dir permissions are configured appropriately for a CA administrator during execution.<br />
**It is assumed all the further CA operations, like certificate signing/revocation operations, CRL generation, etc, should be executed by a CA administrator without root privileges, it means "become:false" in ansible.**<br />
Example:
```
ca@s1-sub-ca-01:~$ cat /etc/passwd | grep "^ca"
ca:x:1001:1001:ca,,,:/home/ca:/bin/bash

ca@s1-sub-ca-01:~$ cat /etc/group | grep "^ca"
ca:x:1001:

ca@s1-sub-ca-01:~$ sudo cat /etc/sudoers.d/90-cloud-init-users | grep "^ca"
ca ALL=(ALL) NOPASSWD:ALL
```
I use ssh key pair to authenticate as a "ca" user on a remote host.
### Process overview
1. Make sub CA directory structure (/ca/certs, /ca/db/, /ca/private, /ca/csrs) with ca_user owner and appropriate permissions;
2. Make crlnumber, serial and index files in /ca/db;
3. Make openssl.cnf based on openssl.cnf.j2;
4. Generate sub CA CSR and private key files with ca_user owner and appropriate permissions;
5. Fetch sub CA CSR to a localhost, copy it to a root CA, sign a sub CA certificate, fetch the sub CA certificate to a localhost, copy it to the sub CA host;
6. Generate root and sub CAs CRLs;
7. Install apache2, sub CA CRL and CRT will be published via this web-server;
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
Otherwise, change them to 'optional'.
### Variables
```
# sub CA administrator and his primary group:
sub_ca_user: "ca"
sub_ca_group: "ca"

# sub CA name, must be a real hostname
sub_ca_name: "s1-root-ca-01"

# sub CA domain suffix, must be your domain
sub_ca_domain: "nokogerra.lab"

# CRT and CRL will be published at 
# http://{{ sub_ca_name }}.{{ sub_ca_domain }}/{{ sub_ca_name }}.crt
# and
# http://{{ sub_ca_name }}.{{ sub_ca_domain }}/{{ sub_ca_name }}.crt
# it is configured accordingly in the openssl.cnf
# so, the name must be resolvable (s1-sub-ca-01.nokogerra.lab in my case)

# CA country
sub_ca_country: "RU"

# CA organization
sub_ca_organization: "NOKOGERRA-LAB"

# sub CA certificate common name
sub_ca_cn: "s1-sub-ca-01"

# sub CA certificate validity period
sub_ca_cert_days: "1095"

# sub CA CRL validity period
sub_ca_crl_days: "7"

# sub CA private key length (will be user in openssl.cnf)
sub_ca_key_bits: "2048"

# sub CA private key encryption, valid values: "yes" or "no"
# in case "yes" is specified, you will be prompted
# to enter a passphrase during the playbook execution,
# the passphrase will be shown in an ansible log
sub_ca_encrypt_key: "yes"

# name constraints (soft!), means for which names and IPs
# this sub CA will be able to sign certificates
# your home lab domain is recommended
sub_ca_certs_name_constraints:
  permitted:
    - "DNS.0=nokogerra.lab"
    - "DNS.1=nokogerra2.lab"
  excluded:
    - "IP.0=0.0.0.0/0.0.0.0"
    - "IP.1=0:0:0:0:0:0:0:0/0:0:0:0:0:0:0:0"

# Generate CRL cron job scheduler (runs once an hour in this example):
sub_ca_gen_crl_scheduler:
  minute: "0"
  hour: "1"
  day: "*"
  month: "*"

# root CA administrator name
sub_ca_root_ca_user: "ca"

# root CA administrator group
sub_ca_root_ca_group: "ca"

# root CA hostname (MUST BE THE SAME as "root_ca_name" of "root_ca" ROLE!)
sub_ca_root_ca_name: "s1-root-ca-01"

# If root CA private key is encrypted, then you must provide
# its passphrase to sign the sub CA certificate
# valid values: "yes" or "no". If "yes" is specified, you will be prompted
# to input the root CA key passphrase
sub_ca_is_root_ca_key_encrypted: "yes"
```
Take a look at **tests**.
