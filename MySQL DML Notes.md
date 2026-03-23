## **1. What is DML?**

**DML (Data Manipulation Language)** in **MySQL** is used to **insert, update, delete, and retrieve data** from tables.

**Primary DML Commands:**
- `INSERT` – Add new rows
- `UPDATE` – Modify existing rows
- `DELETE` – Remove rows
- `SELECT` – Retrieve data (often considered DQL, but used with DML)
### **DML vs DDL**
- **DDL** → Structure (`CREATE`, `ALTER`, `DROP`)
- **DML** → Data (`INSERT`, `UPDATE`, `DELETE`)

---

## **2. AUTO_INCREMENT (MySQL Identity Equivalent)**

### **What is AUTO_INCREMENT?**
- Automatically generates sequential values
- Used mostly for **PRIMARY KEY**
- MySQL version of **IDENTITY**
### **Key Rules:**

1. Works with **integer types only**
2. Only **one AUTO_INCREMENT column per table**
3. Must be **indexed** (usually PRIMARY KEY)
4. Cannot manually insert values unless explicitly allowed
5. Resets with `TRUNCATE` (important ⚠️)

---

### **Syntax**

```sql
CREATE TABLE <table_name> (
    id INT AUTO_INCREMENT PRIMARY KEY,
    -- other columns
);
```

---

### **Examples**

**Basic AUTO_INCREMENT**

```sql
CREATE TABLE Employees (
    EmployeeID INT AUTO_INCREMENT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50)
);
```

**Custom Starting Value**

```sql
ALTER TABLE Employees AUTO_INCREMENT = 1000;
```

---

### **AUTO_INCREMENT Gaps (Same as SQL Server)**

- Failed inserts still consume values

```sql
INSERT INTO Employees (FirstName) VALUES ('John'); -- ID = 1
-- Next insert fails
-- Next successful insert = ID 3
```
---

### **Reset AUTO_INCREMENT**

```sql
ALTER TABLE Employees AUTO_INCREMENT = 1;
```

⚠️ Only works if table is empty or lower than current max

---

## **3. INSERT Statement**

### **Purpose**
- Adds new rows
- Must follow **parent → child order** (foreign keys)

---

### **Syntax**

**With Columns (Recommended)**

```sql
INSERT INTO table_name (col1, col2)
VALUES (val1, val2);
```

**Without Columns**

```sql
INSERT INTO table_name
VALUES (val1, val2);
```

**Multiple Rows**

```sql
INSERT INTO table_name (col1, col2)
VALUES
    (val1, val2),
    (val3, val4);
```

---

### **Examples**

```sql
INSERT INTO Employees (FirstName, LastName)
VALUES ('Ahmed', 'Elsifi');
```

```sql
INSERT INTO Students (Name, GPA)
VALUES
    ('Ali', 3.5),
    ('Sara', 3.8);
```

---

### **INSERT with SELECT**

```sql
INSERT INTO NewEmployees (FirstName, LastName)
SELECT FirstName, LastName
FROM Employees;
```

---

### **MySQL-Specific Features 🔥**

**INSERT IGNORE (skip errors)**

```sql
INSERT IGNORE INTO Users (id, email)
VALUES (1, 'test@mail.com');
```

**REPLACE (overwrite existing row)**

```sql
REPLACE INTO Users (id, name)
VALUES (1, 'Ahmed');
```

**ON DUPLICATE KEY UPDATE (very important)**

```sql
INSERT INTO Users (id, name)
VALUES (1, 'Ahmed')
ON DUPLICATE KEY UPDATE name = 'Updated Ahmed';
```

---

### **Notes**

- Skip AUTO_INCREMENT column
- Use `DEFAULT` or omit columns
- Column order matters if not specified

---

## **4. UPDATE Statement**

### **Purpose**

- Modify existing data
---

### **Syntax**

```sql
UPDATE table_name
SET column1 = value1,
    column2 = value2
WHERE condition;
```

---

### **Examples**

```sql
UPDATE Employees
SET Salary = 60000
WHERE EmployeeID = 1;
```

---

### **Multiple Updates**

```sql
UPDATE Products
SET Price = Price * 1.1,
    UpdatedAt = NOW()
WHERE Category = 'Electronics';
```

---

### **UPDATE with JOIN (MySQL Feature)**

