# Home lab CA
This bundle is intended to provide a simple Certificate Authority for non-production purposes, for example for a home lab. It is based on openssl, however openssl ansible modules are not used to make the process more transparent. The bundle consists of 4 roles that are closely connected to each other:
- root_ca. It installs root CA;
- sub_ca. It installs subordinate CA;
- sign_certificate. It signs certificates with specified data;
- revoke_certificate. It revokes certificates.
Detailed description for each role is available in the corresponding directories (e.g. roles/root_ca).<br />
### What you will get
You gonna get 2 CAs: root CA and sub CA. Actually, number of sub CAs is optional, however I don't think anyone needs more than one sub CA in a home lab.<br />
Root CA is intended to issue certificates for sub CAs only, there are no extensions configured in its openssl.cnf for regular server/client certificates.<br />
Sub CA can't issue certificates for other sub CAs, it's forbidden in root CA openssl.cnf (sub_ca_ext, basicConstraints - pathlen). If you want to change this behavior, edit roles/root_ca/templates/openssl.cnf.j2.<br />
Each CA (root and sub) publishes its own certificate and CRL at:
- http://ca_hostname.ca_domain/ca_hostname.crt
- http://ca_hostname.ca_domain/ca_hostname.crl
Issued certificates are available at http://ca_hostname.ca_domain/certs.<br />
All these items are symlinks to /ca directory of CA.</br>
On a CA file system there is /ca directory:
- /ca/certs - all the issued certificates are stored here;
- /ca/private - CA key is stored in this directory;
- /ca/csrs - CSR location (for CA own CSR and client CSRs);
- /ca/db - database files location, /ca/db/index is a certificate DB.
Also you will get an automated way to issue and revoke certificates with playbooks.
### Common guidance
1. Make sure you have DNS in your lab, and all the names are resolvable;
2. Role sub_ca (installs sub CA) requires access to both root CA and sub CA, so CA user (it's also ansible user) and its primary group should be the same on both CAs, otherwise you will be forced to configure other user in root CA tasks of this role. I haven't tested this bundle with different CA users for root and sub CAs. If you are concerned about a security part (not a case in a home lab, I guess), then you can use different ssh keys for the same user, e.g.:
```
cat .ssh/config 

Host s1-sub-ca-01*
User ca
IdentityFile ~/.ssh/my_sub_key

Host s1-root-ca-01*
User ca
IdentityFile ~/.ssh/my_root_key
```
3. Check roles directories for detailed info and examples.