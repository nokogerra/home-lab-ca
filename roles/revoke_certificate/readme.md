# Home lab revoke certificates
> **Do not change the ansible inventory structure, group names must remain the same. Groups are used for "delegate_to" in some tasks**. Actually, inventory group names are used only in "sign_certificate.yml" and "generate_root_ca_crl.yml" tasks of "sub_ca" role and, obviously, in playbooks. So, you can change the inventory group names, but in that case you have to change those tasks accordingly.
> Just put root CA or sub CA FQDNs or IPs into root_ca or sub_ca group accodringly.

This is the fourth role of the home-lab-ca bundle and it revokes certificates by defined serial numbers and/or common names.<br />
CA administrator must be used as an ansible user with "become:false", so **without root privileges**.<br />
I use ssh key pair to authenticate as a "ca" user on a remote host.
### Process overview
1. Revoke certificates by given serial numbers;
2. Find serial numbers of certificates for provided CNs;
3. Revoke certificates by derived on the step #2 serial numbers;
2. Generate and publish CA CRL.
### Variables
```
# CA administrator and its primary group
revoke_certificate_ca_user: "ca"
revoke_certificate_ca_group: "ca"

# bool value, that defines if the CA key is encrypted
# valid values: "yes" or "no", if "yes" is specified,
# you will be prompted to input the CA key passphrase
revoke_certificate_is_ca_key_encrypted: "yes"


# Certificate revocation reason. Valid values:
# unspecified
# keyCompromise
# cACompromise
# affiliationChanged
# superseded
# cessationOfOperation
# certificateHold
# removeFromCRL
# privilegeWithdrawn
# aACompromise
revoke_certificate_reason: "keyCompromise"

# CA (issuer) common name (the same as its hostname)
# It can be viewed in a certificate content
revoke_certificate_ca_name: "s1-sub-ca-01"

# Common names of certificates to be revoked
# be careful, if you define this variable,
# all VALID certificates with specified CNs will be revoked
# alredy revoked certificates with such CNs will be ignored
revoke_certificate_cn:
  - "vasyan.nokogerra.lab"
  - "taras.nokogerra.lab"

# Serial numbers of certificates to be revoked
# The certificate MUST BE VALID (V), you can check it in /ca/db/index
# of the issuer. In case it is already revoked (R), the task will fail.
revoke_certificate_serial:
  - "05125D1517AFEF75EF7C281C9DDE4D8977532A8C"

# You can use both "revoke_certificate_cn" and "revoke_certificate_serial"
# variables at the same time, or only one of them
```
Take a look at **tests**.
