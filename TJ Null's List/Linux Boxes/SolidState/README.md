## SolidState

Difficulty: Medium

### User Flag

```
rustscan -a 10.10.10.51 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/4dcb4a38-d459-44f5-9d29-19741538c09e)

<br>

Connecting in port 4555 with telnet we discover a service called JAMES Remote Administration Tool 2.3.2.

Searching por default credentials, we find root:root and that works.

![jamesAdministrator](https://github.com/b1d0ws/OSCP/assets/58514930/6be8a441-6de2-40d2-849a-f90b9c336338)

<br>

In this service, we can reset users password to authenticate in their e-mails using POP3.

```
setpassword mindy mindy123
```

![resetMindyPassword](https://github.com/b1d0ws/OSCP/assets/58514930/49c45f01-c431-48f0-a293-35d7bc90924b)

<br>

Reading mindy's e-mails, we discover her password.

```
USER mindy
PASS mindy123
LIST
RETR 2
```

![mindyEmail](https://github.com/b1d0ws/OSCP/assets/58514930/c49f49c8-7c7f-480a-9ef3-b259ee70893d)

![mindyEmailPassword](https://github.com/b1d0ws/OSCP/assets/58514930/0a2c92e7-705c-4523-8f93-d5e7f7d6f883)

<br>

Now we can login via SSH and get the user Flag.

![rbashUserFlag](https://github.com/b1d0ws/OSCP/assets/58514930/b4bbf52b-5ab4-4bea-b412-84ed9f1a32f9)

<br>

## Root Flag

Since rbash is a very restricted shell, we can bypass this by [different ways](https://www.hacknos.com/rbash-escape-rbash-restricted-shell-escape/).   
The most simple is login with SSH like this:

```
ssh mindy@10.10.10.51 -t "bash --noprofile"
```

![rbashBypass](https://github.com/b1d0ws/OSCP/assets/58514930/480c4380-f190-4c77-8583-dea4ea6d56cb)

<br>

Another way is exploiting James Administration Tool, using this [exploit](https://www.exploit-db.com/exploits/50347).

![jamesExploit](https://github.com/b1d0ws/OSCP/assets/58514930/b15619b7-7bb9-40a2-8e56-ea726d4559a5)

<br>

Now that we have a decent shell, we can run LinPeas. At some point, it will indicate to us that are some interesting files inside /opt folder.

![tmpFile](https://github.com/b1d0ws/OSCP/assets/58514930/42f1c90d-2ee7-45a8-a212-e58b6c195dab)

<br>

There is this tmp.py file that call system and removes everything inside /tmp folder. It looks like a script that was created to run at a scheduled time.  

We can find this by putting something inside /tmp and waiting or use pspy to identifiy the process running a possible cronjob.  

After this, just change the content of tmp.py to get the root user. In this case a put a SUID bit in /bin/bash.

```
chmod 4755 /bin/bash
bash -p
```

![gettingRoot](https://github.com/b1d0ws/OSCP/assets/58514930/a76ab8d8-1f3b-4ca2-a008-83b9531fc2bd)
