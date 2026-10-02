https://app.pwnedlabs.io/labs/create-custom-tooling-to-explore-aws

### Scenario

We have some time before our next AWS engagement, so why not create some custom tooling to make our job easier! 

In this lab we will explore S3 enumeration using python scripts.

### Information provided

```
Sample S3 bucket:-

dev.huge-logistics.com
```

### BurpSuite Proxy setup

We will first setup `BurpSuite` proxy in such a way that it intercepts requests done from the shell using AWS CLI.

First install the `Copy as python requests` extension from the `BApp Store`.

![](Pasted%20image%2020260822204022.png)

Then turn the intercept "ON" in the `BurpSuitePro`

Afterwards use the following commands to set up the proxy in the terminal.

```
export HTTP_PROXY=http://127.0.0.1:8080
export HTTPS_PROXY=http://127.0.0.1:8080
```

The above two commands set the environment variables

```
curl http://127.0.0.1:8080/cert --output ./certificate.cer
```

In the above command, we have downloaded the certificate in a `.cer` format which is the default format of `BurpSuitePro`

```
openssl x509 -inform der -in ./certificate.cer -out ./certificate.pem
```

Using the openssl module we will now convert the `.cer` certificate file to a `.pem` file

```
export AWS_CA_BUNDLE="$(pwd)/certificate.pem"
```

We will set this is new certificate as am environment variable as well.

### Intercepting traffic

```
aws s3 ls s3://dev.huge-logistics.com --no-sign-request 
```

This is the aws cli command we will enter and intercept its request in Burp.

The following two requests will be generated once we enter the command. We will send the second request to the `repeater`

![](Pasted%20image%2020260822204455.png)

![](Pasted%20image%2020260822204512.png)

Once we send the request to the `repeater` and hit `send`:-

![](Pasted%20image%2020260822204653.png)

We get the following response back which is similar to the structure we see on the web page whenever we access an S3 bucket endpoint.

We will copy the following request as python code now:-

![](Pasted%20image%2020260822204826.png)

Once the code gets copied, we will paste it in vscode.

### Script generation

```
import requests

burp0_url = "https://s3.us-east-1.amazonaws.com:443/dev.huge-logistics.com?list-type=2&prefix=&delimiter=%2F&encoding-type=url"

burp0_headers = {"Accept-Encoding": "gzip, deflate, br", "User-Agent": "aws-cli/2.35.19 md/awscrt#0.35.0 ua/2.1 os/linux#6.14.0-37-generic md/arch#x86_64 lang/python#3.14.6 md/pyimpl#CPython m/Z,E,C,b cfg/retry-mode#standard md/installer#exe sid/ed05ed9216f1 md/distrib#ubuntu.25 md/prompt#off md/command#s3.ls", "amz-sdk-invocation-id": "2c88d874-f338-4486-9542-f3c38a4d14ea", "amz-sdk-request": "ttl=20260822T124521Z; attempt=2; max=3", "Connection": "keep-alive"}

requests.get(burp0_url, headers=burp0_headers)
```

We get the above code. We will rename the variables and remove the headers as they are not needed currently.

```
import requests

s3_bucket_url = "https://s3.us-east-1.amazonaws.com:443/dev.huge-logistics.com?list-type=2&prefix=&delimiter=%2F&encoding-type=url"

response = requests.get(s3_bucket_url)

if response.status_code == 200:
    print("The bucket exists and you have access!")
    print(response.text)
elif response.status_code == 404:
    print("Bucket does not exist!")
elif response.status_code == 403:
    print("Bucket exists but you do not have access!")
else:
    print("We are not able to access the bucket!")
```

Type of response codes and what it means:-

- 200 - bucket exists and we have access to it
- 404 - invalid bucket, bucket does not exist
- 403 - bucket exists but you do not have access

This is reflected in the code above which we have written.

The following will be the output part of the response we receive from aws.

```
<?xml version="1.0" encoding="UTF-8"?>
<ListBucketResult xmlns="http://s3.amazonaws.com/doc/2006-03-01/"><Name>dev.huge-logistics.com</Name><Prefix></Prefix><KeyCount>5</KeyCount><MaxKeys>1000</MaxKeys><Delimiter>/</Delimiter><EncodingType>url</EncodingType><IsTruncated>false</IsTruncated><Contents><Key>index.html</Key><LastModified>2023-10-16T17:00:47.000Z</LastModified><ETag>&quot;67b9af5c7866413fccc97dad74b997ff&quot;</ETag><Size>5347</Size><StorageClass>STANDARD</StorageClass></Contents><CommonPrefixes><Prefix>admin/</Prefix></CommonPrefixes><CommonPrefixes><Prefix>migration-files/</Prefix></CommonPrefixes><CommonPrefixes><Prefix>shared/</Prefix></CommonPrefixes><CommonPrefixes><Prefix>static/</Prefix></CommonPrefixes></ListBucketResult>
```

It is the same XML structure but very unreadable because of the formatting.

We will try to parse the output using regex.

There are two important types of XML tags, `<prefix>` which means directories and `<key>` which means the object present inside the directory.

