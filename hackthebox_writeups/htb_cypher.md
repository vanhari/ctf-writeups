Let's start with a port scan
```bash
nmap -p- -T4-min-rate=10000 10.10.11.57
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-26 08:18 EDT
Warning: 10.10.11.57 giving up on port because retransmission cap hit (6).
Nmap scan report for 10.10.11.57
Host is up (0.052s latency).
Not shown: 65518 closed tcp ports (reset)
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    open     http
7598/tcp  filtered unknown
9468/tcp  filtered unknown
11608/tcp filtered unknown
14222/tcp filtered unknown
17023/tcp filtered unknown
17187/tcp filtered unknown
17443/tcp filtered unknown
20243/tcp filtered unknown
21020/tcp filtered unknown
26364/tcp filtered unknown
31754/tcp filtered unknown
51510/tcp filtered unknown
57917/tcp filtered unknown
58823/tcp filtered unknown
63789/tcp filtered unknown
```
There is alot of stuff. Lets do a more specific scan.
```bash
nmap -p 22,80 -sC -sV -A 10.10.11.57
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-26 08:29 EDT
Nmap scan report for 10.10.11.57
Host is up (0.061s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 be:68:db:82:8e:63:32:45:54:46:b7:08:7b:3b:52:b0 (ECDSA)
|_  256 e5:5b:34:f5:54:43:93:f8:7e:b6:69:4c:ac:d6:3d:23 (ED25519)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
|_http-title: Did not follow redirect to http://cypher.htb/
|_http-server-header: nginx/1.24.0 (Ubuntu)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
I found a possible username in the source "TheFunay1" and this comment " // TODO: don't store user accounts in neo4j"
Also from the /testing we could download a .jar file with some stuff..

______________________TOBECONTINUED
