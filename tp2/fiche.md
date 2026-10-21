# TP2 — Reconnaissance active du réseau d'ACME
## Mission 1 — Cartographier et justifier ses observations

| | |
|---|---|
| **Auteur(s)** | _Nom(s)_ |
| **Date / heure** | _JJ/MM/AAAA, hh:mm_ |
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
| DMZ | 172.21.0.1 | | | | Interface de ma Kali (passerelle) |
| DMZ | 172.21.0.10 | | | | |
| DMZ | 172.21.0.11 | | | | |
| DMZ | 172.21.0.13 | | | | |
| DMZ | 172.21.0.14 | | | | |
| LAN | 172.22.0.1 | | | | Interface de ma Kali (passerelle) |
| LAN | 172.22.0.12 | | | | |
| LAN | 172.22.0.13 | | | | |
| LAN | 172.22.0.14 | | | | |

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
| 172.21.0.10 | | | | | | |
| 172.21.0.11 | | | | | | |
| 172.21.0.13 | | | | | | |
| 172.21.0.14 | | | | | | |
| 172.22.0.12 | | | | | | |
| 172.22.0.13 | | | | | | |
| 172.22.0.14 | | | | | | |

**Versions inconnues ou incertaines (à signaler) :**
- _IP:port — ce que Nmap affiche, pourquoi la version n'est pas identifiée_

### Questions

**1. Une machine répond au host discovery mais n'a aucun port ouvert trouvé : peut-on la dire « sans service » ?**
_Réponse :_

**2. Comparaison du scan par défaut et du scan avec le port Redis (6379)**

| | Scan par défaut | Scan avec `-p6379` |
|---|---|---|
| Ports trouvés sur 172.21.0.13 | | |
| Différence observée | | |

_Pourquoi cette différence ? (le scan par défaut ne teste qu'une sélection de ports)_

**3. Quelle cible reste discrète ?**
_Réponse + justification :_

**4. Pourquoi « hôte vivant » et « aucun port ouvert trouvé » ne se contredisent-ils pas ?**
_Réponse (différence entre découverte d'hôte et scan de ports ; filtrage ; ports hors sélection ; UDP…) :_

**5. Ai-je testé tous les ports ? Et l'UDP ?**
_Réponse (nombre de ports réellement testés, ce qui n'a pas été couvert) :_

### Scan UDP / SNMP (172.22.0.14)

| | Scan TCP (`-sV`) | Scan UDP (`-sU -sV -p161`) |
|---|---|---|
| Port 161 vu ? | | |
| État | | |
| Service / version | | |

_Conclusion : pourquoi le scan TCP seul passait-il à côté de ce service ?_

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

### Information nouvelle retenue

| Champ | Contenu |
|---|---|
| **Script NSE** | |
| **Cible / port** | |
| **Information révélée** | |
| **À quoi sert-elle ?** | |
| **Type d'information** | ☐ version ☐ configuration ☐ preuve de vulnérabilité |
| **Justification du type** | |

### Fiche mise à jour

| IP | Port | Information ajoutée par NSE |
|---|---|---|
| | | |

---

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
