
---

# What is T-SQL? The Basics in Human Terms

Imagine you have a giant digital filing cabinet — that’s your **database**. Inside, you have folders (called **tables**) where you store all kinds of information, like orders, articles, customers, stock, etc.

- **SQL** is the language you use to talk to that filing cabinet. You say things like:
    
    - “Show me all orders from last month.”
        
    - “Add this new article to the list.”
        
    - “Change the price of article #5.”
        
- **T-SQL** (Transact-SQL) is just SQL, but with some **extra powers**. It lets you:
    
    - Write **scripts** or **programs** that do more than just one query.
        
    - Use **variables** to hold data temporarily.
        
    - Use **conditions** like “if this, then do that.”
        
    - Build **procedures** (think of them like mini programs) inside the database to automate tasks.
        

Think of T-SQL as **SQL + programming superpowers** inside SQL Server.

---

# 1. **Basic SQL Commands and Their Purpose**

|Command|What It Does|Example|
|---|---|---|
|`SELECT`|Retrieves data from tables.|`SELECT * FROM Articles;`|
|`INSERT INTO`|Adds new data (a new row) into a table.|`INSERT INTO Articles VALUES (...)`|
|`UPDATE`|Changes existing data in the table.|`UPDATE Articles SET Price = 10 WHERE NumArt = 5;`|
|`DELETE`|Removes data from the table.|`DELETE FROM Articles WHERE NumArt = 5;`|
|`WHERE`|Filters which rows to act upon or display.|`SELECT * FROM Orders WHERE DatCom > '2023-01-01';`|
|`GROUP BY`|Groups rows for aggregation (like counting).|`SELECT NumCom, COUNT(*) FROM LigneCommande GROUP BY NumCom;`|
|`ORDER BY`|Sorts the output.|`SELECT * FROM Articles ORDER BY Prix DESC;`|

---

# 2. **Stored Procedures — What Are They?**

Think of a stored procedure as a **recipe** stored inside the database. Instead of writing out the whole recipe every time, you just call the name of the recipe, and the kitchen (database) does the work for you.

For example, you want to count the number of articles in each order — you create a stored procedure called `SP_NbrArtparCommandes` that does that counting for you.

Why use stored procedures?

- They save you from repeating the same code over and over.
    
- They allow complex tasks to be done with a simple call.
    
- They can take **parameters** (inputs) to customize their behavior.
    
- They run inside the database, so they are faster and more secure than running lots of queries from your application.
    

---

# 3. **How to Create a Procedure — Basic Syntax**

```sql
CREATE PROCEDURE ProcedureName
    @param1 datatype,
    @param2 datatype OUTPUT -- optional output param
AS
BEGIN
    -- Your T-SQL code here
END
```

Example:

```sql
CREATE PROCEDURE SP_NbrCommandes
AS
BEGIN
    SELECT COUNT(*) AS TotalOrders FROM Commande;
END
```

This procedure, when called, will count all the orders in the `Commande` table and return the number.

---

# 4. **Variables — Temporary Data Holders**

Inside procedures, sometimes you want to **store intermediate results** temporarily — like a scratchpad.

```sql
DECLARE @count int;         -- Declare a variable named @count of type int (integer)
SELECT @count = COUNT(*) FROM Commande;  -- Store the number of orders in @count
PRINT @count;              -- Print the value to the output window
```

---

# 5. **Conditions and Logic: IF / ELSE**

Just like in regular programming, you can make decisions:

```sql
IF @count > 100
    PRINT 'Period rouge';     -- High activity period
ELSE IF @count BETWEEN 50 AND 100
    PRINT 'Period jaune';     -- Medium activity period
ELSE
    PRINT 'Period blanche';   -- Low activity period
```

In your TP, you saw this logic in the procedure that classifies the “type of period” based on the number of commands.

---

# 6. **Grouping and Counting Rows**

Suppose you want to know how many articles are in each order — you use `GROUP BY` with `COUNT()`:

