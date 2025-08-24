Starting with a port scan
```bash
nmap -p- --min-rate=10000 10.10.193.184                                                      
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-21 14:10 EEST
Warning: 10.10.193.184 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.193.184
Host is up (0.22s latency).
Not shown: 65508 closed tcp ports (reset)
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    open     http
3244/tcp  filtered onesaf
3864/tcp  filtered asap-tcp-tls
5723/tcp  filtered omhs
7195/tcp  filtered unknown
7606/tcp  filtered mipi-debug
8050/tcp  filtered unknown
14573/tcp filtered unknown
17082/tcp filtered unknown
17291/tcp filtered unknown
18071/tcp filtered unknown
27298/tcp filtered unknown
27863/tcp filtered unknown
28660/tcp filtered unknown
34667/tcp filtered unknown
35241/tcp filtered unknown
38550/tcp filtered unknown
38922/tcp filtered unknown
39441/tcp filtered unknown
44192/tcp filtered unknown
46597/tcp filtered unknown
46736/tcp filtered unknown
49098/tcp filtered unknown
50481/tcp filtered unknown
52426/tcp filtered unknown
56370/tcp filtered unknown

Nmap done: 1 IP address (1 host up) scanned in 16.95 seconds


nmap -p 22,80,3244,3864,5723,7195                                                            
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-21 14:11 EEST
WARNING: No targets were specified, so 0 hosts scanned.
Nmap done: 0 IP addresses (0 hosts up) scanned in 0.01 seconds
                                                                                                                
┌──(kali㉿kali)-[~]
└─$ nmap -p 22,80,3244,3864,5723,7195 -sC -sV 10.10.193.184
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-21 14:11 EEST
Nmap scan report for 10.10.193.184
Host is up (0.31s latency).

PORT     STATE  SERVICE      VERSION
22/tcp   open   ssh          OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 34:32:0e:2e:ef:b0:78:ba:fd:51:64:4c:d8:a4:93:4f (RSA)
|   256 09:66:35:7a:b3:d3:68:e6:1c:00:67:c9:c2:ac:94:b0 (ECDSA)
|_  256 07:8c:d8:4e:ee:f1:94:79:87:33:1a:6e:2b:ad:81:6b (ED25519)
80/tcp   open   http         nginx 1.18.0 (Ubuntu)
|_http-title: Spicyo
|_http-server-header: nginx/1.18.0 (Ubuntu)
3244/tcp closed onesaf
3864/tcp closed asap-tcp-tls
5723/tcp closed omhs
7195/tcp closed unknown
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.31 seconds
```


