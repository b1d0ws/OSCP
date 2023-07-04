## Irked

Difficulty: Easy

### User Flag

#### Foothold

Rustscan give us some interesting ports like 6697 and 8067 that are related to IRC.

```
rustscan -a 10.10.10.117 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/d0d97ab8-a657-476e-8a48-25d7c2e02f2e)

<br>

Website looks normal and also give us a clue that some IRC is implemented.

![website](https://github.com/b1d0ws/OSCP/assets/58514930/2abdf452-3d55-41a9-b42e-305083cb0a1d)

<br>

If we search for IRC exploits, we find a Backdoor in Github.

![searchingExploit](https://github.com/b1d0ws/OSCP/assets/58514930/e568d8a8-5c8f-48f2-9e21-23baa5bda545)

<br>

Edit the exploit, use it and get a reverse shell.

```
python3 exploit.py -payload bash 10.10.10.117 6697
```

![reverseShell](https://github.com/b1d0ws/OSCP/assets/58514930/63ed5053-849c-458a-9ff1-c5314d14e5dd)

#### User Access

Searching inside djmardov files, we find a backup file containing a password.  

We can use this password to extract info from the website image with steghide.

```
steghide extract -sf irked.jpg
```

![backup](https://github.com/b1d0ws/OSCP/assets/58514930/1017777e-8c84-4645-8215-2b2dbaeeeefc)

![stegHide](https://github.com/b1d0ws/OSCP/assets/58514930/2bcbc292-04fd-4b88-9b47-6cbabac3b291)

### Root Flag

There is this different SUID binary: /usr/bin/viewuser.  

If we try to execute, it seems that it is trying to "sh /tmp/listusers" but can't find the file.

![suid](https://github.com/b1d0ws/OSCP/assets/58514930/4c7f97d9-4432-4422-b5df-4250aecd47b0)

<br>

We can create this file summoning a bash inside it and get root.

```
nano /tmp/listusers

#!/bin/bash
bash -l
```

![privilegeEscalation](https://github.com/b1d0ws/OSCP/assets/58514930/4e1c92fc-4f09-4f34-acca-54abbe3150ca)