```sql
SELECT NumCom, COUNT(NumArt) AS NumberOfArticles
FROM LigneCommande
GROUP BY NumCom;
```

This will return a list of orders (`NumCom`) with the count of articles in each.

---

# 7. **Checking Existence: IF EXISTS / IF NOT EXISTS**

You must **check if something exists before you act** to avoid errors.

Example:  
Before adding a new order line, check if the article exists and if there’s enough stock:

```sql
IF NOT EXISTS (SELECT 1 FROM Article WHERE NumArt = @articleNumber)
BEGIN
    PRINT 'Error: Article does not exist.';
    RETURN; -- Stop executing the procedure
END

IF (SELECT QteEnStock FROM Article WHERE NumArt = @articleNumber) < @quantity
BEGIN
    PRINT 'Error: Not enough stock.';
    RETURN;
END
```

---

# 8. **Inserting and Updating Data**

If checks are OK, you can insert the order line and update the stock:

```sql
INSERT INTO LigneCommande (NumCom, NumArt, QteCommande)
VALUES (@orderNumber, @articleNumber, @quantity);

UPDATE Article
SET QteEnStock = QteEnStock - @quantity
WHERE NumArt = @articleNumber;
```

---

# 9. **sing Parameters Properly**

Parameters are inputs to your procedures — think of them like knobs you turn to change the behavior.

Example:

```sql
CREATE PROCEDURE SP_ComPeriode
    @startDate DATE,
    @endDate DATE
AS
BEGIN
    SELECT * FROM Commande
    WHERE DatCom BETWEEN @startDate AND @endDate;
END
```

This procedure lets you get all orders between two dates you give.

---

# 10. **Common Mistakes and How to Avoid Them**

|Mistake|Explanation and Fix|
|---|---|
|Forgetting to use `BEGIN ... END` after `IF` with multiple statements|T-SQL requires grouping multiple commands.|
|Not checking if record exists before inserting|Causes duplicate data or errors.|
|Using a variable before declaring it|Always `DECLARE` variables before using them.|
|Assigning multiple rows to one variable|Use `SELECT TOP 1` or aggregation functions instead.|
|Forgetting to update stock after inserting order lines|Leads to wrong inventory data.|
|Using wrong data types for parameters|Make sure parameter types match column types exactly.|

---

# 11. **Practical Example from Your TP**

Let me show you how **procedure 4** (`SP_EnregistrerLigneCom`) might look and explain it line by line:

```sql
CREATE PROCEDURE SP_EnregistrerLigneCom
    @NumCom INT,
    @NumArt INT,
    @QteCommande INT
AS
BEGIN
    -- Check if article exists
    IF NOT EXISTS (SELECT 1 FROM Article WHERE NumArt = @NumArt)
    BEGIN
        PRINT 'Error: Article does not exist.';
        RETURN;
    END

    -- Check if stock is enough
    IF (SELECT QteEnStock FROM Article WHERE NumArt = @NumArt) < @QteCommande
    BEGIN
        PRINT 'Error: Not enough stock.';
        RETURN;
    END

    -- Check if order exists, if not create it
    IF NOT EXISTS (SELECT 1 FROM Commande WHERE NumCom = @NumCom)
    BEGIN
        INSERT INTO Commande (NumCom, DatCom)
        VALUES (@NumCom, GETDATE());  -- GETDATE() inserts current date
    END

    -- Insert the new order line
    INSERT INTO LigneCommande (NumCom, NumArt, QteCommande)
    VALUES (@NumCom, @NumArt, @QteCommande);

    -- Update stock
    UPDATE Article
    SET QteEnStock = QteEnStock - @QteCommande
    WHERE NumArt = @NumArt;

    PRINT 'Order line added successfully.';
END
```

**Explanation:**

- Lines 4-8: We verify if the article exists — if not, we print an error and stop.
    
- Lines 10-14: Check if enough stock is available — if not, print error and stop.
    
- Lines 16-20: Check if the order exists — if not, create it with the current date.
    
