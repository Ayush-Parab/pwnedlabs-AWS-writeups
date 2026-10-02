https://app.pwnedlabs.io/labs/identify-the-aws-account-id-from-a-public-s3-bucket

### Scenario:

The ability to expose and leverage even the smallest oversights is a coveted skill. A global Logistics Company has reached out to our cybersecurity company for assistance and have provided the IP address of their website. Your objective? Start the engagement and use this IP address to identify their AWS account ID via a public S3 bucket so we can commence the process of enumeration.

### Information provided

```
AWS credentials and IP address

IP address: 54.204.171.32

Access key ID: AKIAWHE<REDACTED>

Secret access key: /eThpKvOcZBoa3<REDACTED>
```

### Enumeration

Using `nmap` to map the ports in use for the given IP address.

Command:-

```
nmap 54.204.171.32 -sC -sV -p80
```

Output:-

```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-07-14 22:21 IST
Nmap scan report for ec2-54-204-171-32.compute-1.amazonaws.com (54.204.171.32)
Host is up (0.28s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Mega Big Tech
|_http-server-header: Apache/2.4.52 (Ubuntu)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.23 seconds
```

It is very clear that port `TCP-80` is open and running a `Apache` web server.

### Web Enumeration

![](./images/Pasted%20image%2020260714222917.png)

From this website, it is visible that the images have been uploaded from an `AWS S3` bucket. We navigate to the S3 bucket.

![](./images/Pasted%20image%2020260714223025.png)

We can see a list of contents present inside the `S3` bucket named `mega-big-tech`

![](./images/Pasted%20image%2020260714223113.png)

If we try to access the `images` directory under the bucket, we are denied access.

### Account ID enumeration

Now, that we have the name of the public `S3` bucket, we can find out the Account ID using the following tool:-

https://github.com/WeAreCloudar/s3-account-search

Details of how the tool works have been provided in a blog linked inside the repo. In short, it will run at most `10*12` times, it will try combinations from 0-9 for all the 12 digits of the Account ID. 

#### Pre-requisite

We need a role inside our own account with `ListBucket` and `GetObject` policies attached to it.

![](./images/Pasted%20image%2020260714231303.png)

After making this role, we can use it to find the account ID of the public S3 bucket.

Command:-

```
s3-account-search arn:aws:iam::<ATTACKER_ACCOUNT>:role/s3_enumerator_role s3://mega-big-tech
```

Output:-

```
Starting search (this can take a while)
found: 1
found: 10
found: 107
found: 1075
found: <REDACTED>
found: <REDACTED>
found: <REDACTED>
found: <REDACTED>
found: <REDACTED>
found: <REDACTED>
found: <REDACTED>
found: <REDACTED>
```

Hence we found out the associated Account ID!
