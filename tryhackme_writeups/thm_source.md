Quick port scan
```bash
┌──(kali㉿kali)-[~/Downloads]
└─$ nmap -p- --min-rate=10000 10.10.91.197
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-18 15:16 EEST
Warning: 10.10.91.197 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.91.197
Host is up (0.23s latency).
Not shown: 65533 closed tcp ports (reset)
PORT      STATE SERVICE
22/tcp    open  ssh
10000/tcp open  snet-sensor-mgmt

Nmap done: 1 IP address (1 host up) scanned in 20.62 seconds
                                                                                            
┌──(kali㉿kali)-[~/Downloads]
└─$ nmap -p 22,10000 -sC -sV 10.10.91.197 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-18 15:17 EEST
Nmap scan report for 10.10.91.197
Host is up (0.32s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b7:4c:d0:bd:e2:7b:1b:15:72:27:64:56:29:15:ea:23 (RSA)
|   256 b7:85:23:11:4f:44:fa:22:00:8e:40:77:5e:cf:28:7c (ECDSA)
|_  256 a9:fe:4b:82:bf:89:34:59:36:5b:ec:da:c2:d3:95:ce (ED25519)
10000/tcp open  http    MiniServ 1.890 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 38.88 seconds
```
Ok so there is SSH and a website on the port 10000. Its running MiniServ 1.890.

Ok so I found and RCE (Remote code execution) for the MiniServ 
https://github.com/foxsin34/WebMin-1.890-Exploit-unauthorized-RCE

When running the RCE for whoami i get that the account is Root. Not sure if its a containter or not but we can use it for our advantage. I was unable to get a revshell so i tried msfconsole.

Ok now we got a proper shell.

And since we already have root access we can just get both flags easily