- Lines 22-24: Add the new order line with the specified quantity.
    
- Lines 26-29: Deduct the ordered quantity from stock.
    
- Line 31: Confirm success.
    

---

# 12. **Things to Remember Always**

- **Start simple, test small.** Write one thing at a time, run it, check output.
    
- **Always handle errors or invalid data.** Don’t just assume the user inputs are correct.
    
- **Use comments** (`-- this is a comment`) to explain tricky parts or your intent.
    
- **Practice writing queries with sample data.** Don’t try to learn everything at once.
    
- **Understand the data model** (tables and columns) before writing queries or procedures.
    
- **Ask yourself: “What happens if there’s no data?” or “What if quantity is zero?”**
    

---

# 13. **Summary — What You Should Be Able to Do Now**

- Write **basic SQL queries** (`SELECT`, `INSERT`, `UPDATE`, `DELETE`).
    
- Create **stored procedures** with parameters and internal logic.
    
- Use **variables** inside procedures to hold temporary data.
    
- Use **conditional logic** (`IF`, `ELSE`) to make decisions.
    
- Use **aggregation** (`COUNT`, `GROUP BY`) to summarize data.
    
- Check for existence with `IF EXISTS` before inserting or updating.
    
- Update related tables consistently (like stock after order lines).
    
- Handle error cases gracefully (
    

print messages, stop execution).

---

## 🔢 NUMERIC TYPES

These are used for **whole numbers** or **numbers with decimals**.

| Data Type       | Example                                    | What It Does                                                                        |
| --------------- | ------------------------------------------ | ----------------------------------------------------------------------------------- |
| `int`           | `DECLARE @age int = 25;`                   | Declares an integer variable that can store whole numbers. Max range: ~2.1 billion. |
| `bigint`        | `DECLARE @views bigint = 9000000000;`      | Use when your number exceeds the `int` limit. Think very large counts.              |
| `smallint`      | `DECLARE @score smallint = 32000;`         | Smaller whole numbers. Max: 32,767. Saves space if you don’t need big numbers.      |
| `tinyint`       | `DECLARE @flag tinyint = 1;`               | Only for 0–255. Great for binary flags (on/off) or small counters.                  |
| `decimal(p, s)` | `DECLARE @price decimal(6,2) = 1234.56;`   | Precision (6 digits total), scale (2 after decimal). Good for **money**.            |
| `numeric(p, s)` | `DECLARE @amount numeric(8,3) = 4567.123;` | Identical to `decimal`. Pick either.                                                |
| `float`         | `DECLARE @pi float = 3.14159265;`          | For **approximate** floating-point numbers. May not be 100% precise.                |
| `real`          | `DECLARE @ratio real = 0.75;`              | Less precise and takes less space than `float`. Good enough for rough values.       |

---

## 🔤 TEXT / STRING TYPES

Used for **names, descriptions, emails**, etc.

|Data Type|Example|What It Does|
|---|---|---|
|`char(n)`|`DECLARE @code char(5) = 'A123 ';`|Fixed length. Always uses 5 characters, even if value is shorter.|
|`varchar(n)`|`DECLARE @email varchar(100) = 'user@example.com';`|Variable length. Saves space for shorter strings. Most commonly used.|
|`text` (deprecated)|`DECLARE @bio text = 'Long story...';`|Old type for large text. Avoid. Use `varchar(max)` instead.|
|`nchar(n)`|`DECLARE @symbol nchar(2) = N'✓';`|Fixed-length **Unicode**. Supports symbols from all languages.|
|`nvarchar(n)`|`DECLARE @name nvarchar(50) = N'Naoufal';`|Most flexible. Variable-length **Unicode**. Use this for user input.|
|`ntext` (deprecated)|`DECLARE @story ntext;`|Old large Unicode text. Replace with `nvarchar(max)`.|

> 💡 Use `N'...'` for **Unicode** strings with `nchar` / `nvarchar`.

---

## 📅 DATE & TIME TYPES

Work with **dates, times**, or both.

