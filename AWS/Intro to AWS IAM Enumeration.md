https://app.pwnedlabs.io/labs/intro-to-aws-iam-enumeration

### Scenario 

You are a security consultant hired by the global logistics company, Huge Logistics. Following suspicious activity, you are tasked with enumerating the IAM user `dev01` and mapping out any potentially compromised resources. Your mission is to enumerate and evaluate IAM roles, policies, and permissions.

### Information provided

```
IAM user: dev01

Password: G3tt1ngStar73d!

https://794929857501.signin.aws.amazon.com/console

Access key ID: AKIA3SFMD<REDACTED>

Secret access key: OIMngHtqvAZkRf6D<REDACTED>
```

### Enumeration

There are two approaches to solving this challenge. Since we are provided with both, the credentials and the access keys, we can use either the management console (GUI) or the AWS CLI.

#### Management console

We use the link provided which has the `Account number` in it, we enter the username and password.

Since we are tasked with enumerating the IAM user `dev01`, we navigate to the `IAM` services in the console.

![](./images/Pasted%20image%2020260719004847.png)

We navigate to the `IAM users` tab and click on our desired IAM user, which is `dev01`

![](./images/Pasted%20image%2020260719004944.png)

We can see the permissions attached to this IAM user. There are 3 policies attached in total, 2 of them are `Directly` attached and one of them is `Inline`.

Inline policies do not have an ARN associated with them and are only valid for that particular IAM user. They cannot be reused for anything else and will get deleted alongwith the IAM user since they are not separate entities.

Here are the policy permissions in detail after expanding:-

`AmazonGuardDutyReadOnlyAccess:`

![](./images/Pasted%20image%2020260719005342.png)
This policy provides us read only access to Amazon Guard Duty which is a security service provided by AWS.

`dev01:`

![](./images/Pasted%20image%2020260719005433.png)
This is a customer managed policy and provides access to the `Get` and `List` commands in `iam` service of AWS.

`S3_Access:`

![](./images/Pasted%20image%2020260719005504.png)
This is the inline policy which provides `ListBucket` and `GetObject` access on a particular bucket named `hl-dev-artifacts`

We can explore the findings in Guard Duty and play around, however I will skip to the S3 bucket.

If you navigate to the S3 buckets service in management console, you can see that you do not have access to list all the buckets! We have access to list the contents of only one bucket mentioned. How are you going to click on the bucket if you cannot see it? We got an interesting solution to this. 

We will supply the parameters in the URL directly in the session where we have logged in.

```
https://s3.console.aws.amazon.com/s3/buckets/<bucket-name>?region=us-east-1
```

In our case, the bucket name is `hl-dev-artifacts`

Once we paste the URL in the browser, the following page will open up.

![](./images/Pasted%20image%2020260719010252.png)

You can view the contents of `flag.txt` from here.

If you want to check further what the second policy called `dev01` read further or skip to CLI.
First we need to get the information about roles, and one of them seems interesting called `BackendDev`

![](./images/Pasted%20image%2020260719015447.png)

We will check its trust relationships.

![](./images/Pasted%20image%2020260719015532.png)

This means that this role can be assumed by our `dev01` IAM user using AWS STS.

The policy attached to this role:-

![](./images/Pasted%20image%2020260719015646.png)

It means, if we assume this role, we will be able to describe EC2 instances and retrieve secrets from the AWS Secrets Manager.

#### AWS CLI

Now, we will perform the same steps using the CLI instead.

```
aws configure
```

After you enter the above command, enter the access key and secret access key once prompted to.

We will check our identity first.

```
aws sts get-caller-identity
```

```
Output:-

{
    "UserId": "AIDA3SFMDAPOWFB7BSGME",
    "Account": "794929857501",
    "Arn": "arn:aws:iam::794929857501:user/dev01"
}
```

We have successfully logged in as `dev01`

We will list the inline policies attached to our IAM user.

```
aws iam list-user-policies --user-name dev01
```

```
Output:-

{
    "PolicyNames": [
        "S3_Access"
    ]
}
```

Now we will list the managed policies which are attached to the user.

