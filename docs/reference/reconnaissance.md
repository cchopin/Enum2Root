# 0 - Reconnaissance

La phase de reconnaissance vise à cartographier la surface d'attaque de la cible avant toute interaction intrusive. On commence par de la collecte passive (aucun paquet envoyé à la cible), on prépare l'environnement de test (horloge, résolution de noms), puis on réalise un scan de ports actif pour identifier les services exposés. Les résultats orientent toutes les phases suivantes.

!!! note "Jetons"
    Les commandes ci-dessous utilisent des variables substituées automatiquement par l'application : `{IP}` (adresse IP de la cible), `{DOMAIN}` (nom de domaine), `{DC}` (contrôleur de domaine), `{URL}` (URL cible), `{USER}` (nom d'utilisateur) et `{PASS}` (mot de passe). Remplacez-les par vos propres valeurs si vous copiez les commandes manuellement.

## OSINT - Passive reconnaissance

Reconnaissance passive : aucun paquet n'est envoyé à la cible. À faire en premier.

!!! info "Références"
    - [HackTricks - Pentesting Network](https://book.hacktricks.wiki/en/generic-methodologies-and-resources/pentesting-network/index.html)
    - [HackTricks - External Recon Methodology](https://book.hacktricks.wiki/en/generic-methodologies-and-resources/external-recon-methodology/index.html)
    - [Certificate Transparency (certificate.transparency.dev)](https://certificate.transparency.dev/)

### Subdomains via crt.sh (robust: crt.sh often returns 502s)

```bash
# crt.sh is unstable -> retry + JSON check before jq
for i in 1 2 3 4 5; do
  out=$(curl -s --max-time 20 'https://crt.sh/?q=%25.{DOMAIN}&output=json')
  echo "$out" | jq -e . >/dev/null 2>&1 && { echo "$out" | jq -r '.[].name_value' | sed 's/\*\.//g' | sort -u; break; }
  sleep 3
done
```

**Voir aussi :**

- [crt.sh - Certificate Search](https://crt.sh/)
- [jq - Manuel](https://jqlang.github.io/jq/manual/)

### CT fallback (certspotter, no API key)

```bash
curl -s 'https://api.certspotter.com/v1/issuances?domain={DOMAIN}&include_subdomains=true&expand=dns_names' | jq -r 'if type=="array" then .[].dns_names[]? else empty end' | sed 's/\*\.//g' | sort -u
```

**Voir aussi :**

- [SSLMate - Cert Spotter](https://sslmate.com/certspotter/)
- [sslmate/certspotter (GitHub)](https://github.com/sslmate/certspotter)

### Source aggregation (subfinder)

```bash
subfinder -d {DOMAIN} -silent
# others: shodan.io | viewdns.info | dnsdumpster.com
```

**Voir aussi :**

- [subfinder - Documentation](https://docs.projectdiscovery.io/opensource/subfinder/overview)
- [projectdiscovery/subfinder (GitHub)](https://github.com/projectdiscovery/subfinder)
- [DNSdumpster](https://dnsdumpster.com/)

### Whois / tech stack

```bash
whois {DOMAIN}
whatweb https://{DOMAIN}
# wappalyzer (extension) for the tech stack
```

**Voir aussi :**

- [Kali Tools - whois](https://www.kali.org/tools/whois/)
- [Kali Tools - WhatWeb](https://www.kali.org/tools/whatweb/)
- [urbanadventurer/WhatWeb (GitHub)](https://github.com/urbanadventurer/WhatWeb)

## setup - Environment setup

Préparation de l'environnement : synchronisez l'horloge (essentiel pour Kerberos) et résolvez le FQDN.

!!! info "Références"
    - [HackTricks - Pentesting Network](https://book.hacktricks.wiki/en/generic-methodologies-and-resources/pentesting-network/index.html)
    - [HackTricks - Pentesting Kerberos (88)](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-kerberos-88/index.html)

### Sync clock with the target (Kerberos)

```bash
sudo ntpdate {IP}
# alt: sudo rdate -n {IP}
```

**Voir aussi :**

- [ntpdate - Manuel (man 8)](https://linux.die.net/man/8/ntpdate)
- [HackTricks - Pentesting Kerberos (88)](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-kerberos-88/index.html)

### Add to /etc/hosts

```bash
echo "{IP} {DOMAIN} {DC}" | sudo tee -a /etc/hosts
```

**Voir aussi :**

- [hosts - Manuel (man 5)](https://linux.die.net/man/5/hosts)

## scan - Active port scan

Scan nmap : scan complet d'abord, puis scan ciblé/scripté sur les ports ouverts. L'option **-oA** produit le fichier **.nmap** (+ .gnmap/.xml) à charger via « Load .nmap » pour filtrer cette page selon les ports ouverts.

!!! info "Références"
    - [HackTricks - Pentesting Network](https://book.hacktricks.wiki/en/generic-methodologies-and-resources/pentesting-network/index.html)
    - [Nmap - Reference Guide (man page)](https://nmap.org/book/man.html)
    - [Kali Tools - Nmap](https://www.kali.org/tools/nmap/)

### Exportable full scan (.nmap to load into this page)

```bash
mkdir -p scans && sudo nmap -p- -sV -sC -T4 -oA scans/{IP} {IP}
# generates scans/{IP}.nmap  ->  "Load .nmap" button
```

**Voir aussi :**

- [Nmap - Service & Version Detection](https://nmap.org/book/man.html)
- [Nmap - NSE Usage](https://nmap.org/book/nse-usage.html)

### Stealth SYN scan without ping (ICMP filtered)

```bash
sudo nmap {IP} -p- -sS -Pn -n --disable-arp-ping --packet-trace
```

**Voir aussi :**

- [Nmap - TCP SYN (Stealth) Scan](https://nmap.org/book/synscan.html)

### UDP top 50

```bash
mkdir -p scans && sudo nmap -sU --top-ports 50 -oA scans/udp_{IP} {IP}
```

**Voir aussi :**

- [Nmap - UDP Scan](https://nmap.org/book/scan-methods-udp-scan.html)

### Vuln scan (NSE)

```bash
mkdir -p scans
# set PORTS to the open ports found above
PORTS=22,80,443
sudo nmap -p$PORTS --script vuln -oA scans/vuln_{IP} {IP}
```

**Voir aussi :**

- [Nmap - NSE Usage](https://nmap.org/book/nse-usage.html)
- [Nmap - Catégorie de scripts « vuln »](https://nmap.org/nsedoc/categories/vuln.html)

### Exploit/CVE search (on the versions found)

```bash
searchsploit <product> <version>
searchsploit -m <id>   # copies the exploit locally
```

**Voir aussi :**

- [Exploit-DB - SearchSploit](https://www.exploit-db.com/searchsploit)
- [Kali Tools - exploitdb](https://www.kali.org/tools/exploitdb/)
