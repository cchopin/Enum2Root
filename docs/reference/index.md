# Référence complète

Cette section documente **chaque action** de la méthodologie Enum2Root, phase par phase. Pour
chaque commande, tu trouveras la syntaxe exacte (avec les jetons `{IP}`, `{DOMAIN}`, `{USER}`…
substitués dans l'app) et des **liens vers des ressources externes vérifiées** : HackTricks,
PayloadsAllTheThings, GTFOBins, LOLBAS, The Hacker Recipes, PortSwigger, docs officielles des
outils, etc.

!!! tip "Tous les liens sont vérifiés"
    Chaque lien externe de cette référence a été testé (HTTP 2xx). Si tu en trouves un cassé,
    ouvre une [issue](https://github.com/cchopin/Enum2Root/issues).

<div class="grid cards" markdown>

-   :material-radar: __0 - Reconnaissance__

    OSINT passif, préparation de l'environnement, scan nmap actif.

    [:octicons-arrow-right-24: Ouvrir](reconnaissance.md)

-   :material-lan: __1 - Énumération des services__

    Des dizaines de services : FTP, SSH, SMB, LDAP, Kerberos, MSSQL, RDP, WinRM…

    [:octicons-arrow-right-24: Partie 1](enumeration-1.md) · [Partie 2](enumeration-2.md)

-   :material-web: __2 - Attaques web__

    SQLi, XSS, LFI, brute force, WordPress, command injection, SSTI, upload.

    [:octicons-arrow-right-24: Ouvrir](web.md)

-   :material-domain: __3 - Active Directory__

    Acquisition de creds, énumération, Kerberoast, abus d'ACL, ADCS, relais NTLM, domination.

    [:octicons-arrow-right-24: Ouvrir](active-directory.md)

-   :material-shield-key: __4 - Post-exploitation__

    Privesc Linux & Windows, pillage et persistance.

    [:octicons-arrow-right-24: Ouvrir](post-exploitation.md)

-   :material-tunnel: __5 - Shells, transfert & pivot__

    Reverse shells, stabilisation TTY, transfert de fichiers, tunneling, cassage de hash.

    [:octicons-arrow-right-24: Ouvrir](shells-pivot.md)

</div>

## Sources externes référencées

| Source | Domaine | Usage principal |
|--------|---------|-----------------|
| HackTricks | `book.hacktricks.wiki` | Méthodologie par service / technique |
| PayloadsAllTheThings | `swisskyrepo.github.io` | Payloads web (SQLi, XSS, SSTI…) |
| The Hacker Recipes | `thehacker.recipes` | Active Directory en profondeur |
| GTFOBins | `gtfobins.github.io` | Privesc Linux (SUID/sudo) |
| LOLBAS | `lolbas-project.github.io` | Binaires Windows détournables |
| PortSwigger Web Security Academy | `portswigger.net` | Théorie & labs web |
| Docs outils | GitHub / Kali | Référence de chaque outil |

!!! note "Langue"
    La référence détaillée est rédigée en français. Les pages **Guide** existent en
    français et en anglais. Les ressources externes liées sont majoritairement en anglais.