```
aws iam list-attached-user-policies --user-name dev01
```

```
Output:-

{
    "AttachedPolicies": [
        {
            "PolicyName": "AmazonGuardDutyReadOnlyAccess",
            "PolicyArn": "arn:aws:iam::aws:policy/AmazonGuardDutyReadOnlyAccess"
        },
        {
            "PolicyName": "dev01",
            "PolicyArn": "arn:aws:iam::794929857501:policy/dev01"
        }
    ]
}
```

We will check for presence of IAM Groups.

```
aws iam list-groups
```

```
Output:-

{
    "Groups": []
}
```

Since there are no IAM Groups present, we can proceed to the next step of listing down the permissions present inside each policy.

First we will get the details about the policy, including the default versions.

```
aws iam get-policy --policy-arn arn:aws:iam::aws:policy/AmazonGuardDutyReadOnlyAccess
```

```
Output:-

{
    "Policy": {
        "PolicyName": "AmazonGuardDutyReadOnlyAccess",
        "PolicyId": "ANPAIVMCEDV336RWUSNHG",
        "Arn": "arn:aws:iam::aws:policy/AmazonGuardDutyReadOnlyAccess",
        "Path": "/",
        "DefaultVersionId": "v4",
        "AttachmentCount": 1,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "Description": "Provides read only access to Amazon GuardDuty resources",
        "CreateDate": "2017-11-28T22:29:40+00:00",
        "UpdateDate": "2023-11-16T23:07:06+00:00",
        "Tags": []
    }
}
```

We use the default version mentioned to get the actual permissions present inside the policy.

```
aws iam get-policy-version --version-id v4 --policy-arn arn:aws:iam::aws:p
olicy/AmazonGuardDutyReadOnlyAccess
```

```
Output:-

{
    "PolicyVersion": {
        "Document": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Action": [
                        "guardduty:Describe*",
                        "guardduty:Get*",
                        "guardduty:List*"
                    ],
                    "Resource": "*"
                },
                {
                    "Effect": "Allow",
                    "Action": [
                        "organizations:ListDelegatedAdministrators",
                        "organizations:ListAWSServiceAccessForOrganization",
                        "organizations:DescribeOrganizationalUnit",
                        "organizations:DescribeAccount",
                        "organizations:DescribeOrganization",
                        "organizations:ListAccounts"
                    ],
                    "Resource": "*"
                }
            ]
        },
        "VersionId": "v4",
        "IsDefaultVersion": true,
        "CreateDate": "2023-11-16T23:07:06+00:00"
    }
}
```

Similarly we do it for the `dev01` policy as well.

```
aws iam get-policy --policy-arn arn:aws:iam::794929857501:policy/dev01
```

```
Output:-

{
    "Policy": {
        "PolicyName": "dev01",
        "PolicyId": "ANPA3SFMDAPOZWBBGZD4I",
        "Arn": "arn:aws:iam::794929857501:policy/dev01",
        "Path": "/",
        "DefaultVersionId": "v8",
        "AttachmentCount": 1,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "Description": "dev01 policy",
        "CreateDate": "2023-10-01T20:29:16+00:00",
        "UpdateDate": "2025-12-08T12:46:13+00:00",
        "Tags": []
    }
}
```

```
aws iam get-policy-version --version-id v8 --policy-arn arn:aws:iam::79492
9857501:policy/dev01
```

```
{
    "PolicyVersion": {
        "Document": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Sid": "VisualEditor0",
                    "Effect": "Allow",
                    "Action": [
                        "iam:Get*",
                        "iam:List*"
                    ],
                    "Resource": "*"
                }
            ]
        },
        "VersionId": "v8",
        "IsDefaultVersion": true,
        "CreateDate": "2025-12-08T12:46:13+00:00"
    }
}
```

There are no versions present for inline policies.

```
aws iam get-user-policy --user-name dev01 --policy-name S3_Access
```

