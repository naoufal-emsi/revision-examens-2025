
---

## ✅ TP3 – Introduction to T-SQL (Total Beginner-Friendly Guide)

### 📘 General Context:

You’re working with a **SQL Server database** called `GestionCom`. The goal is to:

* Create and manage this database.
* Write T-SQL code (Transact-SQL is Microsoft’s version of SQL) to perform operations like calculations, data filtering, conditional logic, and more.

---

## 🧩 PART 1: Create the Database and Tables

### 🧾 Requirement:

> Créez la base de données conformément aux types de données du script d’insertion.

### 💡 Explanation:

To write or test any SQL, you need a database and tables. This part assumes there’s a script (not included here) for inserting data. You’ll need to create your database and define the table structures (schemas) based on that.

Let’s **create the database and example tables**.

### 🛠️ Step-by-Step Code:

```sql
-- Create the database
CREATE DATABASE GestionCom;
GO

-- Use the database
USE GestionCom;
GO

-- Create some example tables based on what the TP needs:
-- We'll assume some table names and columns based on the context of the TP.

-- Table: Articles
CREATE TABLE Articles (
    NumArt INT PRIMARY KEY,
    NomArt NVARCHAR(100),
    QteEnStock INT,
    SeuilMin INT,
    Prix DECIMAL(10,2)
);

-- Table: Commandes
CREATE TABLE Commandes (
    NumCom INT PRIMARY KEY,
    DatCom DATE
);

-- Table: LigneCommandes (details of each order)
CREATE TABLE LigneCommandes (
    NumCom INT,
    NumArt INT,
    QteCom INT,
    FOREIGN KEY (NumCom) REFERENCES Commandes(NumCom),
    FOREIGN KEY (NumArt) REFERENCES Articles(NumArt)
);
```

> ✅ This part just sets up the structure. If your school gave you an SQL insert script, run that **after** these CREATE statements.

---

## 🧩 PART 2: Calculate the amount of a command and display its type

### 🧾 Requirement:

> Calculez le montant de la commande numéro 102. Affichez "Commande Normale" si < 100, "Commande Spéciale" si > 100.

### 💡 Explanation:

You must:

* Get the total value of order 102.
* Check if it’s ≤ 100 or > 100.
* Display the message accordingly.

### 🛠️ Code:

```sql
DECLARE @Montant DECIMAL(10,2);

-- Calculate total of the order
SELECT @Montant = SUM(A.Prix * L.QteCom)
FROM Articles A
JOIN LigneCommandes L ON A.NumArt = L.NumArt
WHERE L.NumCom = 102;

-- Check and display type
IF @Montant <= 100
    PRINT 'Commande Normale';
ELSE
    PRINT 'Commande Spéciale';
```

> ✅ Here we use a **variable** `@Montant` to store the total. We calculate it using a `JOIN` and then use `IF` for conditional logic.

---

## 🧩 PART 3: List all orders with their totals and type

### 🧾 Requirement:

> Affichez la liste des commandes avec leur total et leur type (Normale ≤ 1000, Spéciale > 1000)

### 💡 Explanation:

Loop over all orders. For each:

* Compute the total.
* Add a column "Type" based on amount.

### 🛠️ Code:

```sql
SELECT 
    C.NumCom,
    C.DatCom,
    SUM(A.Prix * L.QteCom) AS Montant,
    CASE 
        WHEN SUM(A.Prix * L.QteCom) <= 1000 THEN 'Commande Normale'
        ELSE 'Commande Spéciale'
    END AS Type
FROM Commandes C
JOIN LigneCommandes L ON C.NumCom = L.NumCom
JOIN Articles A ON A.NumArt = L.NumArt
GROUP BY C.NumCom, C.DatCom;
```

> ✅ This query uses `GROUP BY` to group by order and calculate totals. The `CASE` statement is used to define the "Type".

---

## 🧩 PART 4: Delete article 5 from order 105 and update stock

### 🧾 Requirement:

> Supprimez l'article 5 de la commande 105, affichez stock et quantité commandée, mettez à jour stock, supprimez commande si vide.

### 💡 Explanation:

Steps:

1. Get quantity of article 5 in order 105.
2. Display current stock and quantity ordered.
3. Add back to stock (since we’re deleting).
4. Delete the line.
5. If no more lines for that order, delete the order.

### 🛠️ Code:

