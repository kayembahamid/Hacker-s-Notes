# AWS Accounts, Users, Groups, Roles, Policies

## AWS Accounts, Users, Groups, Roles, Policies

Below is a graphical representation of the key components of Identity Access Mangement in AWS:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MPe5uLB0Bz5EzTR_O6v%2F-MPe6oFA-3xnA55DNOdU%2Fimage.png?alt=media\&token=87bec597-44c0-4c42-94d7-92abf710d834)

* Organization / root / management account can have multiple other accounts
* An account can have Users, Groups, Roles and Policies
* Users can be members of Groups and Groups can contain Users
* Role is a secure way to grant termporary permissions to trusted entities:
  * Another AWS account (yours or 3rd party's)
  * AWS service
  * Web Identity
  * SAML Federation
  * All of the above mentioned trusted entities can assume a Role given they have the permission `sts:AssumeRole`
* Policies signify what can/can't be done with resources (i.e EC2 `instance`, `image`, `network interface`, `security group`, etc.). Policies are defined as JSON objects
* Level of access that a User, Group or a Role (identities) has on certain resources, is defined by Policies that are attached to said identities
