This is my first windows based ctf

```bash
 Machine Information
As is common in real life Windows pentests, you will start the Fluffy box with credentials for the following account: j.fleischman / J0elTHEM4n1990!
```

I start it with a port scan
```bash
nmap -p- --min-rate=10000 10.10.11.69          
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-26 20:37 EEST
Nmap scan report for 10.10.11.69
Host is up (0.14s latency).
Not shown: 65517 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49511/tcp open  unknown
49546/tcp open  unknown
49667/tcp open  unknown
49685/tcp open  unknown
49687/tcp open  unknown
49690/tcp open  unknown
49704/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 28.41 seconds
                                                                                                                    
┌──(kali㉿kali)-[~]
└─$ nmap -p 53,88,139,380,445,593,636,3268,5985,9389 -sC -sV 10.10.11.69                         
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-26 20:43 EEST
Nmap scan report for 10.10.11.69
Host is up (0.084s latency).

PORT     STATE    SERVICE       VERSION
53/tcp   open     domain        Simple DNS Plus
88/tcp   open     kerberos-sec  Microsoft Windows Kerberos (server time: 2025-08-27 00:43:44Z)
139/tcp  open     netbios-ssn   Microsoft Windows netbios-ssn
380/tcp  filtered is99s
445/tcp  open     microsoft-ds?
593/tcp  open     ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open     ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-08-27T00:44:32+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: commonName=DC01.fluffy.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.fluffy.htb
| Not valid before: 2025-04-17T16:04:17
|_Not valid after:  2026-04-17T16:04:17
3268/tcp open     ldap          Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-08-27T00:44:32+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: commonName=DC01.fluffy.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.fluffy.htb
| Not valid before: 2025-04-17T16:04:17
|_Not valid after:  2026-04-17T16:04:17
5985/tcp open     http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open     mc-nmf        .NET Message Framing
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2025-08-27T00:43:52
|_  start_date: N/A
|_clock-skew: mean: 7h00m00s, deviation: 0s, median: 7h00m00s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 55.66 seconds
```

There is alot of services and im going to do a bit of research on each one of them.



