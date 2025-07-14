I did an nmap scan
```bash
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

nmap -p 22,80 -sC -sV -A 10.10.11.58
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.12 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 97:2a:d2:2c:89:8a:d3:ed:4d:ac:00:d2:1e:87:49:a7 (RSA)
|   256 27:7c:3c:eb:0f:26:e9:62:59:0f:0f:b1:38:c9:ae:2b (ECDSA)
|_  256 93:88:47:4c:69:af:72:16:09:4c:ba:77:1e:3b:3b:eb (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-robots.txt: 22 disallowed entries (15 shown)
| /core/ /profiles/ /README.md /web.config /admin 
| /comment/reply /filter/tips /node/add /search /user/register 
|_/user/password /user/login /user/logout /?q=admin /?q=comment/reply
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-git: 
|   10.10.11.58:80/.git/
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|_    Last commit message: todo: customize url aliases.  reference:https://docs.backdro...
|_http-generator: Backdrop CMS 1 (https://backdropcms.org)
|_http-title: Home | Dog
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Ok se we have the ports 22 and 80 open. Next step is the check out the web page and do a gobuster scan. Before that I noticed that the site is powered by "Backdrop CMS" so that's something to keep in mind for sure
```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.11.58 -t 100 
Starting gobuster in directory enumeration mode
===============================================================
/themes               (Status: 301) [Size: 311] [--> http://10.10.11.58/themes/]
/modules              (Status: 301) [Size: 312] [--> http://10.10.11.58/modules/]
/sites                (Status: 301) [Size: 310] [--> http://10.10.11.58/sites/]
/core                 (Status: 301) [Size: 309] [--> http://10.10.11.58/core/]
/files                (Status: 301) [Size: 310] [--> http://10.10.11.58/files/]
/layouts              (Status: 301) [Size: 312] [--> http://10.10.11.58/layouts/]
/server-status        (Status: 403) [Size: 276]
Progress: 220560 / 220561 (100.00%)
```
After reading my notes more carefully we can see that there is a .git folder that reveals the sourcecode.
```bash
10.10.11.58:80/.git/
Git repository found!
```
Lets dump all of the data with git-dumper to a file
```bash
git-dumper http://10.10.11.58/.git/ ~/website
```

