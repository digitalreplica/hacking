# AWS Assume Role

The standard way for services in one AWS account to do things in another AWS account is for a service to "assume a role" in the other account. It's elegant when done well. But it's so pervasive that it can easily become a tangled mess, and exploitable, making it a primary method of lateral movement within AWS accounts.

Potential weaknesses:
* If used in a build pipeline to create and deploy code artifacts, it may be possible to cross accounts with admin access. Deployment systems need full access to create and delete any AWS resource.
* Developers and admins frequently add themselves into assume role policies for testing and debugging. Getting even a limited user account could lead to elevated privileges.
* Service providers have roles to monitor and manage accounts, and typically ask for overly broad access.

## Understanding who can assume a role
AWS Console: View IAM roles, select a role and look at the "Trust relationships" tab
AWS CLI:
```
aws iam list-roles
```

Each role has a AssumeRolePolicyDocument, described at [AWS::IAM::Role - AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-iam-role.html). This has a series of Statements, that say who can assume this role, and how. In each statement is a Principal, which is the who. There are three types of principals.

### AWS principals
These are user and roles in this AWS account or a different AWS account, and should be scrutinized closely.

### Service principals
Confusingly named, this is the way that AWS itself does things in your account. To launch an instance, you need to give the AWS EC2 service permission to attach your role to the instance. For hacking purposes, all service principals can be ignored.

### Federated principals
Non-AWS identity providers such as SAML. Not as common as AWS principals, but should definately be looked at.

## Wildcard principals
Principals can't use wildcards, but there is a convention to say "all users in an account" using the keyword **root**. Amazon's note
:
> The suffix root in the policy’s Principal attribute equates to “authenticated and authorized principals in the account,” not the special and all-powerful root user principal that is created when an AWS account is created.

Like:
```
"Principal": {
  "AWS": "arn:aws:iam::111122223333:root"
}
```

## Permissions needed to assume a role
To successfully assume a role, two things are needed.
* A AssumeRolePolicyDocument on the role letting principals assume it
* Permissions in your current role to do a STS assume-role

Typically, for pentesting, the second isn't much of a worry, since both are typically set up at the same time.

## Chaining assume roles
Assuming a role may give access to assume a different role. So assume roles can be chained to hop through several to get to a desired role.

## Pentesting Assume Roles
* Run ```aws iam list-roles``` in all accounts to find roles that can be assumed.
* Search through the AssumeRolePolicyDocument statements for **AWS** or **Federated** principals for roles that can be assumed.
* See if you have access to any of those roles.
* If not, search each role to see who can assume **that** role. Keep searching backwards until you find a role you use to start from.
* Assume roles, chaining as needed to get to your desired role.
* If you get a permissions error, check permissions for that user or role for the IAM assume-role permission. Backtrack as needed to find another role to start from.

## Useful commands
Get roles in account that can be assumed (only AWS principals though)
```
aws iam list-roles | jq -r '.Roles[] | select((.AssumeRolePolicyDocument.Statement[].Principal.AWS? | length) > 0) | { RoleName: .RoleName, AssumeRole: .AssumeRolePolicyDocument.Statement[].Principal.AWS}'
```

Assuming a role. The role session name is a required field but can be anything.
```
aws sts assume-role --role-arn "arn:aws:iam::123456789012:role/example-role" --role-session-name AWSCLI-Session
```

## References
* [How to use trust policies with IAM roles](https://aws.amazon.com/blogs/security/how-to-use-trust-policies-with-iam-roles/)
* [AWS JSON policy elements: Principal - AWS Identity and Access Management](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html)