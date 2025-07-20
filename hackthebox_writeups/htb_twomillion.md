About TwoMillion

TwoMillion is an Easy difficulty Linux box that was released to celebrate reaching 2 million users on HackTheBox. The box features an old version of the HackTheBox platform that includes the old hackable invite code. After hacking the invite code an account can be created on the platform. The account can be used to enumerate various API endpoints, one of which can be used to elevate the user to an Administrator. With administrative access the user can perform a command injection in the admin VPN generation endpoint thus gaining a system shell. An .env file is found to contain database credentials and owed to password re-use the attackers can login as user admin on the box. The system kernel is found to be outdated and CVE-2023-0386 can be used to gain a root shell. 



I start with nmap scan
```bash
nmap -p- -T4-min-rate=10000 10.10.11.221

Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-14 06:54 EDT
Warning: 10.10.11.221 giving up on port because retransmission cap hit (6).
Nmap scan report for 10.10.11.221
Host is up (0.068s latency).
Not shown: 65525 closed tcp ports (reset)
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    open     http
8921/tcp  filtered unknown
21782/tcp filtered unknown
28245/tcp filtered unknown
34353/tcp filtered unknown
51566/tcp filtered unknown
61431/tcp filtered unknown
62636/tcp filtered unknown
63774/tcp filtered unknown
```
Decided to do a more specific scan on the ports.
```bash 
nmap -p 22,80,8921,21782,28245,34353,51566,61431,62636,63774 -sC -sV -A -vv 10.10.11.221

PORT      STATE  SERVICE REASON         VERSION
22/tcp    open   ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJ+m7rYl1vRtnm789pH3IRhxI4CNCANVj+N5kovboNzcw9vHsBwvPX3KYA3cxGbKiA0VqbKRpOHnpsMuHEXEVJc=
|   256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOtuEdoYxTohG80Bo6YCqSzUY9+qbnAFnhsk4yAZNqhM
80/tcp    open   http    syn-ack ttl 63 nginx
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://2million.htb/
```
We have to throw 2million.htb to the /etc/hosts
I did some enumerating with ffuf
```bash
ffuf -u http://2million.htb/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100 -mc 302,303,403  
```
The site directed all "not available" directories so we had to do some filtering to try to catch the more important files in the site.
From the game description we can tell that /invite api js is exploitable
