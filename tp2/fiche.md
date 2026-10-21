# TP2 — Reconnaissance active du réseau d'ACME
## Mission 1 — Cartographier et justifier ses observations

| | |
|---|---|
| **Auteur(s)** | Desbeek Lucas |
| **Date / heure** |21/10/2026 09:22 |
| **Machine attaquante** | Kali — _version, IP sur chaque réseau_ |
| **Réseaux ciblés** | DMZ `172.21.0.0/24` · LAN `172.22.0.0/24` |
| **Référence** | Borck, *week4 — Nmap Scanning & Scan Reporting*, parties 1, 2, 3, 5, 7 |
| **Dossier des traces** | `~/tp2-traces/` |

> **Cadre** : toutes les commandes ont été lancées uniquement sur le labo Docker, depuis ma Kali. Les adresses de l'énoncé priment sur celles du guide.

---

## 0. Préparation du labo

- [ ] Paquets installés (`docker.io`, `docker-compose`) et service Docker activé
- [ ] Utilisateur ajouté au groupe `docker` (session rouverte si nécessaire)
- [ ] Dépôt à jour (`git pull` dans `~/secu_tp`)
- [ ] `verifier-labo.sh` exécuté (état initial) : _OK / problèmes rencontrés_
- [ ] `docker-compose up -d` puis `docker-compose ps` : _tous les conteneurs « Up » ?_
- [ ] `verifier-labo.sh --pret` : _OK / problèmes rencontrés_
- [ ] Dossier `~/tp2-traces` créé

**Remarques / incidents :**
_…_

---

## 1a — Qui répond ?

### Commandes

```bash
sudo nmap -sn 172.21.0.0/24
sudo nmap -sn 172.22.0.0/24
```

### Hôtes trouvés

| Réseau | IP | Répond ? | Latence | MAC / fabricant | Nature (cible ACME / interface Kali) |
|---|---|---|---|---|---|
| DMZ | 172.21.0.1 |oui| n/a | n/a | Interface de ma Kali (passerelle) |
| DMZ | 172.21.0.10 |oui| 0.000013 s | D2:49:2E:0A:90:A5 | Cible ACME |
| DMZ | 172.21.0.11 |oui| 0.000006 s | A6:E4:00:D1:B5:A1 | Cible ACME |
| DMZ | 172.21.0.13 |oui| 0.000014 s | 86:E4:C2:10:7F:93 | Cible ACME |
| DMZ | 172.21.0.14 |oui| 0.000014 s | 92:4D:74:AF:E8:29 | Cible ACME |
| LAN | 172.22.0.1 |oui|n/a|n/a| Interface de ma Kali (passerelle) |
| LAN | 172.22.0.12 | oui|0.000010 s|76:89:CB:8A:83:F6 | Cible ACME|
| LAN | 172.22.0.13 | oui|0.000009 s| 8A:53:AE:83:49:A9| Cible ACME|
| LAN | 172.22.0.14 | oui|0.000035 s|4A:0E:0D:04:2E:F2 | Cible ACME|

**Nombre de cibles ACME :** _/ 7_ (les deux `.1` sont exclues du décompte).

### Hypothèse de rôle par cible (à confirmer en 1b)

| IP | Hypothèse de rôle | Sur quoi repose-t-elle ? | Niveau de confiance |
|---|---|---|---|
| 172.21.0.10 | | | |
| 172.21.0.11 | | | |
| 172.21.0.13 | | | |
| 172.21.0.14 | | | |
| 172.22.0.12 | | | |
| 172.22.0.13 | | | |
| 172.22.0.14 | | | |

**Notes :** _comment ai-je distingué les `.1` des vraies cibles ?_

---

## 1b — Quels ports, quels services ?

### Commandes

```bash
nmap -sV -oN ~/tp2-traces/scan-dmz.txt 172.21.0.10,11,13,14
nmap -sV -oN ~/tp2-traces/scan-lan.txt 172.22.0.12,13,14
nmap -sV -p6379 -oN ~/tp2-traces/scan-redis.txt 172.21.0.13
sudo nmap -sU -sV -p161 172.22.0.14
```

