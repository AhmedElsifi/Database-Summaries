# **Complete DML (Data Manipulation Language) Reference for SQL Server**

## **1. What is DML?**

**DML (Data Manipulation Language)** is a subset of SQL used to **insert, update, delete, and retrieve** data from database tables. While often discussed with DDL, these operations focus on the actual data rather than the structure.

**Primary DML Commands:**

- `INSERT` - Add new records to tables
- `UPDATE` - Modify existing records
- `DELETE` - Remove records from tables
- `SELECT` - Retrieve data (though sometimes considered separately as DQL)

**DML vs DDL:**

- **DDL**: Defines/modifies structure (CREATE, ALTER, DROP)
- **DML**: Manipulates data within the structure (INSERT, UPDATE, DELETE)

---

## **2. IDENTITY Property (Auto-increment)**

### **What is IDENTITY?**

- Used with integer columns to automatically generate sequential values
- System-managed incrementing values
- Commonly used for primary key columns
- Also known as **AUTO_INCREMENT** in other database systems

### **Key Rules for IDENTITY:**

1. **Only for integer columns** (INT, BIGINT, SMALLINT, TINYINT)
2. **Cannot manually insert values** into IDENTITY columns (unless using `IDENTITY_INSERT`)
3. **Requires unique values** (typically used with PRIMARY KEY)
4. **Cannot be used on foreign key columns** (foreign keys reference existing values)
5. **Only one IDENTITY column per table**
6. **Resets with TRUNCATE** (but not with DELETE)

### **Syntax:**

```sql
CREATE TABLE <table_name> (
    <column_name> <integer_type> IDENTITY(<start_value>, <increment>),
    -- other columns
);
```

### **Examples:**

**Basic IDENTITY:**

```sql
CREATE TABLE Employees (
    EmployeeID INT IDENTITY(1,1) PRIMARY KEY,  -- Starts at 1, increments by 1
    FirstName NVARCHAR(50),
    LastName NVARCHAR(50)
);
```

**Custom Start and Increment:**

```sql
CREATE TABLE Orders (
    OrderID INT IDENTITY(1000, 10) PRIMARY KEY,  -- Starts at 1000, increments by 10
    OrderDate DATE,
    Amount DECIMAL(10,2)
);
```

### **The "Off-by-One" Error:**

- Occurs when an INSERT fails but the IDENTITY value has already been incremented
- Causes gaps in the sequence (not typically a problem, but can be confusing)

**Example of Off-by-One:**

```sql
-- Table with IDENTITY starting at 1
INSERT INTO Employees (FirstName, LastName) VALUES ('John', 'Doe');  -- Gets ID 1
INSERT INTO Employees (FirstName, LastName) VALUES ('Jane', 'Smith'); -- Gets ID 2
-- If next INSERT fails due to constraint violation, ID 3 is still consumed
-- Next successful INSERT gets ID 4
```

### **Resetting IDENTITY:**

```sql
-- Resets the next value to specified number
DBCC CHECKIDENT ('<table_name>', RESEED, <new_value>);
```

**Example:**

```sql
-- Reset Employees table to start from 1 again
DBCC CHECKIDENT ('Employees', RESEED, 0);
-- Next INSERT will get ID 1
```

---

## **3. INSERT Statement**

### **Purpose:**

- Adds new rows/records to a table
- Must follow **execution/insertion/deletion tree** (insert into parent tables before child tables)

### **Syntax Formats:**

**Format 1: Specifying Columns (Recommended)**

```sql
INSERT INTO <table_name> (column1, column2, column3, ...)
VALUES (value1, value2, value3, ...);
```

**Format 2: Without Column Names**

```sql
INSERT INTO <table_name>
VALUES (value1, value2, value3, ...);
```

**Format 3: Inserting Multiple Rows**

```sql
INSERT INTO <table_name> (column1, column2, column3, ...)
VALUES
    (value1a, value2a, value3a, ...),
    (value1b, value2b, value3b, ...),
    (value1c, value2c, value3c, ...);
```

### **Examples:**

**Basic INSERT with Column Names:**

```sql
-- Skip IDENTITY column (EmployeeID)
INSERT INTO Employees (FirstName, LastName, Department, Salary)
VALUES ('John', 'Doe', 'IT', 50000.00);
```

**INSERT Without Column Names:**

```sql
-- Must provide values in exact table column order
INSERT INTO Employees
VALUES ('Jane', 'Smith', 'HR', 45000.00);
-- Note: This would fail if EmployeeID is first column (IDENTITY)
```

**Inserting Multiple Rows:**

