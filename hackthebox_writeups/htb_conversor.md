I start the machine with a quick port scan.

```bash
┌──(kali㉿kali)-[~]
└─$ nmap -p- --min-rate=5000 10.10.11.92                                
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-05 07:07 EST
Warning: 10.10.11.92 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.11.92
Host is up (0.084s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

                                                                                                       
┌──(kali㉿kali)-[~]
└─$ nmap -p 22,80 -sC -sV 10.10.11.92
Starting Nmap 7.95 ( https://nmap.org ) at 2025-11-05 07:08 EST
Nmap scan report for 10.10.11.92
Host is up (0.36s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 01:74:26:39:47:bc:6a:e2:cb:12:8b:71:84:9c:f8:5a (ECDSA)
|_  256 3a:16:90:dc:74:d8:e3:c4:51:36:e2:08:06:26:17:ee (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Did not follow redirect to http://conversor.htb/
Service Info: Host: conversor.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.94 seconds
```
Ok se we have ssh and a website open. The site seems to be running apache 2.4.52

The website just has a login screen. I decided to create and account and see whats going on. 
It's just a website where you can upload xml and xslt files to make it into more aeshetic format.

I did some fuzzing 
```bash
bout                   [Status: 200, Size: 2842, Words: 577, Lines: 81, Duration: 333ms]
login                   [Status: 200, Size: 722, Words: 30, Lines: 22, Duration: 72ms]
register                [Status: 200, Size: 726, Words: 30, Lines: 21, Duration: 70ms]
javascript              [Status: 301, Size: 319, Words: 20, Lines: 10, Duration: 59ms]
logout                  [Status: 302, Size: 199, Words: 18, Lines: 6, Duration: 289ms]
convert                 [Status: 405, Size: 153, Words: 16, Lines: 6, Duration: 270ms]
                        [Status: 302, Size: 199, Words: 18, Lines: 6, Duration: 78ms]
server-status           [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 363ms]
:: Progress: [220560/220560] :: Job [1/1] :: 192 req/sec :: Duration: [0:26:13] :: Errors: 0 ::
```

I went to the about page and found out that the source code is public and downloadable.

I got the app.py file and the secretkey is "Changemeplease"

```bash
cat install.md 
To deploy Conversor, we can extract the compressed file:

"""
tar -xvf source_code.tar.gz
"""

We install flask:

"""
pip3 install flask
"""

We can run the app.py file:

"""
python3 app.py
"""

You can also run it with Apache using the app.wsgi file.

If you want to run Python scripts (for example, our server deletes all files older than 60 minutes to avoid system overload), you can add the following line to your /etc/crontab.

"""
* * * * * www-data for f in /var/www/conversor.htb/scripts/*.py; do python3 "$f"; done
"""
```

Ok so in the last line its saying that the server executes all the python scripts located in the /scripts/.py location. We just have to find a way to get our reverse shell script in there for example. 

To be quite frank I have no clue but I'll find out a way. Im guessing we can do it via the xml and xslt file.