### Carte des services

| IP | Port/proto | État | Service | Produit | Version | Remarque |
|---|---|---|---|---|---|---|
| 172.21.0.10 |80/tcp|open|http| Apache httpd|2.4.69(Unix)| |
| 172.21.0.11 |22/tcp|open|ssh|OpenSSH|8.9p1 Ubuntu 3ubuntu0.18, protocole 2.0| |
| 172.21.0.13 |aucun|n/a|n/a|n/a|n/a| |
| 172.21.0.14 |aucun|n/a|n/a|n/a|n/a| |
| 172.22.0.12 |3306/tcp |open |mysql |MySQL | MySQL 8.0.43| |
| 172.22.0.13 |139/tcp |open | netbios-ssn| Samba smbd|4 | |
| 172.22.0.13 |445/tcp |open | netbios-ssn| Samba smbd|4 | |
| 172.22.0.14 |aucun |	n/a| 	n/a|	n/a|	n/a | |

**Versions inconnues ou incertaines (à signaler) :**
- _IP:port — ce que Nmap affiche, pourquoi la version n'est pas identifiée_

### Questions

**1. Une machine répond au host discovery mais n'a aucun port ouvert trouvé : peut-on la dire « sans service » ?**
_Réponse :_

**2. Comparaison du scan par défaut et du scan avec le port Redis (6379)**

| | Scan par défaut | Scan avec `-p6379` |
|---|---|---|
| Ports trouvés sur 172.21.0.13 | Aucun : 1000 ports TCP testés, tous fermés (reset) | 6379/tcp ouvert, Redis key-value store 7.4.11 |
| Différence observée | La machine paraît sans service | Un service Redis est identifié avec sa version |

_Pourquoi cette différence ?_ Le scan par défaut ne teste que les 1000 ports TCP les plus courants. Le port 6379 n'en faisait pas partie lors de mon scan, donc Nmap ne l'a jamais sondé. Le service existait depuis le début : c'est le périmètre du scan qui l'a masqué, pas la machine.

**3. Quelle cible reste discrète ?**

_Réponse :_ **172.21.0.14** (DMZ). C'est la seule cible dont aucun port ouvert n'a été trouvé, malgré le scan TCP par défaut : 1000 ports fermés (reset). Les deux autres cibles « silencieuses » au départ se sont révélées avec le bon scan : 172.21.0.13 avec `-p6379` (Redis) et 172.22.0.14 avec le scan UDP (SNMP). Je la classe donc « vivante, aucun port TCP ouvert dans les 1000 testés, UDP et autres ports non testés », et non « sans service ».

**4. Pourquoi « hôte vivant » et « aucun port ouvert trouvé » ne se contredisent-ils pas ?**

_Réponse :_ Ce sont deux questions différentes. Le host discovery (`-sn`) demande seulement si l'hôte répond, ici par ARP sur le même segment. Le scan de ports demande si un service écoute sur des ports précis. Une machine peut être vivante sans rien sur les ports testés, parce que ces ports sont fermés (réponse RST), parce que le service écoute sur un port hors sélection (Redis sur 6379), ou parce qu'il utilise UDP (SNMP sur 161/udp). Dans mes scans, les ports étaient « closed (reset) » et non « filtered » : les hôtes répondaient activement, aucun pare-feu n'ignorait mes paquets. « Aucun port ouvert trouvé » décrit donc ce que j'ai testé, pas ce qui existe.

**5. Ai-je testé tous les ports ? Et l'UDP ?**

_Réponse :_ **Non.**

- **TCP** : 1000 ports par défaut sur chacune des 7 cibles, sur 65 535 possibles. Le port 6379 a été ajouté uniquement sur 172.21.0.13.
- **UDP** : un seul port, 161, sur une seule cible (172.22.0.14). Aucun port UDP n'a été testé sur les 6 autres cibles.
- **Conséquence** : l'absence de résultat ailleurs ne prouve pas l'absence de service. Les ports TCP hors sélection et l'UDP restent non couverts.

