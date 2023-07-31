## Love

Difficulty: Easy

### User Flag

```
rustscan -a love.htb --ulimit 5000 -- -A -sV -sC
```

![rustscan](https://github.com/b1d0ws/OSCP/assets/58514930/9a31e1d8-6ac5-4e79-84c6-226e4241af8a)

<br>

A lot of ports are open, the most intersting ones are: 80, 443 and 5000.  

The nmap result of port 443, return us a subdomain called: stating.love.htb.  

The website on port 80 has a login form that we can SQL Inject, but nothing interesting will come up.

![nmapSubdomain](https://github.com/b1d0ws/OSCP/assets/58514930/dd2c9f3c-1dd8-47a8-8dcf-9cfc6bb9efea)

![website](https://github.com/b1d0ws/OSCP/assets/58514930/71fddd0c-da92-4a5d-88c0-e7a3e129defe)

<br>

If we try to access port 5000, we got a forbidden response.

![forbidden](https://github.com/b1d0ws/OSCP/assets/58514930/d96f15ee-8641-43e0-8432-3d3f842d3d63)

<br>

On subdomain, there is a functionality that scan file that we can insert URLs.  

We can abuse this to trigger SSRF to access port 5000 with 127.0.0.1:5000.  

![subdomainSSRF](https://github.com/b1d0ws/OSCP/assets/58514930/2f2d842d-a122-4cb5-8dfa-b40b1e25d7ff)

<br>

We find this credentials, but on the website the login form requests a Voter ID.  

Enumerating the website with GoBuster, we find /admin directory and there we can use this credentials.

![goBuster](https://github.com/b1d0ws/OSCP/assets/58514930/102945f5-54b5-4c91-8d12-11062eefaa8d)

![websiteAdmin](https://github.com/b1d0ws/OSCP/assets/58514930/d31dedce-941d-4484-be43-1296e07730d0)

### Root Flag