|Data Type|Example|What It Does|
|---|---|---|
|`date`|`DECLARE @dob date = '2000-01-01';`|Stores date only. No time.|
|`time`|`DECLARE @start time = '14:30:00';`|Stores time only. No date.|
|`datetime`|`DECLARE @created datetime = '2025-06-09 14:00:00';`|Classic date + time. Less precise.|
|`smalldatetime`|`DECLARE @stamp smalldatetime = '2025-06-09 14:00';`|Less precision than `datetime`. Round to the nearest minute.|
|`datetime2`|`DECLARE @logtime datetime2 = '2025-06-09 14:30:45.1234567';`|Newer, **more accurate** datetime. Better than `datetime`.|
|`datetimeoffset`|`DECLARE @tztime datetimeoffset = '2025-06-09 14:30:00 +01:00';`|Tracks timezone offset too. Good for global apps.|

---

## ✅ BOOLEAN-LIKE TYPE

|Data Type|Example|What It Does|
|---|---|---|
|`bit`|`DECLARE @is_active bit = 1;`|Boolean type. `1` = true, `0` = false. Can also be `NULL`.|

---

## 🧬 OTHER TYPES

|Data Type|Example|What It Does|
|---|---|---|
|`binary(n)`|`DECLARE @hash binary(16);`|Fixed-size binary. Use for raw bytes.|
|`varbinary(n)`|`DECLARE @img varbinary(max);`|Variable-length binary (images, files, etc.).|
|`uniqueidentifier`|`DECLARE @uuid uniqueidentifier = NEWID();`|Stores a GUID (e.g., user ID or session token).|
|`xml`|`DECLARE @xmlData xml = '<user><id>1</id></user>';`|Used for storing and querying XML.|
|`sql_variant`|`DECLARE @data sql_variant = 123.45;`|Can hold multiple types. Rarely used.|
|`cursor`|`DECLARE myCursor CURSOR FOR SELECT * FROM users;`|Declares a cursor (row-by-row loop). Avoid unless necessary.|
|`table`|||

```sql
DECLARE @users TABLE (
    id int,
    name varchar(50)
);
```

| Declares a table variable — you can use this like a temporary table in code. |

---

## 🧪 Summary

- Use `int`, `decimal`, or `float` for numbers depending on precision and size.
    
- Use `varchar(n)` or `nvarchar(n)` for flexible text.
    
- Use `date`, `time`, or `datetime2` depending on how precise you need.
    
- Use `bit` for true/false values.
    
- Use `table` if you want to store rows temporarily.
    

---
Let’s dig deep — **thorough**, clear, no shortcuts.

---

# 🧠 What is `CASE` in T-SQL?

The `CASE` expression in **Transact-SQL (T-SQL)** is a **control flow expression** that evaluates **conditions** and **returns a value** when the first matching condition is true.

It is NOT a statement (like `IF`); it's an **expression**. This means you can use it:

- Inside a `SELECT`
    
- In a `WHERE` clause
    
- In an `ORDER BY`
    
- In `UPDATE`, `SET`, and more
    

---

## 🔹 WHY use `CASE`?

Because you want to return **different results** depending on the value of a column or an expression.

---

# 📘 Syntax

### ✅ 1. SIMPLE `CASE`

Compare one expression to multiple **specific values**.

```sql
CASE input_expression
  WHEN value1 THEN result1
  WHEN value2 THEN result2
  ...
  ELSE default_result
END
```

### ✅ 2. SEARCHED `CASE`

Use **conditions** (like comparisons) instead of just values.

```sql
CASE 
  WHEN condition1 THEN result1
  WHEN condition2 THEN result2
  ...
  ELSE default_result
END
```

---

# ✅ 1. SIMPLE `CASE` Example

```sql
SELECT 
  name,
  grade,
  CASE grade
    WHEN 'A' THEN 'Excellent'
    WHEN 'B' THEN 'Good'
    WHEN 'C' THEN 'Average'
    ELSE 'Fail'
  END AS grade_description
FROM students;
```

