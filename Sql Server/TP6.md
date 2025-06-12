
---

## 📘 What is this TP about?

You're working with **a database**, like a smart Excel sheet. The database stores:

- Articles (items to sell)
    
- Commands (orders)
    
- Lines of Command (the details: which item, how much, etc.)
    

You're asked to write **triggers**.  
A **trigger** is like an **automatic rule**. When someone tries to add, remove, or change something in the database, the trigger checks if that's allowed and does something about it.

---

## 🧱 Your Database Tables

Imagine 3 tables (boxes):

### 📦 `Articles` (the items you sell)

|Column Name|What it means|
|---|---|
|NumArt|Article ID (unique)|
|DesArt|Article name|
|Prix|Price|
|QteEnStock|Quantity in stock|
|SeuilMin|Minimum quantity allowed|
|SeuilMax|Maximum quantity allowed|

---

### 🧾 `Commandes` (customer orders)

|Column Name|What it means|
|---|---|
|NumCom|Order ID (unique)|
|DatCom|Date of the command|

---

### 📄 `LigneCommandes` (lines in the order, i.e., which item and how many)

|Column Name|What it means|
|---|---|
|NumCom|Order ID (links to `Commandes`)|
|NumArt|Article ID (links to `Articles`)|
|QteCom|Quantity of this article in this order|

---

## 🔥 What is a Trigger?

A **trigger** is a bit of SQL code that **runs automatically** when something happens in the database, like:

- Inserting (adding) a new row
    
- Deleting (removing) a row
    
- Updating (modifying) a row
    

You don’t run triggers yourself. They run **in the background**, like security guards checking what’s going on.

---

## ✅ Task-by-task — Simple Explanation

### **1. Stop orders in the future**

**The problem:**  
We don’t want someone to place an order with a date that is in the future (tomorrow or next year).

**The solution:**  
We make a **trigger** that **prevents** this from happening. If someone tries it, the trigger **blocks** it and shows an error.

---

### **2. Block deletion of a command that still has items**

**The problem:**  
You can't delete a command if it still has items inside (like trying to delete a shopping cart that has products).

**The solution:**  
We make a trigger that checks:  
"Does this command still have items?"  
If yes → **block the deletion**.

---

### **3. When someone deletes a line from a command:**

**The problem:**  
If someone deletes an article from a command:

- We must **put that quantity back into stock** (since it’s not sold).
    
- If this was the **last item** in the order → we **delete the whole command** (empty cart = delete).
    

**The solution:**  
The trigger:

1. Adds the quantity back to the stock
    
2. Deletes the command if there are no more lines
    

---

### **4. When adding an item to an order:**

**The problem:**  
Someone adds a new line:  
"2 phones in order #1" → We must:

- Check if there are at least 2 phones in stock
    
- If yes: decrease stock by 2
    
- If no: **block it** with an error
    

**The solution:**  
A trigger that checks the stock. If OK → subtract it. If not OK → block.

---

### **5. When changing the quantity of an order line:**

**The problem:**  
You already had:  
"2 phones in order #1", and you change it to 5.  
We must:

- Check if there are 3 more in stock (to go from 2 → 5)
    
- If yes: subtract more
    
- If no: block
    
- If the new quantity is less (like 2 → 1), we **give back** 1 to the stock
    

**The solution:**  
Trigger compares old and new quantity → adjusts the stock accordingly.

---

## 🧠 Simple Analogy

Imagine a **supermarket system**:

- `Articles` = what’s in the warehouse
    
- `Commandes` = customer orders
    
- `LigneCommandes` = items in the cart
    

**Triggers = rules** like:

- ❌ "You can't buy with a future date."
    
- ❌ "You can't delete an order that still has stuff in it."
    
- ✅ "If you remove an item, it goes back to the shelf."
    
- ✅ "If you add an item, make sure it’s in stock."
    
- 🔄 "If you change the quantity, stock updates too."
    

---

## 🔁 Summary of What You Do in TP6

|Task|What you're doing|Type of trigger|
|---|---|---|
|1|Block future-dated orders|INSTEAD OF INSERT|
|2|Block deleting a command with items|INSTEAD OF DELETE|
|3|Update stock + auto-delete command if empty|AFTER DELETE|
|4|Check stock and update when item is added|AFTER INSERT|
|5|Check and update stock when quantity changes|AFTER UPDATE|

