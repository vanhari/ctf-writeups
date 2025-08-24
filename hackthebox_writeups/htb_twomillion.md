I start with an nmap scan
```bash
nmap -p- -T4-min-rate=10000 10.10.11.221
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-02 14:45 EEST
Nmap scan report for 10.10.11.221
Host is up (0.074s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 51.26 seconds
```

```bash
ffuf -w /usr/share/wordlists/dirb/big.txt -u http://2million.htb/FUZZ -fc 301 -s     

404
api
home
invite
login
logout
register
```
I know that the exploitable thing is the /invite api since it was told in the description that the it's vulnerable

It calls for the inviteapi.min.js and we can make it more readable with de4js

```bash
function verifyInviteCode(code) {
    var formData = {
        "code": code
    };
    $.ajax({
        type: "POST",
        dataType: "json",
        data: formData,
        url: '/api/v1/invite/verify',
        success: function(response) {
            console.log(response)
        },
        error: function(response) {
            console.log(response)
        }
    })
}

function makeInviteCode() {
    $.ajax({
        type: "POST",
        dataType: "json",
        url: '/api/v1/invite/how/to/generate',
        success: function(response) {
            console.log(response)
        },
        error: function(response) {
            console.log(response)
        }
    })
}
```

```bash
curl -X POST http://2million.htb/api/v1/invite/how/to/generate
{"0":200,"success":1,"data":{"data":"Va beqre gb trarengr gur vaivgr pbqr, znxr n CBFG erdhrfg gb \/ncv\/i1\/vaivgr\/trarengr","enctype":"ROT13"},"hint":"Data is encrypted ... We should probbably check the encryption type in order to decrypt it..."}
```
After decoding the message "In order to generate the invite code, make a POST request to \/api\/v1\/invite\/generate" we weill lrean how to generate the code.


```bash
┌──(kali㉿kali)-[~]
└─$ curl -X POST http://2million.htb/api/v1/invite/generate       
{"0":200,"success":1,"data":{"code":"QzFXUVQtM081WFYtRFBXTFgtUkZYMEc=","format":"encoded"}}
```
And now we have the code "C1WQT-3O5XV-DPWLX-RFX0G"


