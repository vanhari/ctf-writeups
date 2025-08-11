Nmap scan
```bash
nmap -p- -T4-min-rate=10000 10.10.71.237
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-27 00:17 EEST
Nmap scan report for 10.10.71.237
Host is up (0.23s latency).
Not shown: 65529 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
222/tcp  open  rsh-spx
1337/tcp open  waste
3000/tcp open  ppp
8080/tcp open  http-proxy
```
More specific scan
```
nmap -p 22,80,222,1337,3000,8080 -A 10.10.71.237
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-27 00:36 EEST
Nmap scan report for 10.10.71.237
Host is up (0.24s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 33:f0:03:36:26:36:8c:2f:88:95:2c:ac:c3:bc:64:65 (RSA)
|   256 4f:f3:b3:f2:6e:03:91:b2:7c:c0:53:d5:d4:03:88:46 (ECDSA)
|_  256 13:7c:47:8b:6f:f8:f4:6b:42:9a:f2:d5:3d:34:13:52 (ED25519)
80/tcp   open  http    nginx 1.4.6 (Ubuntu)
| http-git: 
|   10.10.71.237:80/.git/
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|     Remotes:
|       https://github.com/electerious/Lychee.git
|_    Project type: PHP application (guessed from .gitignore)
| http-robots.txt: 7 disallowed entries 
|_/data/ /dist/ /docs/ /php/ /plugins/ /src/ /uploads/
|_http-title: Lychee
|_http-server-header: nginx/1.4.6 (Ubuntu)
222/tcp  open  ssh     OpenSSH 9.0 (protocol 2.0)
| ssh-hostkey: 
|   256 be:cb:06:1f:33:0f:60:06:a0:5a:06:bf:06:53:33:c0 (ECDSA)
|_  256 9f:07:98:92:6e:fd:2c:2d:b0:93:fa:fe:e8:95:0c:37 (ED25519)
1337/tcp open  http    Golang net/http server
|_http-title: OliveTin
| fingerprint-strings: 
|   GenericLines: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Accept-Ranges: bytes
|     Content-Length: 3858
|     Content-Type: text/html; charset=utf-8
|     Date: Sat, 26 Jul 2025 21:36:10 GMT
|     Last-Modified: Wed, 19 Oct 2022 15:30:49 GMT
|     <!DOCTYPE html>
|     <html>
|     <head>
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>OliveTin</title>
|     <link rel = "stylesheet" type = "text/css" href = "style.css" />
|     <link rel = "shortcut icon" type = "image/png" href = "OliveTinLogo.png" />
|     <link rel = "apple-touch-icon" sizes="57x57" href="OliveTinLogo-57px.png" />
|     <link rel = "apple-touch-icon" sizes="120x120" href="OliveTinLogo-120px.png" />
|     <link rel = "apple-touch-icon" sizes="180x180" href="OliveTinLogo-180px.png" />
|     </head>
|     <body>
|     <main title = "main content">
|     <fieldset id = "section-switcher" title = "Sections">
|     <button id = "showActions">Actions</button>
|     <button id = "showLogs">Logs</but
|   HTTPOptions: 
|     HTTP/1.0 200 OK
|     Accept-Ranges: bytes
|     Content-Length: 3858
|     Content-Type: text/html; charset=utf-8
|     Date: Sat, 26 Jul 2025 21:36:11 GMT
|     Last-Modified: Wed, 19 Oct 2022 15:30:49 GMT
|     <!DOCTYPE html>
|     <html>
|     <head>
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>OliveTin</title>
|     <link rel = "stylesheet" type = "text/css" href = "style.css" />
|     <link rel = "shortcut icon" type = "image/png" href = "OliveTinLogo.png" />
|     <link rel = "apple-touch-icon" sizes="57x57" href="OliveTinLogo-57px.png" />
|     <link rel = "apple-touch-icon" sizes="120x120" href="OliveTinLogo-120px.png" />
|     <link rel = "apple-touch-icon" sizes="180x180" href="OliveTinLogo-180px.png" />
|     </head>
|     <body>
|     <main title = "main content">
|     <fieldset id = "section-switcher" title = "Sections">
|     <button id = "showActions">Actions</button>
|_    <button id = "showLogs">Logs</but
3000/tcp open  http    Golang net/http server
|_http-title:  Gitea: Git with a cup of tea
| fingerprint-strings: 
|   GenericLines, Help, RTSPRequest: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Cache-Control: no-store, no-transform
|     Content-Type: text/html; charset=UTF-8
|     Set-Cookie: i_like_gitea=32e270f4b9009954; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=_uIfXpUrknPzI1w2Ddrb_UjdoGs6MTc1MzU2NTc3MDQ4ODkxODE1MA; Path=/; Expires=Sun, 27 Jul 2025 21:36:10 GMT; HttpOnly; SameSite=Lax
|     Set-Cookie: macaron_flash=; Path=/; Max-Age=0; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Sat, 26 Jul 2025 21:36:10 GMT
|     <!DOCTYPE html>
|     <html lang="en-US" class="theme-">
|     <head>
|     <meta charset="utf-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <title> Gitea: Git with a cup of tea</title>
|     <link rel="manifest" href="data:application/json;base64,eyJuYW1lIjoiR2l0ZWE6IEdpdCB3aXRoIGEgY3VwIG9mIHRlYSIsInNob3J0X25hbWUiOiJHaXRlYTogR2l0IHdpdGggYSBjdXAgb2YgdGVhIiwic3RhcnRfdXJsIjoiaHR0cDovL2xvY2FsaG9zdDozMDAwLyIsImljb25zIjpbeyJzcmMiOiJodHRwOi
|   HTTPOptions: 
|     HTTP/1.0 405 Method Not Allowed
|     Cache-Control: no-store, no-transform
|     Set-Cookie: i_like_gitea=a2276cfb9b5ebdb5; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=JFD9jJhszi0dnYY0p6Ap3SSX_Qw6MTc1MzU2NTc3MTQzMjY4NDE2OA; Path=/; Expires=Sun, 27 Jul 2025 21:36:11 GMT; HttpOnly; SameSite=Lax
|     Set-Cookie: macaron_flash=; Path=/; Max-Age=0; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Sat, 26 Jul 2025 21:36:11 GMT
|_    Content-Length: 0
8080/tcp open  http    SimpleHTTPServer 0.6 (Python 3.6.9)
|_http-server-header: SimpleHTTP/0.6 Python/3.6.9
|_http-title: Welcome to nginx!
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port1337-TCP:V=7.95%I=7%D=7/27%Time=68854A49%P=x86_64-pc-linux-gnu%r(Ge
SF:nericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20t
SF:ext/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x
SF:20Request")%r(GetRequest,FCC,"HTTP/1\.0\x20200\x20OK\r\nAccept-Ranges:\
SF:x20bytes\r\nContent-Length:\x203858\r\nContent-Type:\x20text/html;\x20c
SF:harset=utf-8\r\nDate:\x20Sat,\x2026\x20Jul\x202025\x2021:36:10\x20GMT\r
SF:\nLast-Modified:\x20Wed,\x2019\x20Oct\x202022\x2015:30:49\x20GMT\r\n\r\
SF:n<!DOCTYPE\x20html>\n\n<html>\n\t<head>\n\n\t\t<meta\x20name=\"viewport
SF:\"\x20content=\"width=device-width,\x20initial-scale=1\.0\">\n\n\t\t<ti
SF:tle>OliveTin</title>\n\t\t<link\x20rel\x20=\x20\"stylesheet\"\x20type\x
SF:20=\x20\"text/css\"\x20href\x20=\x20\"style\.css\"\x20/>\n\t\t<link\x20
SF:rel\x20=\x20\"shortcut\x20icon\"\x20type\x20=\x20\"image/png\"\x20href\
SF:x20=\x20\"OliveTinLogo\.png\"\x20/>\n\n\t\t<link\x20rel\x20=\x20\"apple
SF:-touch-icon\"\x20sizes=\"57x57\"\x20href=\"OliveTinLogo-57px\.png\"\x20
SF:/>\n\t\t<link\x20rel\x20=\x20\"apple-touch-icon\"\x20sizes=\"120x120\"\
SF:x20href=\"OliveTinLogo-120px\.png\"\x20/>\n\t\t<link\x20rel\x20=\x20\"a
SF:pple-touch-icon\"\x20sizes=\"180x180\"\x20href=\"OliveTinLogo-180px\.pn
SF:g\"\x20/>\n\t</head>\n\n\t<body>\n\t\t<main\x20title\x20=\x20\"main\x20
SF:content\">\n\t\t\t<fieldset\x20id\x20=\x20\"section-switcher\"\x20title
SF:\x20=\x20\"Sections\">\n\t\t\t\t<button\x20id\x20=\x20\"showActions\">A
SF:ctions</button>\n\t\t\t\t<button\x20id\x20=\x20\"showLogs\">Logs</but")
SF:%r(HTTPOptions,FCC,"HTTP/1\.0\x20200\x20OK\r\nAccept-Ranges:\x20bytes\r
SF:\nContent-Length:\x203858\r\nContent-Type:\x20text/html;\x20charset=utf
SF:-8\r\nDate:\x20Sat,\x2026\x20Jul\x202025\x2021:36:11\x20GMT\r\nLast-Mod
SF:ified:\x20Wed,\x2019\x20Oct\x202022\x2015:30:49\x20GMT\r\n\r\n<!DOCTYPE
SF:\x20html>\n\n<html>\n\t<head>\n\n\t\t<meta\x20name=\"viewport\"\x20cont
SF:ent=\"width=device-width,\x20initial-scale=1\.0\">\n\n\t\t<title>OliveT
SF:in</title>\n\t\t<link\x20rel\x20=\x20\"stylesheet\"\x20type\x20=\x20\"t
SF:ext/css\"\x20href\x20=\x20\"style\.css\"\x20/>\n\t\t<link\x20rel\x20=\x
SF:20\"shortcut\x20icon\"\x20type\x20=\x20\"image/png\"\x20href\x20=\x20\"
SF:OliveTinLogo\.png\"\x20/>\n\n\t\t<link\x20rel\x20=\x20\"apple-touch-ico
SF:n\"\x20sizes=\"57x57\"\x20href=\"OliveTinLogo-57px\.png\"\x20/>\n\t\t<l
SF:ink\x20rel\x20=\x20\"apple-touch-icon\"\x20sizes=\"120x120\"\x20href=\"
SF:OliveTinLogo-120px\.png\"\x20/>\n\t\t<link\x20rel\x20=\x20\"apple-touch
SF:-icon\"\x20sizes=\"180x180\"\x20href=\"OliveTinLogo-180px\.png\"\x20/>\
SF:n\t</head>\n\n\t<body>\n\t\t<main\x20title\x20=\x20\"main\x20content\">
SF:\n\t\t\t<fieldset\x20id\x20=\x20\"section-switcher\"\x20title\x20=\x20\
SF:"Sections\">\n\t\t\t\t<button\x20id\x20=\x20\"showActions\">Actions</bu
SF:tton>\n\t\t\t\t<button\x20id\x20=\x20\"showLogs\">Logs</but");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port3000-TCP:V=7.95%I=7%D=7/27%Time=68854A49%P=x86_64-pc-linux-gnu%r(Ge
SF:nericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20t
SF:ext/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x
SF:20Request")%r(GetRequest,2DEE,"HTTP/1\.0\x20200\x20OK\r\nCache-Control:
SF:\x20no-store,\x20no-transform\r\nContent-Type:\x20text/html;\x20charset
SF:=UTF-8\r\nSet-Cookie:\x20i_like_gitea=32e270f4b9009954;\x20Path=/;\x20H
SF:ttpOnly;\x20SameSite=Lax\r\nSet-Cookie:\x20_csrf=_uIfXpUrknPzI1w2Ddrb_U
SF:jdoGs6MTc1MzU2NTc3MDQ4ODkxODE1MA;\x20Path=/;\x20Expires=Sun,\x2027\x20J
SF:ul\x202025\x2021:36:10\x20GMT;\x20HttpOnly;\x20SameSite=Lax\r\nSet-Cook
SF:ie:\x20macaron_flash=;\x20Path=/;\x20Max-Age=0;\x20HttpOnly;\x20SameSit
SF:e=Lax\r\nX-Frame-Options:\x20SAMEORIGIN\r\nDate:\x20Sat,\x2026\x20Jul\x
SF:202025\x2021:36:10\x20GMT\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang=\"en
SF:-US\"\x20class=\"theme-\">\n<head>\n\t<meta\x20charset=\"utf-8\">\n\t<m
SF:eta\x20name=\"viewport\"\x20content=\"width=device-width,\x20initial-sc
SF:ale=1\">\n\t<title>\x20Gitea:\x20Git\x20with\x20a\x20cup\x20of\x20tea</
SF:title>\n\t<link\x20rel=\"manifest\"\x20href=\"data:application/json;bas
SF:e64,eyJuYW1lIjoiR2l0ZWE6IEdpdCB3aXRoIGEgY3VwIG9mIHRlYSIsInNob3J0X25hbWU
SF:iOiJHaXRlYTogR2l0IHdpdGggYSBjdXAgb2YgdGVhIiwic3RhcnRfdXJsIjoiaHR0cDovL2
SF:xvY2FsaG9zdDozMDAwLyIsImljb25zIjpbeyJzcmMiOiJodHRwOi")%r(Help,67,"HTTP/
SF:1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20charse
SF:t=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request")%r(HTTPOp
SF:tions,1C2,"HTTP/1\.0\x20405\x20Method\x20Not\x20Allowed\r\nCache-Contro
SF:l:\x20no-store,\x20no-transform\r\nSet-Cookie:\x20i_like_gitea=a2276cfb
SF:9b5ebdb5;\x20Path=/;\x20HttpOnly;\x20SameSite=Lax\r\nSet-Cookie:\x20_cs
SF:rf=JFD9jJhszi0dnYY0p6Ap3SSX_Qw6MTc1MzU2NTc3MTQzMjY4NDE2OA;\x20Path=/;\x
SF:20Expires=Sun,\x2027\x20Jul\x202025\x2021:36:11\x20GMT;\x20HttpOnly;\x2
SF:0SameSite=Lax\r\nSet-Cookie:\x20macaron_flash=;\x20Path=/;\x20Max-Age=0
SF:;\x20HttpOnly;\x20SameSite=Lax\r\nX-Frame-Options:\x20SAMEORIGIN\r\nDat
SF:e:\x20Sat,\x2026\x20Jul\x202025\x2021:36:11\x20GMT\r\nContent-Length:\x
SF:200\r\n\r\n")%r(RTSPRequest,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nC
SF:ontent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\
SF:n\r\n400\x20Bad\x20Request");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
Network Distance: 4 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 8080/tcp)
HOP RTT       ADDRESS
1   215.47 ms 10.6.0.1
2   ... 3
4   293.22 ms 10.10.71.237

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 47.64 seconds
```
Some important stuff here:
Git repository found 10.10.71.237:80/.git/ 

| http-robots.txt: 7 disallowed entries 
|_/data/ /dist/ /docs/ /php/ /plugins/ /src/ /uploads/

Website on 80,1337,3000,8080

The website on port 80 has alot of pictures so i decided to check the metadata of them.
