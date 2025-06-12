### TD 3 Revision Guide - TCP/IP (3ème Année IIR) - Full Walkthrough for Exam

---

## EXERCICE 1: Analyse d'un Routeur avec 3 Interfaces

### 🔎 Ce que l'exercice demande:

1. Identifier les sous-réseaux utilisés.
    
2. Compléter la table de routage du routeur.
    
3. Analyser le comportement du routeur pour deux paquets IP donnés.
    
4. Identifier une adresse IP valide pour une nouvelle machine.
    
5. Analyser les problèmes possibles si des adresses erronées sont utilisées.
    

---

### 🔢 Info des interfaces:

- Fa0/0: 192.168.1.62/27 → Sous-réseau: 192.168.1.32/27
    
- Fa0/1: 192.168.1.94/27 → Sous-réseau: 192.168.1.64/27
    
- Fa0/2: 192.168.1.126/27 → Sous-réseau: 192.168.1.96/27
    

**Masque /27 = 255.255.255.224 = 32 adresses par sous-réseau**

### 1. Sous-réseaux utilisés:

- 192.168.1.32/27
    
- 192.168.1.64/27
    
- 192.168.1.96/27
    

### 2. Table de routage:

|Réseau Destination|Masque|Interface|
|---|---|---|
|192.168.1.32|255.255.255.224|Fa0/0|
|192.168.1.64|255.255.255.224|Fa0/1|
|192.168.1.96|255.255.255.224|Fa0/2|

### 3. Comportement du routeur:

(a) Source: 192.168.1.35 → sous-réseau 192.168.1.32/27  
Destination: 192.168.1.85 → sous-réseau 192.168.1.64/27  
=> Le routeur transfère le paquet de Fa0/0 à Fa0/1

(b) Destination: 192.168.1.185 → en dehors des sous-réseaux connus => **paquet rejeté** (pas de route)

### 4. IP valide pour une machine dans le LAN1 (sous-réseau 192.168.1.32/27):

- Plage valide: .33 à .62 (excluant .32 et .63)
    
- Analyse des options:
    
    - (a) 192.168.1.63/28 ❌ faux masque
        
    - (b) 192.168.1.63/27 ✔ (dernière IP valide)
        
    - (c) 192.168.1.60/28 ❌ masque incorrect
        
    - (d) 192.168.1.60/27 ✔ correct
        
    - (e) 192.168.1.120/27 ❌ appartient à un autre sous-réseau
        

### 5. Si admin donne:

- (a) 192.168.1.70/27 ❌ Pas dans LAN1, appartient à LAN2
    
- (b) 192.168.1.62/27 ✔ Oui, c'est une adresse valide du LAN1
    

---

## EXERCICE 2: Table de Routage d'un Routeur

### 🔎 Ce qu'on demande:

Identifier les lignes incohérentes dans une table de routage

### Interfaces:

- lo: 127.0.0.1
    
- eth0: 200.100.40.1
    
- eth1: 200.100.50.1
    

### Analyse ligne par ligne:

1. OK ✔ adresse directe du routeur eth1
    
2. OK ✔ réseau directement connecté
    
3. ❌ INCOHÉRENTE → eth0 n'est pas dans le réseau 200.100.60.0
    
4. ❌ INCOHÉRENTE → eth0 ne peut atteindre 200.100.70.0 via 40.2 sans route claire
    
5. OK ✔ routage par eth1
    
6. ❌ Adresse 255.255.255.255 = broadcast => incohérent ici
    
7. ❌ default route (0.0.0.0) ne devrait pas pointer vers 127.0.0.1 => fausse
    

---

## EXERCICE 3: Sous-réseautage et Routage Interne

### Objectif:

- Diviser 192.168.81.0/24 en 4 sous-réseaux
    
- Compléter les tables de routage pour tous les routeurs
    

### 1. Adresses des sous-réseaux:

/26 = 64 adresses = 62 machines

- Réseau 0: 192.168.81.0/26
    
- Réseau 1: 192.168.81.64/26
    
- Réseau 2: 192.168.81.128/26
    
- Réseau 3: 192.168.81.192/26
    

### 2. IPs des routeurs:

Prendre la première adresse dispo (pas .0 ni .63):

- R1: IP10 = 192.168.81.1, IP12 = 192.168.81.65
    
- R2: IP21 = 192.168.81.66, IP23 = 192.168.81.129
    
- R3: IP32 = 192.168.81.130, IP34 = 192.168.81.193
    
- R4 (vers internet): IP40 = 192.168.81.194
    

### 3. Tables complètes de R1, R2, R3:

(voir TD pour structure)

Exemple pour R1:

|Réseau destination|Masque|Prochain saut|
|---|---|---|
|192.168.81.0|255.255.255.192|*|
|192.168.81.64|255.255.255.192|IP21|
|192.168.81.128|255.255.255.192|IP21|
|192.168.81.192|255.255.255.192|IP21|
|0.0.0.0|0.0.0.0|IP21 (vers R4)|

### 4. Table de routage station réseau 2:

|Réseau Destination|Masque|Passerelle|
|---|---|---|
|192.168.81.0|255.255.255.192|IP23|
|192.168.81.64|255.255.255.192|IP23|
|192.168.81.128|255.255.255.192|*|
|192.168.81.192|255.255.255.192|IP23|
|0.0.0.0|0.0.0.0|IP23|

---

## EXERCICE 4: Routage avec Contraintes et Scénarios

### 🔎 Objectif:

Configurer les tables de routage des routeurs et stations selon des conditions spécifiques

### 1. Routage basique (tout le monde parle avec tout le monde)

- Tables classiques avec routes vers tous les sous-réseaux
    

### 2. Contraintes à implémenter:

(a) LAN2 accède au serveur Web  
(b) LAN1 & LAN3 **n'accèdent PAS** au serveur Web  
(c) Toutes les machines peuvent accéder aux autres + serveur de fichiers  
(d) Le serveur de fichiers accède au serveur Web  
(e) Passage LAN2 ⇆ LAN3 via routeur 1 uniquement

=> Nécessite listes de contrôle d'accès (ACL) ou tables filtrantes

### 3. Analyse d'un ping:

- De LAN1 vers serveur Web: bloqué (selon règle b)
    
- De LAN2 vers Web: OK, passe par R1 → R3
    

### 4. Panne de R1:

Solution économique:

- Ajouter routes statiques de secours dans R2 et R3 pour rediriger le trafic sans R1
    
- Eviter de changer la topologie physique (reliement direct R2-R3)
    

---

## 🕑 TECHNIQUES & ASTUCES POUR L'EXAM

### 🔄 Reconnaitre les types d'exos:

- "Adresse + masque" => ⇒ calcul de sous-réseaux
    
- "Routeur" => ⇒ table de routage + analyse de paquet
    
- "Comportement d'un ping" => ⇒ chemin logique selon IPs et règles
    

### 🔢 Formules clés:

- Nb hôtes: 2^(32 - masque) - 2
    
- Plage d'un sous-réseau: Incrément = 256 - dernier octet du masque
    

### ✨ Mnémoniques:

- CIDR → /27 = 32 IPs
    
- Classe C → /24
    
- Réseau .0, Broadcast = dernier IP
    

### ⚠️ Erreurs classiques:

- Confondre masque /28 avec /27
    
- Donner une adresse de broadcast à une machine
    
- Routeur sans route par défaut => paquets perdus
    

---