```sql
INSERT INTO Students (FirstName, LastName, EnrollmentDate, GPA)
VALUES
    ('Alice', 'Johnson', '2024-01-15', 3.5),
    ('Bob', 'Williams', '2024-01-15', 3.2),
    ('Charlie', 'Brown', '2024-01-16', 3.8);
```

**INSERT with SELECT (Copying Data):**

```sql
INSERT INTO NewEmployees (FirstName, LastName, Department)
SELECT FirstName, LastName, Department
FROM OldEmployees
WHERE HireDate > '2023-01-01';
```

### **Important Notes:**

1. **Don't insert into IDENTITY columns** (they auto-generate)
2. **Follow dependency order**: Parent tables before child tables
3. **NULL values**: Use `NULL` keyword or omit column from list
4. **DEFAULT values**: Use `DEFAULT` keyword or omit column
5. **With Format 2**: Must match exact column order and count

---

## **4. UPDATE Statement**

### **Purpose:**

- Modifies existing data in tables
- **Does NOT need to follow execution/insertion/deletion tree** (can update any table independently)
- Use WHERE clause to specify which rows to update

### **Syntax:**

```sql
UPDATE <table_name>
SET
    column1 = value1,
    column2 = value2,
    ...
WHERE <condition>;
```

### **Examples:**

**Basic UPDATE with Condition:**

```sql
UPDATE Employees
SET Salary = 55000.00
WHERE EmployeeID = 101;
```

**Multiple Column Update:**

```sql
UPDATE Products
SET
    Price = Price * 1.10,  -- Increase price by 10%
    LastUpdated = GETDATE()
WHERE Category = 'Electronics';
```

**Conditional UPDATE with Operators:**

```sql
-- Update based on comparison
UPDATE Students
SET Status = 'Graduated'
WHERE GraduationDate < GETDATE();

-- Update with multiple conditions
UPDATE Orders
SET Status = 'Cancelled'
WHERE OrderDate < '2023-01-01' AND Status = 'Pending';

-- Update using IN operator
UPDATE Employees
SET Department = 'Administration'
WHERE DepartmentID IN (1, 2, 3);
```

**UPDATE with Subquery:**

```sql
-- Increase salary for employees in high-performing departments
UPDATE Employees
SET Salary = Salary * 1.15
WHERE DepartmentID IN (
    SELECT DepartmentID
    FROM Departments
    WHERE PerformanceRating > 90
);
```

**UPDATE Without WHERE (Updates ALL rows):**

```sql
-- Updates every row in the table (USE WITH CAUTION!)
UPDATE Products
SET LastUpdated = GETDATE();
```

### **Important Notes:**

1. **Always use WHERE clause** unless you intend to update all rows
2. **Can update multiple columns** in one statement
3. **Cannot update IDENTITY columns** directly
4. **Foreign key constraints** may restrict updates
5. **Consider using transactions** for critical updates

---

## **5. DELETE Statement**

### **Purpose:**

- Removes rows from a table
- **Must follow execution/insertion/deletion tree** (delete from child tables before parent tables)
- Can be conditional or affect all rows

### **Syntax:**

```sql
-- Delete specific rows
DELETE FROM <table_name>
WHERE <condition>;

-- Delete ALL rows
DELETE FROM <table_name>;
```

### **Examples:**

**Conditional DELETE:**

```sql
DELETE FROM Employees
WHERE EmployeeID = 101;

DELETE FROM Orders
WHERE OrderDate < '2023-01-01' AND Status = 'Cancelled';
```

**DELETE with JOIN:**

```sql
-- Delete orders from specific customers
DELETE o
FROM Orders o
INNER JOIN Customers c ON o.CustomerID = c.CustomerID
WHERE c.Country = 'Inactive';
```

**DELETE Using Subquery:**

```sql
-- Delete students who haven't enrolled in any courses
DELETE FROM Students
WHERE StudentID NOT IN (
    SELECT DISTINCT StudentID
    FROM Enrollments
);
```

**Delete ALL Rows:**

```sql
DELETE FROM TemporaryData;  -- Removes all rows, keeps table structure
```

### **Important Notes:**

1. **Follow dependency order**: Child tables before parent tables
2. **Foreign key constraints** may restrict deletions
3. **DELETE is logged** (can be rolled back)
4. **Triggers may fire** on DELETE operations
5. **Space is not reclaimed** immediately (use TRUNCATE for this)

---

## **6. TRUNCATE Statement**

### **Purpose:**

- Removes ALL rows from a table quickly
- More efficient than DELETE for clearing entire tables
- Has specific limitations compared to DELETE

