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


