I started by doing an nmap scan
```bash
nmap -p- -sC -sV -vv -T4-min-rate=10000 10.10.246.38
```
And the results were
```bash
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 61 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 90:dd:c8:f1:20:35:33:ea:f0:8d:48:72:9a:52:33:19 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDLVXpmydSeynI8zUJ5zi5p54q78nhQR1r/ctNiVcx/WO8Jpz2rIxc9RNS/ENV6nF8IKpnkcmB0+pGtoaAgLJ1rCQFvdY3tV44F1Gh3K5QubpnyxjuzLF8YuG9TesiMIV0gutynIM+wPm1vmvJOhey8oXOqLxqIEazUBy7dzuk8cX1p+LkybF+dNq7KZoT9uLxAMg02MIDX4BQ1Xc5bNyYfKJ5H8xbtYuRdef6Kd4b7vF8UVRl90iX4HVGW07lsPcWqpU4aFI0rnTLD7tUqHFVAsIOUEtJc/qVoHTmCKx7vtZ4qTQgTZQc5RRzpaBSwBoUjSfORDWd/5SP9uQON9LJ4lD/Pb8zl6qB2SNAwmNsB78Hu15Fy6F4/2jzyoI2R06lREb5QAK9YQRCgPULIMGEJVZ6PoLVjelHZhil/7XoUKHR3RhLfg4FX0VHWmJs0gb/pS3+pfzZrDbjc+Mo5rGYqTCCHZpRA4ARoth3EO9BJtcWWA8Xz1qcXlkYEc/OSpoc=
|   256 ae:a3:83:fd:6d:3d:82:a3:bd:e1:f6:35:d1:5c:53:d5 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJKlR5zsVR/THut/Nscw/m1ezJ1XNsT1Xt/WUZoQ4wWneE5PdZUeafJ9QQAC240Trz7uiEBAtLCh7Xdz3hK0+hs=
|   256 1d:3f:0a:3d:c4:77:3a:be:05:8d:1c:91:cf:74:5c:a4 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOLWnUq16A8RMgReM/Mj7bODW0v35LvJEInrBrL0GXf7
80/tcp open  http    syn-ack ttl 60 Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Publishers Pulse: SPIP Insights & Tips
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```