---

---

## ✅ SETUP: Table Names & Columns

We’ll assume the tables already exist:

### 🧾 `Articles`

```sql
NumArt INT PRIMARY KEY,
DesArt NVARCHAR(100),
Prix DECIMAL(10,2),
QteEnStock INT,
SeuilMin INT,
SeuilMax INT
```

### 📦 `Commandes`

```sql
NumCom INT PRIMARY KEY,
DatCom DATE
```

### 📄 `LigneCommandes`

```sql
NumCom INT,
NumArt INT,
QteCom INT,
FOREIGN KEY (NumCom) REFERENCES Commandes(NumCom),
FOREIGN KEY (NumArt) REFERENCES Articles(NumArt)
```

---

# 🔥 TRIGGER 1 — Prevent orders with a future date

### ❗Goal:

Block anyone trying to insert a command (`Commandes`) if the date is in the future.

### ✅ Trigger Code:

```sql
CREATE TRIGGER trg_PreventFutureDate
ON Commandes
INSTEAD OF INSERT
AS
BEGIN
    IF EXISTS (
        SELECT 1
        FROM inserted
        WHERE DatCom > GETDATE()
    )
    BEGIN
        RAISERROR('Insertion refusée : La date de commande ne peut pas être dans le futur.', 16, 1)
        RETURN
    END

    INSERT INTO Commandes (NumCom, DatCom)
    SELECT NumCom, DatCom FROM inserted
END
```

### 💡 Explanation:

|Line|Meaning|
|---|---|
|`CREATE TRIGGER trg_PreventFutureDate`|Create a new trigger named `trg_PreventFutureDate`.|
|`ON Commandes`|This trigger is applied on the `Commandes` table.|
|`INSTEAD OF INSERT`|We replace the normal INSERT with our own logic.|
|`BEGIN ... END`|This is the body of the trigger.|
|`IF EXISTS (...)`|We check if the inserted date is **greater than today** (i.e., future).|
|`SELECT 1 FROM inserted WHERE DatCom > GETDATE()`|The `inserted` pseudo-table contains the row(s) being inserted. `GETDATE()` gives today’s date and time.|
|`RAISERROR(...)`|Stops the insert and shows an error.|
|`RETURN`|Exit the trigger.|
|`INSERT INTO Commandes (...) SELECT ...`|If everything is OK, do the real insert.|

---

# 🔥 TRIGGER 2 — Prevent deleting a command if it has items

### ❗Goal:

Don’t allow deleting a `Commande` that still has lines in `LigneCommandes`.

### ✅ Trigger Code:

```sql
CREATE TRIGGER trg_PreventDeleteWithLines
ON Commandes
INSTEAD OF DELETE
AS
BEGIN
    IF EXISTS (
        SELECT 1
        FROM LigneCommandes LC
        JOIN deleted d ON LC.NumCom = d.NumCom
    )
    BEGIN
        RAISERROR('Suppression refusée : Cette commande contient encore des articles.', 16, 1)
        RETURN
    END

    DELETE FROM Commandes
    WHERE NumCom IN (SELECT NumCom FROM deleted)
END
```

### 💡 Explanation:

|Line|Meaning|
|---|---|
|`INSTEAD OF DELETE`|Replace the normal delete with our own logic.|
|`deleted`|A special table that holds rows the user is trying to delete.|
|`JOIN LigneCommandes`|We check if there’s any line connected to this command.|
|`IF EXISTS (...)`|If we find at least 1, block the delete.|
|`RAISERROR(...)`|Display an error message.|
|`DELETE FROM Commandes`|If there are no lines, we perform the real delete.|

---

# 🔥 TRIGGER 3 — When a line is deleted: update stock, maybe delete order

### ❗Goal:

- Add the quantity back to stock
    
- If it was the **last line** for that order → delete the order
    

### ✅ Trigger Code:

```sql
CREATE TRIGGER trg_AfterDeleteLigneCommande
ON LigneCommandes
AFTER DELETE
AS
BEGIN
    -- 1. Restore stock
    UPDATE Articles
    SET QteEnStock = QteEnStock + d.QteCom
    FROM Articles A
    JOIN deleted d ON A.NumArt = d.NumArt

    -- 2. Check if the order has any remaining lines
    DELETE FROM Commandes
    WHERE NumCom IN (
        SELECT d.NumCom
        FROM deleted d
        WHERE NOT EXISTS (
            SELECT 1 FROM LigneCommandes WHERE NumCom = d.NumCom
        )
    )
END
```