```sql
UPDATE Orders o
JOIN Customers c ON o.CustomerID = c.CustomerID
SET o.Status = 'Cancelled'
WHERE c.Country = 'Inactive';
```

---

### **Update All Rows**

```sql
UPDATE Products
SET UpdatedAt = NOW();
```

⚠️ Dangerous without `WHERE`

---

## **5. DELETE Statement**

### **Purpose**

- Remove rows

---

### **Syntax**

```sql
DELETE FROM table_name WHERE condition;
DELETE FROM table_name;
```

---

### **Examples**

```sql
DELETE FROM Employees
WHERE EmployeeID = 1;
```

---

### **DELETE with JOIN (MySQL Style)**

```sql
DELETE o
FROM Orders o
JOIN Customers c ON o.CustomerID = c.CustomerID
WHERE c.Country = 'Inactive';
```

---

### **DELETE with LIMIT (MySQL Feature)**

```sql
DELETE FROM Logs
ORDER BY CreatedAt
LIMIT 100;
```

---

### **Notes**

- Must follow **child → parent delete order**
- Fully logged (slower than TRUNCATE)

---

## **6. TRUNCATE Statement**

### **Purpose**

- Fast delete of ALL rows

---

### **Syntax**

```sql
TRUNCATE TABLE table_name;
```

---

### **Key Behavior in MySQL**

- Drops and recreates table internally ⚡
- Resets AUTO_INCREMENT
- Cannot be used with foreign key references

---

### **TRUNCATE vs DELETE**

|Feature|DELETE|TRUNCATE|
|---|---|---|
|WHERE|Yes|No|
|Speed|Slow|Fast|
|AUTO_INCREMENT|No reset|**Reset**|
|Logging|Row-based|Minimal|
|Foreign Keys|Allowed|❌ Restricted|
|Triggers|Yes|No|

---

## **7. Referential Actions (MySQL)**

### **FOREIGN KEY Options**

```sql
FOREIGN KEY (col)
REFERENCES parent(col)
ON DELETE <action>
ON UPDATE <action>
```

---

### **Supported Actions**

#### **1. RESTRICT / NO ACTION (Default)**

- Prevent delete/update

```sql
ON DELETE RESTRICT
```

---

#### **2. CASCADE**

```sql
ON DELETE CASCADE
```

- Deletes child rows automatically

---

#### **3. SET NULL**

```sql
ON DELETE SET NULL
```

- Sets FK to NULL

---

#### **4. SET DEFAULT**

❌ **Not supported in MySQL (important difference!)**

---

### **Example**

```sql
CREATE TABLE Orders (
    OrderID INT AUTO_INCREMENT PRIMARY KEY,
    CustomerID INT,
    FOREIGN KEY (CustomerID)
        REFERENCES Customers(CustomerID)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);
```
---

## **8. Execution / Dependency Tree**

### **Concept**
Same as SQL Server:

```
Parent → Child → Grandchild
```
---
### **Order**
**INSERT**
1. Parent
2. Child
3. Grandchild

**DELETE**
1. Grandchild
2. Child
3. Parent

**UPDATE**
- No strict order (constraints apply)
---
## **9. Practical Example**
```sql
CREATE TABLE Departments (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE Employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    salary DECIMAL(10,2),
    FOREIGN KEY (department_id)
        REFERENCES Departments(id)
        ON DELETE SET NULL
);

-- INSERT
INSERT INTO Departments (name)
VALUES ('IT'), ('HR');

INSERT INTO Employees (name, department_id, salary)
VALUES ('Ahmed', 1, 5000);

-- UPDATE
UPDATE Employees
SET salary = salary * 1.1
WHERE department_id = 1;

-- DELETE
DELETE FROM Employees WHERE id = 1;

-- TRUNCATE
TRUNCATE TABLE Employees;
```

---
## **10. Best Practices (MySQL)**
### **INSERT**
- Always specify columns
- Use `ON DUPLICATE KEY UPDATE` for upserts
- Use bulk inserts for performance
---
### **UPDATE**
- Always test with `SELECT` first
- Use transactions (`START TRANSACTION`)

---
### **DELETE**
- Avoid deleting large data without backup
- Consider soft delete (`is_deleted` column)
---
### **TRUNCATE**
- Use only when safe
- Ensure no FK dependencies
---
### **General**
```sql
START TRANSACTION;

-- your queries

ROLLBACK; -- test safely
COMMIT;   -- finalize
```