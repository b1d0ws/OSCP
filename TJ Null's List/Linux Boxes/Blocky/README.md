## Blocky

Difficulty: Easy

### User Flag

```
rustscan -a blocky.htb --ulimit 5000 -- A -sV -sC
```

Rustscan Informations:

21 - ProFTPD 1.3.5a  
22 - OpenSSH 7.2p2  
25565 - Minecraft 1.11.2  

![rustscan](https://user-images.githubusercontent.com/58514930/219134056-fe628c03-0ab3-49a6-aa45-84c9343a5702.png)

<br>

```
gobuster dir -u http://blocky.htb -w /usr/share/wordlists/dirb/big.txt -t 30
```

![goBuster](https://user-images.githubusercontent.com/58514930/219135492-af594bce-f14f-4d74-9641-9e7db45c29bc.png)

<br>

Inside /plugins we have interesting jar files. Open them with jadx and find credentials --> root:8YsqfCTnvxAUeduzjNSXe22

![firstCredentials](https://user-images.githubusercontent.com/58514930/219135637-b038b37c-94bc-4757-9969-4387f69f7308.png)

<br>

This credentials works in phpmyadmin. There we find a user called notch.

![phpMyAdmin](https://user-images.githubusercontent.com/58514930/219136417-dc40c937-5cfd-4b08-a8fe-5a3fe53100a6.png)

<br>

We can use the first password with this user and access SSH --> notch:8YsqfCTnvxAUeduzjNSXe22

![userFlag](https://user-images.githubusercontent.com/58514930/219136830-a82b0901-12c8-4081-9df9-8b4a47418d2c.png)

<br>

### Root Flag

User notch is in sudo group, just **sudo su**.

![rootUser](https://user-images.githubusercontent.com/58514930/219138345-b7d82b75-68a9-490b-9090-be467236452a.png)
