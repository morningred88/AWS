# Security and Compliance for SysOps

## KMS key rotation

### Automatic key rotation

* Key rotation every 1 year
* No change to CMK ID

### Manual key rotation

* Key rotation every 90 days, 180 days
* New key has a different CMK ID
* Better to use alias

## CloudHSM 

* Unlike KMS, you have dedicated hardware to create Customer Master key (CMK).
* HSM: Hardware Security Module
* Need to install software for CloudHSM to work. And the software is used to manage the user access control. IAM is used for CRUD of CMK keys
* Customer can import **asymetric** keys to CloudHSM.