```
import requests
import re

s3_bucket_url = "https://s3.us-east-1.amazonaws.com:443/dev.huge-logistics.com?list-type=2&prefix=&delimiter=%2F&encoding-type=url"

response = requests.get(s3_bucket_url)

if response.status_code == 200:
    print("The bucket exists and you have access!")
    prefixes = re.findall(r'<Prefix>(.*?)</Prefix>', response.text)
    print("Directories discovered:-")
    for prefix in prefixes:
        if prefix == "":
            print("/")
        else:
            print(prefix)
elif response.status_code == 404:
    print("Bucket does not exist!")
elif response.status_code == 403:
    print("Bucket exists but you do not have access!")
else:
    print("We are not able to access the bucket!")
```

The above code will list down the prefixes or directories in a readable manner:-

```
Directories discovered:-
/
admin/
migration-files/
shared/
static/
```

Next, we will check for the keys or files present inside each directory or prefix.

```
import requests
import re

s3_bucket_url = "https://s3.us-east-1.amazonaws.com:443/dev.huge-logistics.com?list-type=2&prefix=&delimiter=%2F&encoding-type=url"
response = requests.get(s3_bucket_url)
if response.status_code == 200:
    print("The bucket exists and you have access!")
    prefixes = re.findall(r'<Prefix>(.*?)</Prefix>', response.text)
    print("Directories discovered:-")
    for prefix in prefixes:
        if prefix == "":
            print("/")
        else:
            print(prefix)
        s3_bucket_new_url = f"https://s3.us-east-1.amazonaws.com:443/dev.huge-logistics.com?list-type=2&prefix={prefix}&delimiter=%2F&encoding-type=url"
        new_response = requests.get(s3_bucket_new_url)
        if new_response.status_code == 200:
            keys = re.findall(r'<Key>(.*?)</Key>', new_response.text)
            for key in keys:
                print(f"File discovered - {key}")
        elif new_response.status_code == 403:
            print(f"You do not have access to the {prefix} directory.")
        else:
            print("We are not able to access the directory!")
elif response.status_code == 404:
    print("Bucket does not exist!")
elif response.status_code == 403:
    print("Bucket exists but you do not have access!")
else:
    print("We are not able to access the bucket!")
```

Output:-

```
The bucket exists and you have access!
Directories discovered:-
/
File discovered - index.html
admin/
You do not have access to the admin/ directory.
migration-files/
You do not have access to the migration-files/ directory.
shared/
File discovered - shared/
File discovered - shared/hl_migration_project.zip
static/
File discovered - static/
File discovered - static/logo.png
File discovered - static/script.js
File discovered - static/style.css
```

We get the list of files accessible to us!

Next part is downloading the files and keeping the same directory structure intact, for this we will utilize the `os` module in python.

Entire code:-

```
import requests
import re
import urllib3
import os

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

s3_bucket_url = "https://s3.us-east-1.amazonaws.com:443/dev.huge-logistics.com?list-type=2&prefix=&delimiter=%2F&encoding-type=url"
response = requests.get(s3_bucket_url)
#print(response)
file_list = []

if response.status_code == 200:
    print("The bucket exists and you have access!")
    #print(response.text)
    prefixes = re.findall(r'<Prefix>(.*?)</Prefix>', response.text)
    print("Directories discovered:-")
    for prefix in prefixes:
        if prefix == "":
            print("/")
        else:
            print(prefix)
        s3_bucket_new_url = f"https://s3.us-east-1.amazonaws.com:443/dev.huge-logistics.com?list-type=2&prefix={prefix}&delimiter=%2F&encoding-type=url"
        new_response = requests.get(s3_bucket_new_url)
        if new_response.status_code == 200:
            keys = re.findall(r'<Key>(.*?)</Key>', new_response.text)
            for key in keys:
                print(f"File discovered - {key}")
                if not key.endswith('/'):
                    file_list.append(key)
        elif new_response.status_code == 403:
            print(f"You do not have access to the {prefix} directory.")
        else:
            print("We are not able to access the directory!")
elif response.status_code == 404:
    print("Bucket does not exist!")
elif response.status_code == 403:
    print("Bucket exists but you do not have access!")
else:
    print("We are not able to access the bucket!")

print("This is the file list:-\n")
print(file_list)

directory_name = "dev.huge-logistics.com"

for file in file_list:
    download_url = f'https://s3.amazonaws.com/dev.huge-logistics.com/{file}'
    file_response = requests.get(download_url, stream=True, verify=False)
    file_path = os.path.join(directory_name, file)
    os.makedirs(os.path.dirname(file_path), exist_ok=True)
    with open(file_path, 'wb') as open_file:
        for data in file_response.iter_content(1024):
            open_file.write(data)
```

After we run the above code, we can see the directory structure and the files downloaded to our system:-

![](Pasted%20image%2020260822210123.png)

The flag for this challenge can be obtained using the following command which calculates a `md5sum` of the zip file:-

```
md5sum ./dev.huge-logistics.com/shared/hl_migration_project.zip
```





