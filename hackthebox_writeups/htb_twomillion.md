I start with an nmap scan
```bash
nmap -p- -T4-min-rate=10000 10.10.11.221
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-02 14:45 EEST
Nmap scan report for 10.10.11.221
Host is up (0.074s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 51.26 seconds
```

```bash
ffuf -w /usr/share/wordlists/dirb/big.txt -u http://2million.htb/FUZZ -fc 301 -s     

404
api
home
invite
login
logout
register
```
I know that the exploitable thing is the /invite api.

