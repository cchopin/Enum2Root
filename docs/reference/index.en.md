# Full reference

This section documents **every action** of the Enum2Root methodology, phase by phase. For each
command you get the exact syntax (with the `{IP}`, `{DOMAIN}`, `{USER}`… tokens substituted in
the app) and **links to verified external resources**: HackTricks, PayloadsAllTheThings,
GTFOBins, LOLBAS, The Hacker Recipes, PortSwigger, official tool docs, etc.

!!! tip "All links are verified"
    Every external link in this reference has been tested (HTTP 2xx). If you find a broken one,
    open an [issue](https://github.com/cchopin/Enum2Root/issues).

!!! note "Language"
    The detailed reference pages are currently written in French; the external resources they
    link to are mostly in English. The **Guide** and **Tools** pages are available in both
    languages.

<div class="grid cards" markdown>

-   :material-radar: __0 - Reconnaissance__

    Passive OSINT, environment prep, active nmap scanning.

    [:octicons-arrow-right-24: Open](reconnaissance.md)

-   :material-lan: __1 - Service enumeration__

    Dozens of services: FTP, SSH, SMB, LDAP, Kerberos, MSSQL, RDP, WinRM…

    [:octicons-arrow-right-24: Part 1](enumeration-1.md) · [Part 2](enumeration-2.md)

-   :material-web: __2 - Web attacks__

    SQLi, XSS, LFI, brute force, WordPress, command injection, SSTI, upload.

    [:octicons-arrow-right-24: Open](web.md)

-   :material-domain: __3 - Active Directory__

    Credential acquisition, enumeration, Kerberoast, ACL abuse, ADCS, NTLM relay, dominance.

    [:octicons-arrow-right-24: Open](active-directory.md)

-   :material-shield-key: __4 - Post-exploitation__

    Linux & Windows privesc, looting and persistence.

    [:octicons-arrow-right-24: Open](post-exploitation.md)

-   :material-tunnel: __5 - Shells, transfer & pivot__

    Reverse shells, TTY stabilization, file transfer, tunneling, hash cracking.

    [:octicons-arrow-right-24: Open](shells-pivot.md)

</div>

## Referenced external sources

| Source | Domain | Main use |
|--------|--------|----------|
| HackTricks | `book.hacktricks.wiki` | Methodology per service / technique |
| PayloadsAllTheThings | `swisskyrepo.github.io` | Web payloads (SQLi, XSS, SSTI…) |
| The Hacker Recipes | `thehacker.recipes` | Active Directory in depth |
| GTFOBins | `gtfobins.github.io` | Linux privesc (SUID/sudo) |
| LOLBAS | `lolbas-project.github.io` | Abusable Windows binaries |
| PortSwigger Web Security Academy | `portswigger.net` | Web theory & labs |
| Tool docs | GitHub / Kali | Reference for each tool |