```sql
DECLARE @Qte INT;
DECLARE @Stock INT;

-- Get quantity and stock
SELECT 
    @Qte = L.QteCom,
    @Stock = A.QteEnStock
FROM LigneCommandes L
JOIN Articles A ON A.NumArt = L.NumArt
WHERE L.NumCom = 105 AND L.NumArt = 5;

PRINT 'Quantité commandée: ' + CAST(@Qte AS VARCHAR);
PRINT 'Stock actuel: ' + CAST(@Stock AS VARCHAR);

-- Update stock
UPDATE Articles
SET QteEnStock = QteEnStock + @Qte
WHERE NumArt = 5;

-- Delete line
DELETE FROM LigneCommandes
WHERE NumCom = 105 AND NumArt = 5;

-- Check if command is now empty
IF NOT EXISTS (SELECT * FROM LigneCommandes WHERE NumCom = 105)
BEGIN
    DELETE FROM Commandes WHERE NumCom = 105;
    PRINT 'Commande supprimée';
END
ELSE
BEGIN
    PRINT 'Erreur: Commande contient encore des articles';
END
```

> ✅ This part tests your understanding of variables, conditionals, transactions, and joins.

---

## 🧩 PART 5: Top 5 highest orders stored in a temp table

### 🧾 Requirement:

> Stocker dans une table temporaire les 5 meilleures commandes (montant le plus élevé) triées par montant décroissant.

### 💡 Explanation:

You create a **temporary table** (only exists during the session) and insert the 5 most expensive orders.

### 🛠️ Code:

```sql
-- Create temporary table
CREATE TABLE #Top5Commandes (
    NumCom INT,
    DatCom DATE,
    MontantCom DECIMAL(10,2)
);

-- Insert top 5
INSERT INTO #Top5Commandes
SELECT TOP 5 
    C.NumCom,
    C.DatCom,
    SUM(A.Prix * L.QteCom) AS MontantCom
FROM Commandes C
JOIN LigneCommandes L ON C.NumCom = L.NumCom
JOIN Articles A ON A.NumArt = L.NumArt
GROUP BY C.NumCom, C.DatCom
ORDER BY MontantCom DESC;

-- Show contents
SELECT * FROM #Top5Commandes;
```

> ✅ `#Top5Commandes` is a temporary table. `TOP 5` with `ORDER BY DESC` gives highest values first.

---

## 🧩 PART 6: Reorder if stock is below minimum

### 🧾 Requirement:

* Check if any article has stock ≤ minimum
* Create a new command (NumCom = max + 1, date = today)
* Insert a line in the command with quantity = 3 × seuil minimum

### 💡 Explanation:

You're simulating a restock. This requires:

* `GETDATE()` for current date
* `IDENTITY` or calculating `MAX(NumCom)` + 1
* Conditional logic and insert

### 🛠️ Code:

```sql
DECLARE @NumArt INT, @SeuilMin INT, @NumCom INT;

-- Check if there's a product below threshold
SELECT TOP 1 
    @NumArt = NumArt,
    @SeuilMin = SeuilMin
FROM Articles
WHERE QteEnStock <= SeuilMin;

IF @NumArt IS NOT NULL
BEGIN
    -- Get new command number
    SELECT @NumCom = ISNULL(MAX(NumCom), 0) + 1 FROM Commandes;

    -- Insert new command
    INSERT INTO Commandes (NumCom, DatCom)
    VALUES (@NumCom, GETDATE());

    -- Insert command line
    INSERT INTO LigneCommandes (NumCom, NumArt, QteCom)
    VALUES (@NumCom, @NumArt, @SeuilMin * 3);

    PRINT 'Commande créée pour réapprovisionnement';
END
ELSE
BEGIN
    PRINT 'Aucun article en dessous du seuil';
END
```

---

## ✅ Summary of What You Learned

| Concept                      | Description                       |
| ---------------------------- | --------------------------------- |
| `CREATE DATABASE`            | Create a new database             |
| `JOIN`                       | Combine data from multiple tables |
| `SUM`, `GROUP BY`            | Aggregate data                    |
| `CASE`                       | Conditional logic inside SELECT   |
| `IF`, `PRINT`                | Conditional branching in T-SQL    |
| `TEMP TABLE`                 | Store intermediate results        |
| `VARIABLES`                  | Store data inside scripts         |
| `DELETE`, `UPDATE`, `INSERT` | Data manipulation                 |
| `GETDATE()`                  | Get current system date           |
| `ISNULL()`                   | Avoid null problems               |
| `MAX()`                      | Get the highest value             |

---
