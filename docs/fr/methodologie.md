# Couverture méthodologique

- **0 — Reconnaissance :** OSINT passif, préparation, scan nmap actif.
- **1 — Énumération des services :** FTP, SSH, SMTP, DNS, Web, Kerberos, MSRPC, SMB/NetBIOS,
  SNMP, LDAP, MSSQL, MySQL, NFS, RDP, WinRM, Redis, PostgreSQL, Oracle, rsync, Tomcat, Jenkins,
  Docker, AJP/Ghostcat, MongoDB, VNC, IPMI.
- **2 — Attaques web :** SQLi, XSS, LFI/traversal, brute force de formulaire, WordPress, command
  injection, SSTI, upload.
- **3 — Active Directory :** acquisition de creds, énumération, Kerberoast/AS-REP, abus
  d'ACL/délégations, ADCS, coercition/relais NTLM, mouvement latéral, tickets & délégations,
  Shadow Credentials, domination du domaine.
- **4 — Post-exploitation :** privesc Linux & Windows, pillage & persistance.
- **5 — Shells, transfert & pivot :** reverse shells & stabilisation TTY, transfert de fichiers,
  tunneling (ligolo-ng, chisel, SSH), référence de cassage de hash.

Les sections « techniques » (recon, web, Active Directory, post-exploitation) restent toujours
visibles car non liées à un port précis ; les sections par service sont filtrées par l'import nmap.
