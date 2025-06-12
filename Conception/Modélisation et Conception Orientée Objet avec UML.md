
# 1. Modélisation Dynamique

## A. Objectif

La modélisation dynamique permet de décrire le comportement du système au fil du temps, en mettant l'accent sur les interactions entre objets.

# 2. Diagrammes d'interactions (Diagrammes de séquence)

Utilisés tout au long du cycle de vie du projet :

- Pour documenter les **cas d'utilisation**.
    
- Pour **décrire les interactions** dans un langage proche de l'utilisateur (les messages = événements).
    
- Pour modéliser **l'implémentation d'une classe** ou d'une opération.
    
- Pour **représenter les interactions informatiques** et les **flots de contrôle**.
    

# 3. Objets / Participants

- Représentation : `nomRôle:NomClasse` (souligné pour indiquer une instance).
    

# 4. Ligne de vie

- Ligne verticale sous l'objet = durée de vie dans l'interaction.
    
- **Création** : message pointant vers l'objet.
    
- **Destruction** : fin par une croix (×).
    

# 5. Messages

- Communication entre lignes de vie.
    
- **Ordre horizontal** = pas important.
    
- **Axe vertical** = temps qui passe.
    
- Messages étiquetés (nom de l'opération ou signal).
    

## Types de messages

- **Signal** (asynchrone)
    
- **Invocation d'opération** (synchrone)
    
- **Création / Destruction**
    
- Représentations graphiques différentes :
    
    - Flèche simple : asynchrone
        
    - Flèche triangulaire : appel de procédure (synchrone)
        
    - Flèche pointillée : message de retour
        

## Flot de contrôle

- Messages asynchrones simples à flèche unique.
    
- En système concurrent :
    
    - **Demi-flèche** = envoi de message sans attente.
        
    - **Flèche pleine** = avec attente de réponse.
        

## Appel de procédure

- Emboîté, la procédure appelée doit finir avant le retour à l'appelant.
    

## Retour explicite

- Utile en système concurrent.
    
- Flèche pointillée pour signaler la fin.
    

## Création / destruction d'instance

- Message de création = flèche vers une instance.
    
- Destruction = croix en bas de ligne de vie.
    

## Message perdu / trouvé

- **Complet** : message avec envoi et réception connus.
    
- **Perdu** : envoi connu, réception inconnue.
    
- **Trouvé** : réception connue, envoi inconnu.
    

## Étiquettes de message

Syntaxe :

```
[« garde »] [itération] [résultat :=] nomMessage(arguments)
```

- `garde` : condition booléenne (optionnelle, entre crochets).
    
- `itération` : boucle.
    
- `nomMessage` : nom de l'opération ou signal.
    

## Appel récursif / Réflexivité

- **Récursif** : même objet actif plusieurs fois (activation dédoublée).
    
- **Réflexivité** : un objet s'envoie un message à lui-même.
    

# 6. Cadre d'interaction (UML 2.0+)

- Permet de **représenter plusieurs scénarios** dans un même diagramme.
    
- Contient des **opérateurs d'interaction**.
    

## Opérateurs principaux

- `alt` : alternative (choix entre opérandes)
    
- `opt` : option (choix unique)
    
- `break` : sortie / scénario d'exception
    
- `loop` : boucle
    
- `par` : parallèle (messages envoyés en même temps)
    
- `neg` : scénario interdit ou invalide
    
- `assert`, `ignore`, `consider`, `critical`, `seq`, `strict` : autres cas d’usage
    

---

**Remarque** : les diagrammes de séquence UML sont un outil clé pour visualiser les comportements systèmes, la collaboration entre objets, et les scénarios dynamiques complexes.

> Ce mini-notebook sert de mémo complet pour la partie "Modélisation dynamique avec UML". Utilisable pour vos révisions, projets, ou même pour l'enseignement.