```
Output:-

{
    "UserName": "dev01",
    "PolicyName": "S3_Access",
    "PolicyDocument": {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "s3:ListBucket",
                    "s3:GetObject"
                ],
                "Resource": [
                    "arn:aws:s3:::hl-dev-artifacts",
                    "arn:aws:s3:::hl-dev-artifacts/*"
                ]
            }
        ]
    }
}
```


We will try to enumerate more details using the second policy called `dev01`.
Lets list out all the roles and see if there is anything interesting.

```
aws iam list-roles
```

The output consisted of many roles, but I have shown the one which looks interesting.

```

{
            "Path": "/",
            "RoleName": "BackendDev",
            "RoleId": "AROA3SFMDAPO2RZ36QVN6",
            "Arn": "arn:aws:iam::794929857501:role/BackendDev",
            "CreateDate": "2023-09-29T12:30:29+00:00",
            "AssumeRolePolicyDocument": {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Effect": "Allow",
                        "Principal": {
                            "AWS": "arn:aws:iam::794929857501:user/dev01"
                        },
                        "Action": "sts:AssumeRole"
                    }
                ]
            },
            "Description": "Grant permissions to backend developers",
            "MaxSessionDuration": 3600
        },
```

In the above role, you can see in the `AssumeRolePolicyDocument` section. Our IAM user `dev01` is able to assume this particular role. The use case seems to be backend development. Now, we will check for policies attached to this particular `BackendDev` role.

```
aws iam list-attached-role-policies --role-name BackendDev
```

```
Output:-

{
    "AttachedPolicies": [
        {
            "PolicyName": "BackendDevPolicy",
            "PolicyArn": "arn:aws:iam::794929857501:policy/BackendDevPolicy"
        }
    ]
}
```

We will try to enumerate the policy called `BackendDevPolicy` which is attached to the role.

```
aws iam get-policy --policy-arn arn:aws:iam::794929857501:policy/BackendDevPolicy
```

```
Output:-

{
    "Policy": {
        "PolicyName": "BackendDevPolicy",
        "PolicyId": "ANPA3SFMDAPO7OINIQIRR",
        "Arn": "arn:aws:iam::794929857501:policy/BackendDevPolicy",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 1,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "Description": "Policy defining permissions for backend developers",
        "CreateDate": "2023-09-29T12:44:09+00:00",
        "UpdateDate": "2023-09-29T12:44:09+00:00",
        "Tags": []
    }
}
```

Using the version, we will enumerate the exact permissions of this particular role policy.

```
aws iam get-policy-version --policy-arn arn:aws:iam::794929857501:policy/BackendDevPolicy --version-id v1
```

```
Output:-

{
    "PolicyVersion": {
        "Document": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Sid": "VisualEditor0",
                    "Effect": "Allow",
                    "Action": [
                        "ec2:DescribeInstances",
                        "secretsmanager:ListSecrets"
                    ],
                    "Resource": "*"
                },
                {
                    "Sid": "VisualEditor1",
                    "Effect": "Allow",
                    "Action": [
                        "secretsmanager:GetSecretValue",
                        "secretsmanager:DescribeSecret"
                    ],
                    "Resource": "arn:aws:secretsmanager:us-east-1:794929857501:secret:prod/Customers-QUhpZf"
                }
            ]
        },
        "VersionId": "v1",
        "IsDefaultVersion": true,
        "CreateDate": "2023-09-29T12:44:09+00:00"
    }
}
```
 
 According to the permissions in the above policy, the `BackendDevPolicy` should be able to describe the EC2 instances and also retrieve secrets stored in the `SecretsManager`.  

Now, our aim is to list the contents of the S3 bucket available to us.

```
aws s3 ls s3://hl-dev-artifacts
```

```
Output:-

2023-10-02 02:09:53       1235 android-kotlin-extensions-tooling-232.9921.47.pom
2023-10-02 02:09:53     214036 android-project-system-gradle-models-232.9921.47-sources.jar
2023-10-02 02:08:05         32 flag.txt
```

Now, to extract the contents of `flag.txt` onto the terminal we will use one final command:-

```
aws s3 cp s3://hl-dev-artifacts/flag.txt -
```

The output will be the flag which solves this lab!