### Scan UDP / SNMP (172.22.0.14)

| | Scan TCP (`-sV`) | Scan UDP (`-sU -sV -p161`) |
|---|---|---|
| Port 161 vu ? | Non testé en TCP | Oui |
| État | 1000 ports TCP fermés (reset) | 161/udp open |
| Service / version | Aucun | SNMP : SNMPv1 et SNMPv3 annoncés (pysnmp / SNMP Laboratories), version du produit non identifiée ; hôte `enum-edge-router-01` |

_Conclusion :_ SNMP utilise UDP, pas TCP. Un scan TCP, même complet, ne peut donc pas le voir : il faut cibler le bon protocole et le bon port. Le nom d'hôte `enum-edge-router-01` suggère un équipement réseau, mais c'est une hypothèse à confirmer. Le scan ne prouve pas non plus que l'accès SNMP soit ouvert.

---

## 1c — Comparer deux façons de scanner *(à faire en dernier)*

> Ce n'est pas une étape pour avancer dans la carte : c'est une comparaison de méthode.

### Commandes (cible 172.21.0.11, ports 22 et 80)

```bash
sudo nmap -sS -p22,80 172.21.0.11
nmap -sT -p22,80 172.21.0.11
```

### Comparaison

| Critère | SYN (`-sS`) | TCP connect (`-sT`) |
|---|---|---|
| Ports trouvés | | |
| Privilèges root nécessaires ? | | |
| Séquence de paquets échangés | | |
| Connexion complète établie ? | | |
| Visible dans les logs applicatifs ? | | |

**Les ports trouvés changent-ils ?** _…_

**Lequel nécessite des privilèges pour fabriquer les paquets, et pourquoi ?** _…_

**Qu'est-ce qui change dans l'établissement de la connexion ?** _(SYN / SYN-ACK / RST vs SYN / SYN-ACK / ACK)_

**Une trace absente des logs applicatifs signifie-t-elle forcément que le réseau n'a rien vu ?**
_Réponse (pare-feu, IDS, capture réseau, journaux système…) :_

---

## 1d — Un script pour une question précise

### Commandes

```bash
nmap -sC 172.21.0.10,11
nmap -p6379 --script redis-info 172.21.0.13
```

### Informations révélées par NSE

| Cible / port | Script | Information révélée |
|---|---|---|
| 172.21.0.10:80 | `http-title`, `http-methods` | Titre de page « WebServer » ; méthode HTTP **TRACE** signalée comme potentiellement risquée |
| 172.21.0.11:22 | `ssh-hostkey` | Empreintes des clés d'hôte : ECDSA 256 bits (`19:a0:5b:13:…:05:61`) et ED25519 (`99:aa:c4:36:…:c1:ab`) |
| 172.21.0.13:6379 | `redis-info` | Version 7.4.11, rôle `master`, 0 réplica, écoute sur toutes les interfaces (`*`), 1 client connecté (172.21.0.1), PID 1, 1,06 Mo de mémoire utilisée |

### Information nouvelle retenue

| Champ | Contenu |
|---|---|
| **Script NSE** | `redis-info` |
| **Cible / port** | 172.21.0.13 : 6379/tcp |
| **Information révélée** | Le script a obtenu la sortie de la commande `INFO` de Redis sans fournir d'identifiants : version 7.4.11, rôle `master`, écoute sur toutes les interfaces (`*`), 0 réplica, 1 client connecté |
| **À quoi sert-elle ?** | Elle confirme la version annoncée par `-sV` et montre que le service est interrogeable sans authentification. Pour un attaquant, c'est un point d'entrée à étudier. Pour un défenseur, c'est un signal de configuration à corriger (authentification, restriction de l'écoute) |
| **Type d'information** | ☐ version ☒ **configuration** ☐ preuve de vulnérabilité |
| **Justification du type** | Le script décrit comment Redis est configuré et exposé (accès sans mot de passe, écoute sur `*`, rôle master). La version n'est qu'une confirmation. Aucune vulnérabilité n'est prouvée : aucune exploitation n'a été tentée |

