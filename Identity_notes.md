# Identity

## STS & Cross account access

### AssumeRole API

* Define IAM role
* Use login from 3 party identity provider or web application
* After user's identity has been confimed, call STS AssumeRole API
* STS return role credentials. 

What AWS services user can take a action, depends on the defined role.

## Identity federation with SAML & Cognito

Principle of identity federation: Trade a token retrieved from 3. party, such as from Active Directory (SAML) and Cognito (Cognito user pools, Facebook) with AWS temporary credentials by AssumeRole API call to STS. 

### 3 types of identity federation

* SAML: For enterprise user
* Identity broken: Also for enterprise user if the company does not use SAML. Identity broker needs to be programed.
* Cognito: For web application user

