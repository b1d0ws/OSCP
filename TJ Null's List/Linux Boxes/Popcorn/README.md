## Popcorn

Difficulty: Medium

### User Flag

```
rustscan -a popcorn.htb --ulimit 500 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/64c88446-fed3-411d-9f1a-b550ea19881e)

<br>

On port 80 we have only a normal website.

![website](https://github.com/b1d0ws/OSCP/assets/58514930/27cd4e9d-d121-49aa-8eda-9f63d4eaa3b2)

<br>

Gobuster return to us a /torrent directory.

```
gobuster dir -u http://popcorn.htb --wordlist=/usr/share/wordlists/dirb/big.txt -t 50
```

![goBuster](https://github.com/b1d0ws/OSCP/assets/58514930/70ee125f-f3a0-492b-8cc4-6717e4b23c28)

<br>

You can register and upload your torrent on this Torrent Hoster.

![torrentSite](https://github.com/b1d0ws/OSCP/assets/58514930/ed770122-77ca-4819-aa5f-68d82ea3699a)

<br>

Upload a sample torrent and edit it's image, but change Content-Type to "image/png" to upload a php reverse shell.

![uploadingTorrent](https://github.com/b1d0ws/OSCP/assets/58514930/9bb4db5c-78d7-45e1-b217-e7d07a93748c)

![editImage](https://github.com/b1d0ws/OSCP/assets/58514930/cae3dbb7-8dc4-4254-a8ab-cb2cb565f5a3)

![editImage2](https://github.com/b1d0ws/OSCP/assets/58514930/2c0ded7a-e44d-4e54-b495-71a0ae9d9750)

![changingContentType](https://github.com/b1d0ws/OSCP/assets/58514930/de9c0567-ac2f-40f1-bb67-e943ecadd82b)

<br>

Now click on "Image File Not Found"" and get your reverse shell.

### Root Flag

If you enumerate george home directory, there is this motd.legal-displayed file.  

Searching for exploits with searchsploit, you find one that works.

```
chmod 700 /var/www/.ssh
./exploit.sh
```

![findingMotdFile](https://github.com/b1d0ws/OSCP/assets/58514930/7fc4f4e4-eae9-49c9-9fbf-3dc0c18eef75)

![motdExploits](https://github.com/b1d0ws/OSCP/assets/58514930/fa2c7ea8-1f26-4a2e-803b-1cf69ce157fc)

![privEsc](https://github.com/b1d0ws/OSCP/assets/58514930/90c86104-48b9-4ee9-bd5d-be914cda715a)
