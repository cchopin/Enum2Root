# Methodology coverage

- **0 — Reconnaissance:** passive OSINT, environment prep, active nmap scanning.
- **1 — Service enumeration:** FTP, SSH, SMTP, DNS, Web, Kerberos, MSRPC, SMB/NetBIOS, SNMP,
  LDAP, MSSQL, MySQL, NFS, RDP, WinRM, Redis, PostgreSQL, Oracle, rsync, Tomcat, Jenkins, Docker,
  AJP/Ghostcat, MongoDB, VNC, IPMI.
- **2 — Web attacks:** SQLi, XSS, LFI/path traversal, form brute force, WordPress, command
  injection, SSTI, file upload.
- **3 — Active Directory:** credential acquisition, domain enumeration, Kerberoast/AS-REP,
  ACL/delegation abuse, ADCS, coercion/NTLM relay, lateral movement, tickets & delegation, Shadow
  Credentials, domain dominance.
- **4 — Post-exploitation:** Linux & Windows privesc, looting & persistence.
- **5 — Shells, transfer & pivot:** reverse shells & TTY stabilization, file transfer, tunneling
  (ligolo-ng, chisel, SSH), hash cracking reference.

Generic technique sections (recon, web, Active Directory, post-exploitation) are always visible
since they aren't tied to a single port; per-service sections are filtered by the nmap import.