### **Syntax:**

```sql
TRUNCATE TABLE <table_name>;
```

### **Examples:**

**Basic TRUNCATE:**

```sql
TRUNCATE TABLE LogEntries;
TRUNCATE TABLE TemporarySessionData;
```

### **TRUNCATE vs DELETE:**

| Aspect              | DELETE                                | TRUNCATE                                            |
| ------------------- | ------------------------------------- | --------------------------------------------------- |
| **Rows Affected**   | Can delete specific rows (with WHERE) | Always removes ALL rows                             |
| **Logging**         | Fully logged (row-by-row)             | Minimally logged (page deallocations)               |
| **Speed**           | Slower (row operations)               | Faster (page operations)                            |
| **Identity Reset**  | Does NOT reset IDENTITY               | RESETS IDENTITY to seed value                       |
| **Dependency Tree** | Can use on any table                  | Can only use on **lowest level** in dependency tree |
| **Triggers**        | Fires DELETE triggers                 | Does NOT fire triggers                              |
| **Rollback**        | Can be rolled back                    | Can be rolled back (but logs minimal)               |
| **Foreign Keys**    | Works with foreign key constraints    | Requires no foreign key references                  |

### **Key Restrictions for TRUNCATE:**

1. **Cannot TRUNCATE tables** referenced by foreign keys
2. **Cannot use WHERE clause** (always removes all rows)
3. **Cannot be used** if table participates in indexed view
4. **Requires higher permissions** than DELETE

---

## **7. Referential Actions (ON DELETE / ON UPDATE)**

### **Purpose:**

- Defines behavior when referenced data is deleted or updated
- Applied to FOREIGN KEY constraints
- Controls cascade effects in related tables

### **ON DELETE Actions:**

#### **1. NO ACTION (Default)**

- Prevents deletion if referenced by child table
- Must follow execution/insertion/deletion tree

**Syntax:**

```sql
CREATE TABLE <table_name> (
    <column_name> <data_type> REFERENCES <parent_table>(<parent_column>) ON DELETE NO ACTION,
    -- or explicitly
    <column_name> <data_type> REFERENCES <parent_table>(<parent_column>) ON DELETE NO ACTION
);
```

**Example:**

```sql
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT REFERENCES Customers(CustomerID) ON DELETE NO ACTION,
    -- Delete from Customers fails if customer has orders
    OrderDate DATE
);
```

#### **2. CASCADE**

- Automatically deletes child rows when parent row is deleted
- **Dangerous**: Can cause unintended mass deletions

**Syntax:**

```sql
<column_name> <data_type> REFERENCES <parent_table>(<parent_column>) ON DELETE CASCADE
```

**Example:**

```sql
CREATE TABLE OrderItems (
    ItemID INT PRIMARY KEY,
    OrderID INT REFERENCES Orders(OrderID) ON DELETE CASCADE,
    -- If order is deleted, all its items are automatically deleted
    ProductID INT,
    Quantity INT
);
```

#### **3. SET NULL**

- Sets foreign key to NULL when referenced row is deleted
- Foreign key column must allow NULL values

**Syntax:**

```sql
<column_name> <data_type> REFERENCES <parent_table>(<parent_column>) ON DELETE SET NULL
```

**Example:**

```sql
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    DepartmentID INT REFERENCES Departments(DepartmentID) ON DELETE SET NULL,
    -- If department is deleted, employee's DepartmentID becomes NULL
    Name NVARCHAR(100)
);
```

#### **4. SET DEFAULT**

- Sets foreign key to default value when referenced row is deleted
- Column must have a DEFAULT constraint defined

**Syntax:**

```sql
<column_name> <data_type> DEFAULT <default_value>
    REFERENCES <parent_table>(<parent_column>) ON DELETE SET DEFAULT
```

**Example:**

```sql
CREATE TABLE Products (
    ProductID INT PRIMARY KEY,
    CategoryID INT DEFAULT 1
        REFERENCES Categories(CategoryID) ON DELETE SET DEFAULT,
    -- If category is deleted, Product's CategoryID becomes 1 (default)
    ProductName NVARCHAR(100)
);
```

### **ON UPDATE Actions:**

#### **1. NO ACTION (Default)**

- Prevents update if referenced by child table
- Parent primary key changes are blocked

**Syntax:**

```sql
<column_name> <data_type> REFERENCES <parent_table>(<parent_column>) ON UPDATE NO ACTION
```

#### **2. CASCADE**

- Automatically updates child foreign keys when parent primary key changes
- **Use with caution**: Can cause performance issues

**Syntax:**