### 🔍 Explanation:

- `grade` is a column (`input_expression`).
    
- `WHEN` compares it to each value.
    
- If it matches `'A'`, it returns `'Excellent'`, etc.
    
- If it matches nothing, it returns `'Fail'`.
    

✅ Use **Simple CASE** when:

- You're comparing the **same column or variable** to different values.
    

---

# ✅ 2. SEARCHED `CASE` Example

```sql
SELECT 
  name,
  age,
  CASE 
    WHEN age < 13 THEN 'Child'
    WHEN age BETWEEN 13 AND 19 THEN 'Teen'
    WHEN age >= 20 THEN 'Adult'
    ELSE 'Unknown'
  END AS age_group
FROM people;
```

### 🔍 Explanation:

- This form doesn’t use a single expression.
    
- Each `WHEN` includes a **full condition**.
    
- It returns a label depending on the age.
    

✅ Use **Searched CASE** when:

- You need **flexible logic**
    
- You're using **comparison operators**, like `<`, `>`, `=`, `BETWEEN`, etc.
    

---

# 🔄 CASE in `ORDER BY`

You can use `CASE` to control how rows are sorted:

```sql
SELECT name, is_admin
FROM users
ORDER BY 
  CASE 
    WHEN is_admin = 1 THEN 0  -- Admins first
    ELSE 1
  END;
```

✅ This pushes admins to the top, because 0 comes before 1.

---

# 🧮 CASE Inside Aggregates (e.g. `COUNT`, `SUM`)

```sql
SELECT 
  department,
  COUNT(*) AS total_employees,
  SUM(
    CASE 
      WHEN gender = 'F' THEN 1 
      ELSE 0 
    END
  ) AS female_employees
FROM employees
GROUP BY department;
```

### 🔍 Explanation:

- `CASE` checks if gender is `'F'`
    
- If true → returns `1` (to be summed)
    
- If false → returns `0`
    

This is how you do **conditional counting**.

---

# 🧱 CASE in WHERE Clause

```sql
SELECT * FROM orders
WHERE 
  status = 
    CASE 
      WHEN GETDATE() < '2025-12-31' THEN 'PENDING'
      ELSE 'ARCHIVED'
    END;
```

✅ This filters orders **differently depending on the date**.

---

# 💥 CASE in UPDATE Statements

```sql
UPDATE employees
SET salary = 
  CASE 
    WHEN experience_years > 10 THEN salary * 1.2
    WHEN experience_years BETWEEN 5 AND 10 THEN salary * 1.1
    ELSE salary
  END;
```

✅ This gives conditional salary raises.

---

# 🔐 CASE with NULL Values

```sql
SELECT 
  name,
  bonus,
  CASE 
    WHEN bonus IS NULL THEN 'No Bonus'
    ELSE 'Bonus Awarded'
  END AS bonus_status
FROM employees;
```

✅ Always handle `NULL` carefully — `=` won’t match it; use `IS NULL`.

---

# 🔄 Difference Between `CASE` and `IF`

|Feature|`CASE`|`IF`|
|---|---|---|
|Used in|Expressions (e.g. SELECT, WHERE)|Blocks of T-SQL code|
|Returns|A value|Executes statements|
|Can be nested|Yes|Yes|
|Inline?|Yes|No|

---

# 🧠 Pro Tips

- Always include `ELSE`, or SQL will return `NULL` when no match.
    
- You can nest CASE inside another CASE.
    
- Prefer `CASE` when building **queries**. Use `IF` for stored procedures and flow control.
    

---

## ✅ FULL USE-CASE: Dynamic Labels in a Report

```sql
SELECT 
  product_name,
  quantity,
  CASE 
    WHEN quantity = 0 THEN 'Out of stock'
    WHEN quantity < 10 THEN 'Low stock'
    ELSE 'In stock'
  END AS stock_status
FROM products;
```

➡️ Clean, readable, and helps UI show proper labels.

---
