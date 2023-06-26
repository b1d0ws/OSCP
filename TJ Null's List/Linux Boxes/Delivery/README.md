## Delivery

Difficulty: Easy

### User Flag

```
rustscan -a delivery.htb --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/31555645-ec09-4c73-b339-04e8b55bd426)

![website](https://github.com/b1d0ws/OSCP/assets/58514930/cff19a25-2031-4206-b3da-c1d8bede9b83)

<br>

Looking in the source code of the web we find an important information. Apparently you need access to a @delivery.htb to access the MatterMost server.  

We can create a ticket in helpdesk.delivery.htb to gain access to one.

![sourceCode](https://github.com/b1d0ws/OSCP/assets/58514930/c49b8eb7-8fce-43f9-a4bc-bb9d3ec8ffe0)

![creatingTicket](https://github.com/b1d0ws/OSCP/assets/58514930/d21d2098-9cab-4b36-963e-5511d8247861)

![ticketCreated](https://github.com/b1d0ws/OSCP/assets/58514930/91af5de1-741d-4005-b22d-6d04a45ddd1e)

<br>

With access to 9462228@delivery.htb we can create an account on mattermost.

![matterMost](https://github.com/b1d0ws/OSCP/assets/58514930/df4100cd-ada0-4e95-a746-aa2bfd787371)

![creatingAccount](https://github.com/b1d0ws/OSCP/assets/58514930/bf21c175-8d18-4dbe-ba1d-2788ac3557f6)

<br>

Now access the ticket created to validate the account.

![checkTicket](https://github.com/b1d0ws/OSCP/assets/58514930/7e7610ff-f905-4504-a99e-b60f013cb4d2)

![viewTicket](https://github.com/b1d0ws/OSCP/assets/58514930/1bef6ed3-4c04-4d7d-9764-7f82ca7c0060)

<br>

Access the mattermost panel with the new account and get the credentials to the server. From that you got user flag.

![matterMostPanel](https://github.com/b1d0ws/OSCP/assets/58514930/09e782f0-1215-4428-ab4e-d4ded50e9df2)

### Root Flag

You can find mysql credentials inside /opt/mattermost/config/config.json.

![configJson](https://github.com/b1d0ws/OSCP/assets/58514930/24a1cbf9-74b1-491e-bd49-22d63ebe18bc)

<br>

Access mysql and get the root hash.

```
mysql -u mmuser -p
use mattermost
select username,password from Users;
```

![gettingHashes](https://github.com/b1d0ws/OSCP/assets/58514930/e078939c-9ca6-4c7c-af63-2e3c2c1a8fa9)

<br>

Based on the text found before:
> PleaseSubscribe! may not be in RockYou but if any hacker manages to get our hashes, they can use hashcat rules to easily crack all variations of common words or phrases.  

Generate a custom list based on PleaseSubscribe! password and crack the hash with this wordlist. The password found is the root one.

```
https://infinitelogins.com/2020/11/16/using-hashcat-rules-to-create-custom-wordlists/

hashcat --force password.txt -r /usr/share/hashcat/rules/best64.rule --stdout | sort -u > hashcat_words.txt

hashcat -m 3200 hash hashcat_words.txt
```

![hashcatPass](https://github.com/b1d0ws/OSCP/assets/58514930/4aa121ca-8dd2-4256-94b4-d14a48b270f2)
