Port scanning

```bash
nmap -p- --min-rate=5000 10.10.201.96           
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-29 23:38 EEST
Nmap scan report for 10.10.201.96
Host is up (0.26s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 15.40 seconds
                                                                                                       
┌──(kali㉿kali)-[~]
└─$ nmap -p 22,8080 -sC -sV 10.10.201.96              
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-29 23:38 EEST
Nmap scan report for 10.10.201.96
Host is up (0.23s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 73:75:85:f6:df:11:83:14:4f:52:3a:fd:03:c9:eb:ae (RSA)
|   256 85:5c:83:3b:0c:3d:f3:17:8e:7f:b1:fd:10:38:57:b6 (ECDSA)
|_  256 b9:80:08:97:3f:f1:c2:c5:59:12:99:e0:ec:48:45:ec (ED25519)
8080/tcp open  http    Node.js Express framework
|_http-title: VulnNet &ndash; Your reliable news source &ndash; Try Now!
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.86 seconds
```

SO there is an ssh and a website. 

I decided to use burpsuite and checks if there is something there. 

I decoded the cookies into readable format

```bash
{"username":"Guest","isGuest":true,"encoding": "utf-8"}
```

I feel like this is something we could exploit
