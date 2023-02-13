## Nibbles

Difficulty: Easy

### User Flag

```
rustscan -a 10.10.10.75 --ulimit 5000 -- -A -sC
```

![rustscan](https://user-images.githubusercontent.com/58514930/218508727-f249844a-3cb8-4301-8d32-da5da1b19b39.png)

<br>

Checking the source code we find a nibbleblog directory.

![sourceCode](https://user-images.githubusercontent.com/58514930/218508943-313e1456-1ad3-4b5e-920a-c53e5388b886.png)

Searching for nibbleblog we discover two important things:
1. This CMS is vulnerable to Arbitrary File Upload (CVE-2015-6967).
2. The default credentials of it is admin:nibbles

We can use this [exploit](https://github.com/dix0nym/CVE-2015-6967) to get initial access.

![exploit](https://user-images.githubusercontent.com/58514930/218512123-ba5cec6f-090c-419a-b64f-b98c5c188cea.png)

![userFlag](https://user-images.githubusercontent.com/58514930/218512370-aee31d5b-2a6c-4567-9c20-ee6ff2bfa4eb.png)

### Root Flag

Doing some enumeration with sudo -l we see that our user can use /home/nibbler/personal/stuff/monitor.sh as root.

![sudo -l](https://user-images.githubusercontent.com/58514930/218512466-d7b8eb2f-7e1e-44c0-8f96-eb63aec6e3c4.png)

Just unzip personal.zip, put a reverse shell inside this .sh file and execute it with root.

![gettingRoot](https://user-images.githubusercontent.com/58514930/218512828-029ea66c-a9f3-4bfc-9219-ed0814945d66.png)

![rootFlag](https://user-images.githubusercontent.com/58514930/218512845-60ecdf91-f4f5-4a44-bf7e-76eeecd62f70.png)
