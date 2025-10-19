port scanning

```bash
nmap -p- --min-rate=5000 10.10.11.85  
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-15 07:33 EDT
Nmap scan report for 10.10.11.85
Host is up (0.077s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 14.78 seconds
                                                                                                                    
┌──(kali㉿kali)-[~]
└─$ nmap -p 22,80 -sC -sV 10.10.11.85     
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-15 07:33 EDT
Nmap scan report for 10.10.11.85
Host is up (0.068s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)
| ssh-hostkey: 
|   256 95:62:ef:97:31:82:ff:a1:c6:08:01:8c:6a:0f:dc:1c (ECDSA)
|_  256 5f:bd:93:10:20:70:e6:09:f1:ba:6a:43:58:86:42:66 (ED25519)
80/tcp open  http    nginx 1.22.1
|_http-server-header: nginx/1.22.1
|_http-title: Did not follow redirect to http://hacknet.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.20 seconds
```

We have ssh and a website. Im going to enumerate the machine for possible hidden directories. 

I let the scans run and i tried to make a username 
test@gmail.com:test:test
and it said that it was already use. That may be useful

I created a temp user of
vanha@gmail.com:vanha



