Port scanning
```bash
nmap -p- --min-rate=5000 10.10.243.254
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-30 00:13 EEST
Nmap scan report for 10.10.243.254
Host is up (0.23s latency).
Not shown: 65533 closed tcp ports (reset)
PORT      STATE SERVICE
22/tcp    open  ssh
30024/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 15.59 seconds
                                                                                                       
┌──(kali㉿kali)-[~]
└─$ nmap -p 22,30024 -sC -sV 10.10.243.254
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-30 00:13 EEST
Nmap scan report for 10.10.243.254
Host is up (0.28s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 57:26:0c:24:f0:9c:f3:49:bc:47:67:b3:66:3a:95:86 (RSA)
|   256 c6:91:f1:f0:fb:21:82:98:a4:ec:48:7a:8d:e0:03:15 (ECDSA)
|_  256 16:a4:a8:49:aa:1d:c1:17:5d:b3:60:44:aa:7a:41:27 (ED25519)
30024/tcp open  ftp     vsftpd 3.0.5
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.6.25.159
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 ftp      ftp          1743 Mar 23  2021 id_rsa
|_-rw-r--r--    1 ftp      ftp            78 Mar 23  2021 note.txt
Service Info: OSs: Linux, Unix; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.64 seconds
```

So there is ftp and ssh

In the ftp there was a note and id_rsa

```bash
 cat note.txt 
I always forget my password. Just let me store an ssh key here.
- errorcauser
```

and i bruteforced for the passphrase
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt john
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 1 for all loaded hashes
Cost 2 (iteration count) is 2 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
cupcake          (id_rsa)     
1g 0:00:00:00 DONE (2025-08-30 00:16) 25.00g/s 16000p/s 16000c/s 16000C/s mariah..pebbles
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```


We were ableto log in to the ssh.

there is a note
```bash
-bash-4.4$ cat note.txt
Hi Error!
I've set up a webserver locally so no one outside could access it.
It is for testing purposes only.  There are still a few things I need to do like setting up a custom theme.
You can check it out, you already know what to do.
-Cth
:)
```
There is a website running on localhost:631 which is just portforward and check out on my machine