```sql
<column_name> <data_type> REFERENCES <parent_table>(<parent_column>) ON UPDATE CASCADE
```

**Example:**

```sql
CREATE TABLE OrderItems (
    ItemID INT PRIMARY KEY,
    OrderID INT REFERENCES Orders(OrderID) ON UPDATE CASCADE,
    -- If OrderID changes in Orders table, it automatically updates here
    ProductID INT
);
```

#### **3. SET NULL**

- Sets foreign key to NULL when parent primary key changes

**Syntax:**

```sql
<column_name> <data_type> REFERENCES <parent_table>(<parent_column>) ON UPDATE SET NULL
```

#### **4. SET DEFAULT**

- Sets foreign key to default value when parent primary key changes

**Syntax:**

```sql
<column_name> <data_type> DEFAULT <default_value>
    REFERENCES <parent_table>(<parent_column>) ON UPDATE SET DEFAULT
```

### **Complete FOREIGN KEY with Actions Example:**

```sql
CREATE TABLE OrderDetails (
    DetailID INT PRIMARY KEY,
    OrderID INT NOT NULL,
    ProductID INT NOT NULL,
    Quantity INT,

    CONSTRAINT FK_OrderDetails_Orders
        FOREIGN KEY (OrderID)
        REFERENCES Orders(OrderID)
        ON DELETE CASCADE
        ON UPDATE NO ACTION,

    CONSTRAINT FK_OrderDetails_Products
        FOREIGN KEY (ProductID)
        REFERENCES Products(ProductID)
        ON DELETE NO ACTION
        ON UPDATE CASCADE
);
```

---

## **8. Execution/Insertion/Deletion Tree**

### **Concept:**

- Hierarchical order for data operations based on table dependencies
- **Parent tables** must be populated before **child tables**
- **Child tables** must be cleared before **parent tables**

### **Tree Structure:**

```
        Parent Table (Customers)
           /         \
Child Table 1      Child Table 2
(Orders)           (SupportTickets)
      |                  |
Grandchild Table   Grandchild Table
(OrderItems)       (TicketNotes)
```

### **Operation Order:**

**INSERT Order (Top-Down):**

1. Customers (parent)
2. Orders & SupportTickets (children)
3. OrderItems & TicketNotes (grandchildren)

**DELETE Order (Bottom-Up):**

1. OrderItems & TicketNotes (grandchildren)
2. Orders & SupportTickets (children)
3. Customers (parent)

**UPDATE Order:**

- No strict order needed
- Can update any table independently
- Constraints may restrict certain updates

---

## **9. Practical Examples**

### **Complete Database Scenario:**

```sql
-- 1. CREATE TABLES with IDENTITY and constraints
CREATE TABLE Departments (
    DepartmentID INT IDENTITY(1,1) PRIMARY KEY,
    DepartmentName NVARCHAR(100) NOT NULL
);

CREATE TABLE Employees (
    EmployeeID INT IDENTITY(100,1) PRIMARY KEY,
    FirstName NVARCHAR(50) NOT NULL,
    LastName NVARCHAR(50) NOT NULL,
    DepartmentID INT REFERENCES Departments(DepartmentID) ON DELETE SET NULL,
    HireDate DATE DEFAULT GETDATE(),
    Salary DECIMAL(10,2) CHECK (Salary > 0)
);

-- 2. INSERT data (follow dependency tree)
INSERT INTO Departments (DepartmentName)
VALUES ('IT'), ('HR'), ('Finance');

INSERT INTO Employees (FirstName, LastName, DepartmentID, Salary)
VALUES
    ('John', 'Doe', 1, 50000.00),
    ('Jane', 'Smith', 2, 45000.00),
    ('Bob', 'Johnson', 1, 55000.00);

-- 3. UPDATE data
UPDATE Employees
SET Salary = Salary * 1.10
WHERE DepartmentID = 1;  -- Give IT department 10% raise

-- 4. DELETE data
DELETE FROM Employees
WHERE EmployeeID = 102;

-- 5. TRUNCATE a log table
CREATE TABLE AuditLog (
    LogID INT IDENTITY(1,1) PRIMARY KEY,
    Action NVARCHAR(50),
    Timestamp DATETIME DEFAULT GETDATE()
);

TRUNCATE TABLE AuditLog;  -- Clear all log entries

-- 6. Handle ON DELETE CASCADE
DELETE FROM Departments WHERE DepartmentID = 1;
-- If Employees table has ON DELETE CASCADE, employees in IT dept are deleted
-- If ON DELETE SET NULL, their DepartmentID becomes NULL
-- If ON DELETE NO ACTION (default), deletion fails
```

