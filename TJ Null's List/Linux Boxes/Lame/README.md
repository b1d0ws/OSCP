## Lame Write-Up

Difficulty: Easy

```
nmap 10.10.10.3 -T4 -Pn
```

![nmap](https://user-images.githubusercontent.com/58514930/218098192-b078b6be-11ba-4c8a-83fd-db954bea751e.png)

<br>

```
nmap 10.10.10.3 -T4 -Pn -p 139,445 -A
```

![agressive nmap](https://user-images.githubusercontent.com/58514930/218098663-48af2afe-9427-4080-86da-eb19c5ee31be.png)

<br>

Searching for Samba 3.0.20-Debian, we found this [exploit](https://github.com/amdsyad/exploit-smb-3.0.20/blob/master/exploit-smb-3.0.20.py).

![exploit](https://user-images.githubusercontent.com/58514930/218098982-3eef727f-c1ba-4790-84bd-758fed3e847f.png)

<br>

Generate a new payload with msfvenom with out LHOST and get the shell.

![gettingShell](https://user-images.githubusercontent.com/58514930/218099355-b99c0f81-2a25-494b-af28-ca4a80f6e147.png)

<br>

We are already root, just get the flags.