### 💡 Explanation:

|Step|Action|
|---|---|
|`AFTER DELETE`|This trigger runs **after** a line is deleted.|
|`UPDATE Articles ...`|Add the deleted quantity back to stock.|
|`JOIN deleted`|Get which article was deleted and how much.|
|`DELETE FROM Commandes ...`|If the deleted line was the **last one**, delete the whole command.|
|`NOT EXISTS`|No more lines for this order? Then delete it.|

---

# 🔥 TRIGGER 4 — When a line is added: check stock, update it

### ❗Goal:

When someone adds a line:

- Make sure there’s enough stock
    
- Subtract the quantity from stock
    

### ✅ Trigger Code:

```sql
CREATE TRIGGER trg_AfterInsertLigneCommande
ON LigneCommandes
AFTER INSERT
AS
BEGIN
    IF EXISTS (
        SELECT 1
        FROM inserted i
        JOIN Articles A ON i.NumArt = A.NumArt
        WHERE i.QteCom > A.QteEnStock
    )
    BEGIN
        RAISERROR('Stock insuffisant pour l''article demandé.', 16, 1)
        ROLLBACK
        RETURN
    END

    UPDATE Articles
    SET QteEnStock = QteEnStock - i.QteCom
    FROM Articles A
    JOIN inserted i ON A.NumArt = i.NumArt
END
```

### 💡 Explanation:

|Step|Action|
|---|---|
|`AFTER INSERT`|Trigger runs after a new line is inserted.|
|`inserted`|Holds the new lines that were added.|
|`i.QteCom > A.QteEnStock`|Check if requested quantity is too high.|
|`RAISERROR + ROLLBACK`|Stop the operation and undo it.|
|`UPDATE Articles ... - i.QteCom`|Subtract from stock.|

---

# 🔥 TRIGGER 5 — When quantity of a line is changed

### ❗Goal:

If quantity is increased or decreased:

- Adjust the stock accordingly
    

### ✅ Trigger Code:

```sql
CREATE TRIGGER trg_AfterUpdateLigneCommande
ON LigneCommandes
AFTER UPDATE
AS
BEGIN
    -- 1. Check stock if quantity increased
    IF EXISTS (
        SELECT 1
        FROM inserted i
        JOIN deleted d ON i.NumCom = d.NumCom AND i.NumArt = d.NumArt
        JOIN Articles A ON i.NumArt = A.NumArt
        WHERE i.QteCom > d.QteCom AND (i.QteCom - d.QteCom) > A.QteEnStock
    )
    BEGIN
        RAISERROR('Stock insuffisant pour modifier la quantité.', 16, 1)
        ROLLBACK
        RETURN
    END

    -- 2. Adjust stock
    UPDATE Articles
    SET QteEnStock = QteEnStock + d.QteCom - i.QteCom
    FROM Articles A
    JOIN inserted i ON A.NumArt = i.NumArt
    JOIN deleted d ON i.NumCom = d.NumCom AND i.NumArt = d.NumArt
END
```

### 💡 Explanation:

|Step|Action|
|---|---|
|`AFTER UPDATE`|Runs when a line is modified.|
|`inserted`|Holds the **new** version of the line.|
|`deleted`|Holds the **old** version.|
|`i.QteCom > d.QteCom`|Quantity increased?|
|`i.QteCom - d.QteCom > stock`|Is the increase available in stock?|
|`RAISERROR + ROLLBACK`|Stop if not enough stock.|
|`QteEnStock = QteEnStock + old - new`|Adjust stock difference.|

---

## 🧠 Final Summary:

|Task|Trigger Type|Action|
|---|---|---|
|🚫 Future date insert|`INSTEAD OF INSERT`|Block future dates|
|🚫 Delete with items|`INSTEAD OF DELETE`|Block deletes with items|
|♻️ Delete item from order|`AFTER DELETE`|Add back stock, maybe delete order|
|➕ Add item to order|`AFTER INSERT`|Check stock, subtract it|
|✏️ Modify item quantity|`AFTER UPDATE`|Adjust stock correctly|

---
