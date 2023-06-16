## Magic

Difficulty: Medium

### User Flag 

#### Foothold

```
rustscan -a 10.10.10.185 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/17305671-404e-489b-8e35-c2ea062c2272)

<br>

The site seems to upload images, but there is a login procedure before.

![website](https://github.com/b1d0ws/OSCP/assets/58514930/14680e19-d535-4eaa-87b0-c881659acb13)

![login](https://github.com/b1d0ws/OSCP/assets/58514930/e8d28bb5-8763-43bf-8e2c-b62e61890b04)

<br>

You can bypass in two different ways: changing 302 Found to 200 OK accessing direct upload.php or use " ' OR 1=1 -- - " in password field to bypass the login form.

![sqlInjection](https://github.com/b1d0ws/OSCP/assets/58514930/45e4ab2f-b96a-4485-8619-157e4dcf6055)

![uploadPHP](https://github.com/b1d0ws/OSCP/assets/58514930/db8fd7e3-33e5-4a4e-90a3-f62cf7a5bf02)

<br>

Upload functions appears to have two methods of litiming the upload. First the file need to have JPG, JPEG & PNG extension and this can be bypass using double extension.

The other method checks the magic bytes of the files, so we can just upload a reverse.php.png with PNG magic bytes inside it.

```
echo '89 50 4E 47 0D 0A 1A 0A' | xxd -p -r >> reverse.php.png
cat php-reverse-shell.php >> reverse.php.png
```

![creatingPayload](https://github.com/b1d0ws/OSCP/assets/58514930/dca0ee12-82f7-4389-8487-2218d8784e5e)

![uploadedExploit](https://github.com/b1d0ws/OSCP/assets/58514930/629c469a-1ec9-4ea6-b10f-bdc5706ef2fa)

<br>

To execute this file, you discover in the main page source code that the images are located at /images/uploads.

![accessingReverse](https://github.com/b1d0ws/OSCP/assets/58514930/431574a3-2be3-4f6c-a3ca-fcccdc2d0e43)

#### User Access

We find credentials inside /var/www/Magic/db.php, but there is no mysql binary in the machine. You can dump the database portforwarding 3306 port or use mysqldump binary.

Inside Magic database there is another password that can be used to authenticate as theseus.

```
mysqldump -u theseus -p Magic
```

![dbPHP5](https://github.com/b1d0ws/OSCP/assets/58514930/a5b9de31-22d2-4129-8370-9be0822a9b72)

![mysqldump](https://github.com/b1d0ws/OSCP/assets/58514930/1dbbb53e-9b44-4682-a68a-b8c97aa77a40)

![theseusPass](https://github.com/b1d0ws/OSCP/assets/58514930/c77741f4-4797-478b-a5a8-8597446d4a47)

### Root Flag

Enumerating SUID, there is this /bin/sysinfo binary.  

Executing, we notice that he uses some common binaries in linux such as free.

```
find / -type f -perm -04000 -ls 2>/dev/null
```

![sysinfoSUID](https://github.com/b1d0ws/OSCP/assets/58514930/d3ccacd7-8ff3-4de9-938c-f334c474ff92)

![freeOutput](https://github.com/b1d0ws/OSCP/assets/58514930/1196ebb9-bc94-4b33-a37c-7d2f6b6ea267)

<br>

We can trace the system calls with strace to see the binaries being called.  

Doing this, we dsicover that free and other binaries are being called in a relative path. We can PATH Hijack!

```
strace -f sysinfo
```

![straceFree](https://github.com/b1d0ws/OSCP/assets/58514930/29b2464e-a68f-4816-bd95-77c281ba4a76)

![privEsc](https://github.com/b1d0ws/OSCP/assets/58514930/7aeeefcb-e530-4fbc-9749-8f6f19be79b5)