### **Resetting IDENTITY After Testing:**

```sql
-- After testing with many INSERT/DELETE operations
DBCC CHECKIDENT ('Employees', RESEED, 100);
-- Next EmployeeID will be 101
```

---

## **10. Best Practices for DML Operations**

### **INSERT Best Practices:**

1. **Always specify column names** (except for quick ad-hoc queries)
2. **Use transactions** for multiple related inserts
3. **Validate data** before inserting
4. **Consider bulk operations** for large datasets (`BULK INSERT`)

### **UPDATE Best Practices:**

1. **Always use WHERE clause** (unless intentionally updating all rows)
2. **Test with SELECT first**: `SELECT * FROM table WHERE condition`
3. **Use transactions** and verify row counts
4. **Backup before mass updates**

### **DELETE Best Practices:**

1. **Backup data** before mass deletions
2. **Use soft delete** (status flag) instead of physical delete when possible
3. **Archive old data** rather than deleting
4. **Consider referential integrity** before deleting

### **TRUNCATE Best Practices:**

1. **Use for temporary/working tables**
2. **Backup if data might be needed**
3. **Check for foreign key constraints**
4. **Use on tables where identity reset is desired**

### **General DML Tips:**

1. **Use BEGIN TRANSACTION** and test with **ROLLBACK** before committing
2. **Monitor performance** for large operations
3. **Consider indexing impact** on DML operations
4. **Use appropriate isolation levels**
5. **Handle errors** with TRY-CATCH blocks

---

## **11. Quick Reference Cheat Sheet**

| Operation    | Syntax                                   | Identity Reset | Can Rollback | Fires Triggers  |
| ------------ | ---------------------------------------- | -------------- | ------------ | --------------- |
| **INSERT**   | `INSERT INTO table (cols) VALUES (vals)` | No             | Yes          | INSERT triggers |
| **UPDATE**   | `UPDATE table SET col=val WHERE cond`    | No             | Yes          | UPDATE triggers |
| **DELETE**   | `DELETE FROM table WHERE cond`           | No             | Yes          | DELETE triggers |
| **TRUNCATE** | `TRUNCATE TABLE table`                   | **Yes**        | Yes          | No              |

| ON DELETE Action | Description                   | NULL Allowed? | Default Required? |
| ---------------- | ----------------------------- | ------------- | ----------------- |
| **NO ACTION**    | Prevents delete if referenced | N/A           | No                |
| **CASCADE**      | Deletes child rows            | N/A           | No                |
| **SET NULL**     | Sets FK to NULL               | **Yes**       | No                |
| **SET DEFAULT**  | Sets FK to default            | N/A           | **Yes**           |

| ON UPDATE Action | Description                   |
| ---------------- | ----------------------------- |
| **NO ACTION**    | Prevents update if referenced |
| **CASCADE**      | Updates child FKs             |
| **SET NULL**     | Sets FK to NULL               |
| **SET DEFAULT**  | Sets FK to default            |

---

## **12. Common Error Solutions**

### **Identity Insert Error:**

```sql
-- Error: Cannot insert explicit value for identity column
-- Solution: Enable identity insert temporarily
SET IDENTITY_INSERT Employees ON;
INSERT INTO Employees (EmployeeID, FirstName, LastName)
VALUES (999, 'Temp', 'Employee');
SET IDENTITY_INSERT Employees OFF;
```

### **Foreign Key Violation:**

```sql
-- Error: The DELETE statement conflicted with the REFERENCE constraint
-- Solution 1: Delete child records first
DELETE FROM OrderItems WHERE OrderID = 100;
DELETE FROM Orders WHERE OrderID = 100;

-- Solution 2: Use ON DELETE CASCADE (define in table creation)
```

### **Check Constraint Violation:**

```sql
-- Error: The INSERT statement conflicted with the CHECK constraint
-- Solution: Ensure data meets constraint conditions
INSERT INTO Employees (Salary) VALUES (50000.00);  -- Must be > 0
```

### **Truncate Error with Foreign Keys:**

```sql
-- Error: Cannot truncate table because it is referenced by a FOREIGN KEY constraint
-- Solution 1: Drop and recreate constraints
-- Solution 2: Delete all rows with DELETE instead
-- Solution 3: Temporarily disable constraints (not recommended for production)
```

---

## **Summary**

DML operations are fundamental to working with data in SQL Server. Understanding the nuances of each command, their constraints, and the proper order of operations is crucial for effective database management. Always consider data integrity, performance implications, and business requirements when performing DML operations.
