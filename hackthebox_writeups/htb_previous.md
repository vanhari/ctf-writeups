I start with a port scan

```bash
nmap -p- --min-rate=10000 10.10.11.83
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-27 13:07 EEST
Nmap scan report for 10.10.11.83
Host is up (0.072s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 10.06 seconds
                                                                                                    
┌──(kali㉿kali)-[~]
└─$ nmap -p 22,80 -sC -sV 10.10.11.83                                   
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-27 13:08 EEST
Nmap scan report for 10.10.11.83
Host is up (0.093s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://previous.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.87 seconds
```

Ok so there is an ssh and a http. 

```bash
gobuster dir -w /usr/share/wordlists/dirb/big.txt  -u http://previous.htb/ -t 200
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://previous.htb/
[+] Method:                  GET
[+] Threads:                 200
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/api4                 (Status: 307) [Size: 36] [--> /api/auth/signin?callbackUrl=%2Fapi4]
/api3                 (Status: 307) [Size: 36] [--> /api/auth/signin?callbackUrl=%2Fapi3]
/api2                 (Status: 307) [Size: 36] [--> /api/auth/signin?callbackUrl=%2Fapi2]
/apis                 (Status: 307) [Size: 36] [--> /api/auth/signin?callbackUrl=%2Fapis]
/apicache             (Status: 307) [Size: 40] [--> /api/auth/signin?callbackUrl=%2Fapicache]
/api_test             (Status: 307) [Size: 40] [--> /api/auth/signin?callbackUrl=%2Fapi_test]
/api-doc              (Status: 307) [Size: 39] [--> /api/auth/signin?callbackUrl=%2Fapi-doc]
/api                  (Status: 307) [Size: 35] [--> /api/auth/signin?callbackUrl=%2Fapi]
/apimage              (Status: 307) [Size: 39] [--> /api/auth/signin?callbackUrl=%2Fapimage]
/cgi-bin/             (Status: 308) [Size: 8] [--> /cgi-bin]
/docs                 (Status: 307) [Size: 36] [--> /api/auth/signin?callbackUrl=%2Fdocs]
/docs2                (Status: 307) [Size: 37] [--> /api/auth/signin?callbackUrl=%2Fdocs2]
/docs51               (Status: 307) [Size: 38] [--> /api/auth/signin?callbackUrl=%2Fdocs51]
/docs41               (Status: 307) [Size: 38] [--> /api/auth/signin?callbackUrl=%2Fdocs41]
/signin               (Status: 200) [Size: 3481]
Progress: 20469 / 20470 (100.00%)
===============================================================
Finished
===============================================================

```

There is an email "jeremy@previous.htb" so atleast we have a name for something.