### Points d'attention sur cette sortie

- **Le client connecté `172.21.0.1` est ma propre Kali** : c'est la connexion du script lui-même, pas une autre machine du réseau.
- **« Linux 6.19.14+kali-amd64 » n'est pas la distribution du conteneur** : c'est le noyau de l'hôte, partagé par les conteneurs Docker. Avec `Process ID: 1`, cela suggère un conteneur, mais ne dit rien de l'image utilisée.
- **Une connexion sans mot de passe n'est pas une compromission** : elle prouve une exposition, pas un impact. Je n'ai pas lu ni modifié de données.

### Autres informations notées

- **172.21.0.10** : le titre « WebServer » est générique et ne révèle pas le rôle de l'application. TRACE activé est un point de configuration à vérifier, pas une vulnérabilité confirmée par ce seul résultat.
- **172.21.0.11** : les empreintes de clés servent à identifier la machine et à repérer un changement de clé (usurpation, réinstallation). Elles ne disent rien sur la sécurité du service.

### Fiche mise à jour

| IP | Port | Information ajoutée par NSE |
|---|---|---|
| 172.21.0.10 | 80/tcp | Titre « WebServer » ; TRACE activé (à vérifier) |
| 172.21.0.11 | 22/tcp | Empreintes ECDSA et ED25519 |
| 172.21.0.13 | 6379/tcp | Redis 7.4.11, master, écoute sur `*`, accessible sans authentification apparente |

## Bilan de la mission

### Checklist « Fini quand »

- [ ] La carte couvre les **sept** cibles
- [ ] Les versions inconnues sont signalées
- [ ] Au moins un output est enregistré (`~/tp2-traces/…`)
- [ ] Je peux expliquer une différence entre deux scans

### Traces enregistrées

| Fichier | Commande | Contenu |
|---|---|---|
| `~/tp2-traces/scan-dmz.txt` | `nmap -sV …` | |
| `~/tp2-traces/scan-lan.txt` | `nmap -sV …` | |
| `~/tp2-traces/scan-redis.txt` | `nmap -sV -p6379 …` | |

### Une différence entre deux scans, expliquée
_Scan A vs scan B → différence → cause :_

---

## Préparation de la discussion

Deux observations utiles à un attaquant **et** à un défenseur :

### Observation 1
- **Constat :** _…_
- **Intérêt pour l'attaquant :** _…_
- **Intérêt pour le défenseur :** _…_
- **Preuve :** _commande + extrait de sortie_
- **Ce que cette preuve ne permet pas encore d'affirmer :** _…_

### Observation 2
- **Constat :** _…_
- **Intérêt pour l'attaquant :** _…_
- **Intérêt pour le défenseur :** _…_
- **Preuve :** _commande + extrait de sortie_
- **Ce que cette preuve ne permet pas encore d'affirmer :** _…_

---

## Bonus *(si avance rapide)* — Script SSH ciblé (guide week4, partie 5)

| Champ | Contenu |
|---|---|
| **Script choisi** | |
| **Commande** | |
| **Pourquoi ce choix ?** | |
| **Résultat du script** | |
| **Résultat de `-sV` sur le même port** | |
| **Comparaison** | |

> Ne pas lancer tous les scripts, ni de scans très lents sur tout le réseau.

---

## Journal de bord

| Heure | Action / commande | Résultat | Remarque |
|---|---|---|---|
| | | | |

## Sources
- Borck, *week4 — Nmap Scanning & Scan Reporting*
- Énoncé TP2 — Reconnaissance active du réseau d'ACME

Un alias Apache (/icons/) laissait remonter dans l’arborescence via des séquences ../ encodées (.%2e/), permettant de sortir du répertoire web autorisé et de lire des fichiers système comme httpd.conf. C’est un path traversal : l’encodage URL du point contourne un filtre qui ne cherchait que la chaîne littérale ../. Le serveur ne vérifiait pas correctement que le chemin final restait dans le DocumentRoot après décodage
