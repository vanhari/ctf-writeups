Port scanning
```bash
 nmap -p- -min-rate=10000 10.10.147.189                  
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-14 17:53 EEST
Warning: 10.10.147.189 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.147.189
Host is up (0.23s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE
85/tcp open  mit-ml-dev

Nmap done: 1 IP address (1 host up) scanned in 17.93 seconds
                                                                                                       
┌──(kali㉿kali)-[~]
└─$ nmap -p 85 -sC -sV  10.10.147.189
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-14 17:54 EEST
Nmap scan report for 10.10.147.189
Host is up (0.23s latency).

PORT   STATE SERVICE VERSION
85/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-title: 0H N0! PWN3D 4G4IN
|_http-server-header: Apache/2.4.7 (Ubuntu)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.30 seconds
```
Ok so we have just a website nothing much to go off
THere is a website where we can fuzz for subdirectories
```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  -u http://10.10.147.189:85/ -t 150 -x php,html
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.147.189:85/
[+] Method:                  GET
[+] Threads:                 150
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 647]
/.html                (Status: 403) [Size: 285]
/.php                 (Status: 403) [Size: 284]
/app                  (Status: 301) [Size: 314] [--> http://10.10.147.189:85/app/]
/.php                 (Status: 403) [Size: 284]
/.html                (Status: 403) [Size: 285]
Progress: 214130 / 661683 (32.36%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 214406 / 661683 (32.40%)
===============================================================
Finished
===============================================================
```
I found an login panel and for some reason the admin had default logins of
admin:password
Now that i have access to the admin panel i was able to upload a reverse shell in there and gain access to their machine.

I transported linpeas over to automate the scan,
And the first thing i see is
```bash
╔══════════╣ Searching passwords in config PHP files
/var/www/html/app/castle/application/config/database.php:            'password' => 'toadisthebest',
```
We can't use ssh since it's not allowed but we can still log in no problem

I decided to rerun the linpeas but now as the user Toad since we have more priviledges than www.data
I stumbled upon this
```bash
PWD_token=aWthVGVOVEFOdEVTCg==
And it translates to

echo "aWthVGVOVEFOdEVTCg==" | base64 -d    

ikaTeNTANtES
```
And it was the password for Mario

I still can't read the user.txt since it was root permissions which is quite odd. But there is also mysql so we might check that out

I got into the sql with
```bash
mario@mkingdom:/var/www/html/app/castle/application/config$ mysql -u toad -h localhost -p
<l/app/castle/application/config$ mysql -u toad -h localhost -p              
Enter password: toadisthebest

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 99
```
The sql had nothing interesting. I was stuck on this part for a while. but there were stuff running on root access.

We were able to put our own ip in to the hosts file and when the server runs the scripts it directs it to our own website

This following explanation is going to most likely make 0 sense since i was so exhausted.

But basically we changed the /etc/hosts file to make the ip of the site the crontab was accessing to be ours. And it was accessing a bash file so i replicated that exact file but the insides of it included a rev shell
and since it was executed by root the revshell had root access. But for some reason i still wasnt able to read the flags so i had to move them to the /tmp folder and then was I able to read them

