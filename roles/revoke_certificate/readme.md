# Home lab revoke certificates
> **Do not change the ansible inventory structure, group names must remain the same. Groups are used for "delegate_to" in some tasks**
> Just put root CA or sub CA FQDNs or IPs into root_ca or sub_ca group accodringly.
> Otherwise, you should search/replace all entries of the ansible group names in all home-lab-ca roles.<br />
This is the fourth role of the home-lab-ca bundle and it revokes certificates by defined serial numbers and/or common names.<br />
CA administrator must be used as an ansible user with "become:false", so **without root privileges**.<br />
I use ssh key pair to authenticate as a "ca" user on a remote host.
### Process overview
1. Revoke certificates;
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
revoke_certificate_cn:
  - "vasyan.nokogerra.lab"
  - "taras.nokogerra.lab"

# Serial numbers of certificates to be revoked
# The certificate must be valid (V), you can check it in /ca/db/index
# of the issuer. In case it is already revoked (R), the task will fail.
revoke_certificate_serial:
  - "05125D1517AFEF75EF7C281C9DDE4D8977532A8C"
```
Take a look at **tests**.
