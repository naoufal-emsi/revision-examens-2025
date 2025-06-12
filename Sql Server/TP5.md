
---

## 🛠️ **1. `SP_NbrArtparCommandes`**

> Affiche combien d’articles sont dans chaque commande.

### ✅ Code :

```sql
CREATE PROCEDURE SP_NbrArtparCommandes
AS
BEGIN
    SELECT NumCom, COUNT(*) AS NbrArticles
    FROM LigneCommande
    GROUP BY NumCom
END
```

### 🧠 Explication :

| Ligne                                    | Description                                                                |
| ---------------------------------------- | -------------------------------------------------------------------------- |
| `CREATE PROCEDURE SP_NbrArtparCommandes` | Crée une procédure nommée `SP_NbrArtparCommandes`.                         |
| `AS BEGIN ... END`                       | Tout le code à l’intérieur sera exécuté quand tu appelles cette procédure. |
| `SELECT NumCom, COUNT(*) AS NbrArticles` | Pour chaque commande, compte combien de lignes (articles) il y a.          |
| `FROM LigneCommande`                     | Depuis la table des lignes de commandes.                                   |
| `GROUP BY NumCom`                        | Regroupe les résultats **par commande**.                                   |

---

## 📆 **2. `SP_ComPeriode(@d1, @d2)`**

> Affiche les commandes entre deux dates.

### ✅ Code :

```sql
CREATE PROCEDURE SP_ComPeriode
    @DateDebut DATE,
    @DateFin DATE
AS
BEGIN
    SELECT * FROM Commande
    WHERE DatCom BETWEEN @DateDebut AND @DateFin
END
```

### 🧠 Explication :

|Ligne|Description|
|---|---|
|`@DateDebut`, `@DateFin`|Paramètres = les deux dates que tu veux utiliser.|
|`BETWEEN`|Cherche les commandes dont la date est **entre les deux**.|

---

## 🧮 **3. `SP_TypeComPeriode(@d1, @d2)`**

> Même chose que SP_ComPeriode + une couleur selon le **nombre total de commandes**.

### ✅ Code :

```sql
CREATE PROCEDURE SP_TypeComPeriode
    @DateDebut DATE,
    @DateFin DATE
AS
BEGIN
    DECLARE @nb INT

    SELECT * FROM Commande
    WHERE DatCom BETWEEN @DateDebut AND @DateFin

    SELECT @nb = COUNT(*) FROM Commande
    WHERE DatCom BETWEEN @DateDebut AND @DateFin

    IF @nb > 100
        PRINT 'Période rouge'
    ELSE IF @nb BETWEEN 50 AND 100
        PRINT 'Période jaune'
    ELSE
        PRINT 'Période blanche'
END
```

### 🧠 Explication :

| Ligne                       | Description                                            |
| --------------------------- | ------------------------------------------------------ |
| `DECLARE @nb INT`           | Variable pour stocker le nombre de commandes.          |
| `SELECT @nb = COUNT(*) ...` | Compte combien de commandes sont entre les deux dates. |
| `IF`                        | Affiche une couleur selon le résultat.                 |

---

## 🧾 **4. `SP_EnregistrerLigneCom(@numCom, @numArt, @qte)`**

> Crée une ligne commande **et gère tout** (vérifie stock, commande, met à jour).

### ✅ Code :

```sql
CREATE PROCEDURE SP_EnregistrerLigneCom
    @NumCom INT,
    @NumArt INT,
    @QteCommande INT
AS
BEGIN
    -- Vérifier que l'article existe
    IF NOT EXISTS (SELECT 1 FROM Article WHERE NumArt = @NumArt)
    BEGIN
        PRINT 'Erreur : article inexistant.'
        RETURN
    END

    -- Vérifier stock
    DECLARE @Stock INT
    SELECT @Stock = QteEnStock FROM Article WHERE NumArt = @NumArt
    IF @Stock < @QteCommande
    BEGIN
        PRINT 'Erreur : stock insuffisant.'
        RETURN
    END

    -- Vérifier si la commande existe, sinon la créer
    IF NOT EXISTS (SELECT 1 FROM Commande WHERE NumCom = @NumCom)
    BEGIN
        INSERT INTO Commande(NumCom, DatCom)
        VALUES (@NumCom, GETDATE())
    END

    -- Ajouter la ligne de commande
    INSERT INTO LigneCommande(NumCom, NumArt, QteCommande)
    VALUES (@NumCom, @NumArt, @QteCommande)

    -- Mettre à jour le stock
    UPDATE Article
    SET QteEnStock = QteEnStock - @QteCommande
    WHERE NumArt = @NumArt
END
```

### 🧠 Explication étape par étape :

1. **Vérifie si l’article existe** — sinon stop.
    
2. **Vérifie si le stock est suffisant** — sinon stop.
    
3. **Crée la commande si elle n’existe pas encore**.
    
4. **Ajoute la ligne de commande** (article + quantité).
    
5. **Met à jour le stock** (on soustrait la quantité commandée).
    

---

## 🧮 **5. `SP_NbrCommandes`**

> Renvoie le nombre total de commandes.

### ✅ Code :

```sql
CREATE PROCEDURE SP_NbrCommandes
AS
BEGIN
    SELECT COUNT(*) AS NbrCommandes FROM Commande
END
```

---

## 🔢 **6. `SP_NbrArtCom(@NumCom)`**

> Renvoie combien d’articles pour **une commande donnée**.

### ✅ Code :

```sql
CREATE PROCEDURE SP_NbrArtCom
    @NumCom INT
AS
BEGIN
    SELECT COUNT(*) AS NbrArticles
    FROM LigneCommande
    WHERE NumCom = @NumCom
END
```

---

## 🎨 **7. `SP_TypePeriode`**

> Donne **le type de période** selon le nombre total de commandes.

### ✅ Code :

```sql
CREATE PROCEDURE SP_TypePeriode
AS
BEGIN
    DECLARE @nb INT

    EXEC SP_NbrCommandes -- Appelle la proc qui affiche le count (affiche, mais ne renvoie pas dans variable)

    -- Solution 1: refaire le count ici
    SELECT @nb = COUNT(*) FROM Commande

    IF @nb > 100
        PRINT 'Période rouge'
    ELSE IF @nb BETWEEN 50 AND 100
        PRINT 'Période jaune'
    ELSE
        PRINT 'Période blanche'
END
```

---

## ✅ Pour les tester (exemples) :

```sql
EXEC SP_NbrArtparCommandes
EXEC SP_ComPeriode '2024-01-01', '2024-06-01'
EXEC SP_TypeComPeriode '2024-01-01', '2024-06-01'
EXEC SP_EnregistrerLigneCom 100, 1, 3
EXEC SP_NbrCommandes
EXEC SP_NbrArtCom 100
EXEC SP_TypePeriode
```

---

