# Home lab sign certificates
> **Do not change the ansible inventory structure, group names must remain the same. Groups are used for "delegate_to" in some tasks**
> Just put root CA or sub CA FQDNs or IPs into root_ca or sub_ca group accodringly.
> Otherwise, you should search/replace all entries of the ansible group names in all home-lab-ca roles.<br />
This is the third role of the home-lab-ca bundle and it generates CSRs and keys on a localhost, then transfers the CSRs to a CA, sign certificates and fetch them from the CA to the localhost. The target of a playbook, based on this role, is a CA, it doesn't matter which one (root or sub). I prefer to use ansible inventory group name.<br />
CA administrator must be used as an ansible user with "become:false", so **without root privileges**.<br />
I use ssh key pair to authenticate as a "ca" user on a remote host.
### Process overview
1. Make directory structure on a localhost in /home/$USER/home_lab_ca_local: certs, configs, csrs, private;
2. Generate an openssl config for each CSR, make CSR and key;
3. Transfer all the CSRs to the CA, sign the certificates ans publish them at http://ca_fqdn/certs;
4. Fetch all the issued certificates to the localhost /home/$USER/home_lab_ca_local/certs.
### Certificate extensions
There are 3 certificate extensions available:
```
# "server_ext" - available only on sub CA
[server_ext]
...
authorityKeyIdentifier = keyid:always
basicConstraints = critical,CA:false
...
extendedKeyUsage = clientAuth,serverAuth
keyUsage = critical,digitalSignature,keyEncipherment
subjectKeyIdentifier = hash
...

# "client_ext" - available only on sub CA
[client_ext]
...
authorityKeyIdentifier = keyid:always
basicConstraints = critical,CA:false
...
extendedKeyUsage = clientAuth
keyUsage = critical,digitalSignature
subjectKeyIdentifier = hash
...

# "sub_ca_ext" - available only on root CA
[sub_ca_ext]
...
authorityKeyIdentifier = keyid:always
basicConstraints = critical,CA:true,pathlen:0
...
extendedKeyUsage = clientAuth,serverAuth
keyUsage = critical,keyCertSign,cRLSign
...
subjectKeyIdentifier = hash
```

### Variables
```
# CA FQDN, from which the issued certificates will be fetched:
sign_certificate_ca_fqdn: "s1-sub-ca-01.nokogerra.lab"

# CA administrator and its primary group
sign_certificate_ca_user: "ca"
sign_certificate_ca_group: "ca"

# bool value, that defines if the CA key is encrypted
# valid values: "yes" or "no", if "yes" is specified,
# you will be prompted to input the CA key passphrase
sign_certificate_is_ca_key_encrypted: "yes"

# An extension, which will be used for all the certificates,
# that is going to be issued during the current playbook run
# "sub_ca_ext" must be used only for reissuing a sub CA certificate (against a root CA)
sign_certificate_extensions: "server_ext"

# In case, you haven't modify the policy in openssl.cnf.j2 of root and sub CAs
# then the country MUST be the same as root and sub CAs country
sign_certificate_country: "RU"

# Feel free to choose a state
sign_certificate_state: "Moscow"

# In case, you haven't modify the policy in openssl.cnf.j2 of root and sub CAs
# then the org name MUST be the same as root and sub CAs org name
sign_certificate_organization: "NOKOGERRA-LAB"

# key length (bits)
sign_certificate_key_length: "2048"

# a list of data is used to generate CSRs
# "cn", "days" and "encrypt" are mandatory
# if "sans" is not specified, then the certificate will have
# a single SAN equals to a "cn"
# valid values for "encrypt": "yes" or "no"
# if "yes" is specified, then you will be prompted
# to input the key passphrase
# usually it is convenient only for a sub CA key
sign_certificate_common_names:
  - cn: "vasyan.nokogerra.lab"
    days: 720
    encrypt: "yes"
    sans:
      - "DNS.1 = vasyan.nokogerra.lab"
      - "DNS.2 = vasyanpro.nokogerra.lab"
      - "IP.1 = 10.215.102.94"
  - cn: "*.nokogerra.lab"
    days: 30
    encrypt: "no"
  - cn: taras.nokogerra.lab
    days: 60
    encrypt: "no"
```
Take a look at **tests**.
