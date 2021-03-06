# Identity Access Management IAM

* allows you to manage users
* IAM is global
    - it is not tied to a region
    * any users/groups/rules you create are global
* gives you identity federation to
    * facebook
    * linkedin
    * ActiveDirectory
* supports multifactor authentication
* allows you to provide temporary access for users/devices and services when necessary
    * how ???
* allows you to setup your own password rotation policy
* supports PCI-DSS compliance

## IAM policy

* defines what a "trusted entity" can access
* does not define how long they can access the resource(s)
    * that is defined by what kind of entity you use: users and groups are permenant but roles are temporary (todo: is this exactly correct?)
* Amazon has ~148 "policy" objects that you can "attach" to a user, group, role
* You can create your own policy objects
* A policy joins a group of "actions" with a group of "resources" via an ALLOW or DENY
* A policy is a JSON object
* A policy is a bit like a firewall rule - you can allow/deny access to sets of
  actions within a certain amazon service and optionally restrict to
  particular amazon resources
* A policy object can have many users/groups/roles attached and a
  user/group/role can have many policy objects attached
* A "managed policy" is one what is kept in the IAM policy repository
    * A managed policy can references up to 2 entities e.g. user/group/role.
    * Managed policies are automatically updated by Amazon when new features become available
* An "inline policy" can be pasted inline into a particular user/group/role
* Policies are written in _IAM Policy Language_ (A dialect of JSON)

You can attach a policy to any of

1. Users
2. Roles
3. Groups

There are ? kinds of policies

1. Trust policy
    * Defines who is allowed to assume a role
    * Written in IAM policy language
    * JSON format
1. Permissions policy
    * uses with a role
        * defines what actions and resources the role can access
    * Written in IAM policy language
    * JSON format

QUESTION: is resour based policy a different type?

### Resource based policies

Some AWS resources support **resource based policies** - you can attach
policies directly to these resources without having to go through an IAM role.

Examples of services which support resource based policies:

1. Amazon Simple Storage Service (S3) buckets
1. Amazon Glacier vaults
1. Amazon Simple Notification Service (SNS) topics
1. Amazon Simple Queue Service (SQS) queues.

Unlike a user-based policy, a resource-based policy specifies who (in the form
of a list of AWS account ID numbers) can access that resource

Scenario:

    You are `you@account-a` and you are accessing a resource in `account-b` via
    a resource-based policy

Advantage:

    You don't have to switch role so you can still access all your stuff in
    `account-a` - handy if you need to copy data to/from `account-b`

Example of resource based policy on an S3 bucket

```json
{
    "Version": "2008-10-17",
    "Id": "Policy1399437007199",
    "Statement": [
        {
            "Sid": "Stmt1399437a002721",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::017242624401:user/jlrscout"
            },
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::jlrscout-assets-staging/*",
                "arn:aws:s3:::jlrscout-assets-staging"
            ]
        }
    ]
}
```

### Principal

* Every policy has a **Principal**
* The Pincipal is the entity which is being allowed to perform actions or access resources
* Things which can be principals in a policy
    1. AWS root account
    1. IAM account
    1. Role
* How to set the _Principal_ in a policy:
    * For permissions policies the "Principal" is inferred by what users or roles the policy is attached to
    * For services which support _resource based policies_ you identify the Principal in the JSON (I think)

## Role

http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html

* Roles are similar to a user except rather than being associated with a person
  it is designed to be able to be assumed by any person/robot who needs it
* A role is a set of permissions
    * it grants access to actions and resources in AWS
* A role can be be used by any of:
    1. An AWS user in the same account
    1. An AWS user in a different account
    1. An AWS web service e.g. EC2
    1. An external user authenticated by an identity provider e.g. SAML, OpenID Connect
* Only IAM users can assume roles - the root account user cannot.
* Roles do not have standard long-term credentials
    * a role has no passwords or access keys
    * instead a temporary security credentials are created when a user assumes a role

Use cases

You can use roles to delegate access to users, applications, or services that
don't normally have access to your AWS resources

> You might want to allow a mobile app to use AWS resources, but not want to
> embed AWS keys within the app (where they can be difficult to rotate and
> where users can potentially extract them)

> You want to give AWS access to users who already have identities defined
> outside of AWS, such as in your corporate directory

> You might want to grant access to your account to third parties so that they
> can perform an audit on your resources

See _delegation_ section below

## Delegation

* grants permission to somebody else to access resources you control
* Setup trust between
    1. the account which controls the resource (the "trusting" account)
    1. the account which wants access to the resource (the "trusted" account)
* Trusted and trusting accounts can be
    1. the same account
    1. two accounts in the same organization
    1. two accounts in different organizations
* Note that trusted and trusting happens at the **account** level, not the IAM user level
    * => as the trusting account you have to trust every user in the trusted account - it is up to an admin of the trusted account to lock down which IAM users can assume the role


To setup a delegation

1. Create a role in the trusting account and attach two policies
    1. Add a permissions policy to the role in the _trusting_ account
        * grant whatever assumes the role the needed permissions to access the resource
        * This is visible in the "Permissions" tab of the AWS console
    1. Add a trust policy to the role
        * says which **accounts** are allowed to grant their users permissions to assume the role
        * Visible under the "Trust relationships" tab in AWS Console
        * Trust relationships for delegation are setup between two **accounts**
          - you cannot delegate access to a single IAM user within an account -
            you must delegate access to any IAM user from that account.
1. To complete the configuration, the administrator of the trusted account must give specific groups or users in that account permission to switch to the role. There doesn't seem to be a handy managed policy for this  so the following snippet is available in the docs:
    ```
    {
    "Version": "2012-10-17",
    "Statement": {
        "Effect": "Allow",
        "Action": "sts:AssumeRole",
        "Resource": "arn:aws:iam::ACCOUNT-ID-WITHOUT-HYPHENS:role/Test*"
    }
    }
    ```
1. After that there are two ways for a user to switch roles
    1. You can then send the users a link that takes the user to the Switch
       Role page with all the details already filled in.
    1. you can provide the user with the account ID number or account alias
       that contains the role and the role name. The user then goes to the
       Switch Role page and adds the details manually.

NB: You cannot switch roles if you sign in as the root account


