
---

# 🎯 Objectif du TP 4 — Introduction au T-SQL

On travaille avec une base de données de **gestion commerciale (GestionCom)**. Elle contient :

- Des **articles** (produits à vendre),
    
- Des **commandes**,
    
- Des **lignes de commande** (les détails d’une commande : quels articles, combien, etc).
    

---

## 🧠 RAPPEL : Les tables de la BDD

### Table `Articles` :

|Champ|Type|Description|
|---|---|---|
|`NumArt`|INT (PK)|Identifiant unique de l’article|
|`NomArt`|NVARCHAR(100)|Nom ou désignation de l’article|
|`QteEnStock`|INT|Quantité en stock|
|`SeuilMin`|INT|Seuil minimum de stock|
|`Prix`|DECIMAL(10,2)|Prix de l’article|

---

### Table `Commandes` :

|Champ|Type|Description|
|---|---|---|
|`NumCom`|INT (PK)|Numéro de la commande|
|`DatCom`|DATE|Date de la commande|

---

### Table `LigneCommandes` :

|Champ|Type|Description|
|---|---|---|
|`NumCom`|INT|Numéro de la commande|
|`NumArt`|INT|Numéro de l’article commandé|
|`QteCom`|INT|Quantité commandée|

---

## ✅ EXERCICE 1 : Afficher la liste des articles

> Tu dois afficher **chaque article** dans une phrase comme :  
> **L'article numéro 1 portant la désignation Article 1 coûte 10.5**

### 💡 Logique :

On va utiliser une **requête SELECT** avec **concaténation** (`+`) pour former une **phrase complète**.

### 📘 T-SQL :

```sql
SELECT 
    'L''article numéro ' + CAST(NumArt AS NVARCHAR) + 
    ' portant la désignation ' + NomArt + 
    ' coûte ' + CAST(Prix AS NVARCHAR) AS DescriptionArticle
FROM Articles;
```

### 🧠 Explication :

- `CAST(... AS NVARCHAR)` : On convertit les nombres (`INT` ou `DECIMAL`) en texte pour pouvoir les **concaténer** avec des phrases.
    
- `'L''article numéro '` : On utilise `''` pour insérer une apostrophe `'` dans une chaîne T-SQL.
    
- On assemble toute la phrase dans un alias `AS DescriptionArticle`.
    

---

## ✅ EXERCICE 2 : Programme qui affiche chaque commande avec ses articles et le montant total

### a. Afficher le numéro et la date de chaque commande :

```sql
SELECT 
    'Commande N° : ' + CAST(NumCom AS NVARCHAR) + 
    ' effectuée le : ' + CONVERT(NVARCHAR, DatCom, 103) AS InfoCommande
FROM Commandes;
```

- `CONVERT(NVARCHAR, DatCom, 103)` : Affiche la date en format `jj/mm/aaaa` (style 103).
    

---

### b. Liste des articles associés à chaque commande

Ici on fait une **jointure** entre :

- `Commandes`
    
- `LigneCommandes`
    
- `Articles`
    

### 📘 T-SQL :

```sql
SELECT 
    c.NumCom,
    a.NumArt,
    a.NomArt,
    lc.QteCom,
    a.Prix,
    (a.Prix * lc.QteCom) AS Montant
FROM Commandes c
JOIN LigneCommandes lc ON c.NumCom = lc.NumCom
JOIN Articles a ON lc.NumArt = a.NumArt
ORDER BY c.NumCom;
```

### 🧠 Explication :

- On relie les 3 tables par leurs **clés étrangères**.
    
- Pour chaque ligne, on affiche :
    
    - Le numéro de commande
        
    - L’article commandé
        
    - La quantité
        
    - Le prix
        
    - Le **montant = prix * quantité**
        

---

### c. Montant total de chaque commande

On utilise `GROUP BY` pour **additionner les montants par commande** :

```sql
SELECT 
    c.NumCom,
    SUM(a.Prix * lc.QteCom) AS MontantTotal
FROM Commandes c
JOIN LigneCommandes lc ON c.NumCom = lc.NumCom
JOIN Articles a ON lc.NumArt = a.NumArt
GROUP BY c.NumCom;
```

---

## ✅ EXERCICE 3 : Vérifier si une commande a au moins un article, sinon la supprimer

### Étape 1 : Vérifier s’il y a des articles dans la commande

On compte les lignes de `LigneCommandes` pour chaque commande.

```sql
SELECT 
    c.NumCom,
    COUNT(lc.NumArt) AS NombreArticles
FROM Commandes c
LEFT JOIN LigneCommandes lc ON c.NumCom = lc.NumCom
GROUP BY c.NumCom;
```

---

### Étape 2 : Supprimer les commandes **sans article**

On sélectionne les commandes **avec 0 article** et on les **supprime** :

```sql
-- 1. Afficher les commandes sans article
SELECT c.NumCom
FROM Commandes c
LEFT JOIN LigneCommandes lc ON c.NumCom = lc.NumCom
WHERE lc.NumArt IS NULL;

-- 2. Supprimer ces commandes
DELETE FROM Commandes
WHERE NumCom IN (
    SELECT c.NumCom
    FROM Commandes c
    LEFT JOIN LigneCommandes lc ON c.NumCom = lc.NumCom
    WHERE lc.NumArt IS NULL
);
```

### BONUS : Afficher un message avant suppression (via CURSOR + PRINT)

```sql
DECLARE @NumCom INT;

DECLARE curseur CURSOR FOR
SELECT c.NumCom
FROM Commandes c
LEFT JOIN LigneCommandes lc ON c.NumCom = lc.NumCom
WHERE lc.NumArt IS NULL;

OPEN curseur;
FETCH NEXT FROM curseur INTO @NumCom;

WHILE @@FETCH_STATUS = 0
BEGIN
    PRINT 'Aucun article pour la commande ' + CAST(@NumCom AS NVARCHAR) + '. Elle sera supprimée.';

    DELETE FROM Commandes WHERE NumCom = @NumCom;

    FETCH NEXT FROM curseur INTO @NumCom;
END

CLOSE curseur;
DEALLOCATE curseur;
```

---

## 📚 CONSEILS POUR L’EXAMEN :

1. **Comprends bien la logique des jointures :**
    
    - `JOIN` = fusionner deux tables sur une colonne commune
        
    - `LEFT JOIN` = garder tous les enregistrements de gauche même si pas de correspondance
        
2. **Mémorise ces opérations SQL de base :**
    
    - `SELECT`, `JOIN`, `GROUP BY`, `HAVING`, `WHERE`, `ORDER BY`, `DELETE`, `SUM()`
        
3. **CAST et CONVERT** pour afficher proprement les textes ou dates.
    
4. **Lis les énoncés comme des phrases à traduire en requête.**
    
    - Ex : “afficher les commandes sans article” → penser `LEFT JOIN` + `IS NULL`
        

---

## 🧠 RÉSUMÉ POUR RELECTURE RAPIDE

|Tâche|SQL utilisé|
|---|---|
|Afficher articles|`SELECT` + `CAST`|
|Détails commandes|`JOIN` entre 3 tables|
|Montant commande|`SUM(...) GROUP BY`|
|Vérifier commandes vides|`LEFT JOIN` + `IS NULL`|
|Supprimer commandes vides|`DELETE` avec `IN (...)` ou `CURSOR`|

---
