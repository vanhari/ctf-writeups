```bash
════════════════════════════════════════════════════════
  RECON SUMMARY                                           
  ════════════════════════════════════════════════════════

  Target:        10.129.44.23
  DNS:           none
  HTTP:          5985
  HTTPS:         none
  Generated:     2026-06-17 16:46:12

  ────────────────────────────────────────────────────────
  OPEN PORTS & SERVICES
  ────────────────────────────────────────────────────────
  PORT         STATE      SERVICE          VERSION
  ────────────────────────────────────────────────────────
  53/tcp       open       domain           Simple DNS Plus
  88/tcp       open       kerberos-sec     Microsoft Windows Kerberos (server time: 2026-06-18 03:41:52Z)
  135/tcp      open       msrpc            Microsoft Windows RPC
  139/tcp      open       netbios-ssn      Microsoft Windows netbios-ssn
  389/tcp      open       ldap             Microsoft Windows Active Directory LDAP (Domain: checkpoint.htb, Site: Default-First-Site-Name)
  445/tcp      open       microsoft-ds?    
  464/tcp      open       kpasswd5?        
  593/tcp      open       ncacn_http       Microsoft Windows RPC over HTTP 1.0
  636/tcp      open       ldapssl?         
  3268/tcp     open       ldap             Microsoft Windows Active Directory LDAP (Domain: checkpoint.htb, Site: Default-First-Site-Name)
  3269/tcp     open       globalcatLDAPssl? 
  5985/tcp     open       http             Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
  9389/tcp     open       mc-nmf           .NET Message Framing
  49664/tcp    open       msrpc            Microsoft Windows RPC
  49671/tcp    open       msrpc            Microsoft Windows RPC
  49672/tcp    open       msrpc            Microsoft Windows RPC
  49675/tcp    open       msrpc            Microsoft Windows RPC
  49676/tcp    open       ncacn_http       Microsoft Windows RPC over HTTP 1.0
  49680/tcp    open       msrpc            Microsoft Windows RPC
  49706/tcp    open       msrpc            Microsoft Windows RPC
  49715/tcp    open       msrpc            Microsoft Windows RPC

  ────────────────────────────────────────────────────────
  SUBDOMAINS  (0 found)
  ────────────────────────────────────────────────────────
  none

  ────────────────────────────────────────────────────────
  INTERESTING PATHS  (0 found)
  ────────────────────────────────────────────────────────
  none

  ════════════════════════════════════════════════════════
```
Ok thats a crazy amount of ports. 

```bash
nxc smb 10.129.44.23 -u alex.turner -p 'Checkpoint2024!'
SMB         10.129.44.23    445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:checkpoint.htb) (signing:True) (SMBv1:None)
SMB         10.129.44.23    445    DC01             [+] checkpoint.htb\alex.turner:Checkpoint2024! 
```

