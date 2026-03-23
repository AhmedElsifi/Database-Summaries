## **1. What is DDL?**

**DDL (Data Definition Language)** is the subset of SQL used to **define, modify, and remove database structures** such as:

- Databases
- Tables
- Indexes
- Constraints
- Storage structures

### **What DDL is Used For:**

- Creating and managing database schemas
- Defining table structures with columns and data types
- Establishing relationships between tables
- Setting up constraints to enforce data integrity
- Managing indexes and storage engines

**Common DDL Commands:**

- `CREATE` - Create database, table, index, etc.
- `ALTER` - Modify database or table
- `DROP` - Delete database, table, or object
- `TRUNCATE` - Remove all data from a table
- `RENAME` - Rename database objects

---

## **2. Storage & Architecture in MySQL**

|Component|Description / Usage|
|---|---|
|**Database**|Logical container for tables (stored as a folder)|
|**Table**|Stored as files inside database directory|
|**Storage Engine**|Defines how data is stored (InnoDB, MyISAM)|
|**Data Directory**|Physical location where databases are stored|

---

## **3. Storage Engines**

|Engine|Description|Usage|
|---|---|---|
|`InnoDB`|Default engine, supports transactions & foreign keys|Most applications|
|`MyISAM`|Fast reads, no transactions or FK support|Read-heavy systems|
|`MEMORY`|Stored in RAM|Temporary data|

---

## **4. Creating a Database**

### **4.1 Without Coding (GUI Method)**

Using tools like **phpMyAdmin / MySQL Workbench**:

1. Create new database
2. Set:
    - Character Set (`utf8mb4`)
    - Collation (`utf8mb4_unicode_ci`)
3. Click **Create**
---

### **4.2 With SQL Coding**

**Syntax:**

```sql
CREATE DATABASE <database_name>
CHARACTER SET <charset>
COLLATE <collation>;
```

**Example:**

```sql
CREATE DATABASE SchoolDB
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;
```

**Notes:**

- `utf8mb4` supports all characters (recommended)
- MySQL handles physical files automatically

---

## **5. Managing Databases**

### **5.1 View Databases**

```sql
SHOW DATABASES;
```

---

### **5.2 Select Database**

```sql
USE <database_name>;
```

---

### **5.3 Delete Database**

```sql
DROP DATABASE <database_name>;
```

---

## **6. Creating Tables**

### **6.1 Basic Table Syntax**

**Syntax:**

```sql
CREATE TABLE <table_name> (
    <column_name> <data_type>,
    <column_name> <data_type>
);
```

**Example:**

```sql
CREATE TABLE Students (
    StudentID INT,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    BirthDate DATE
);
```

---

### **6.2 SQL Data Types Reference**

|Data Type|Usage|
|---|---|
|`INT`, `BIGINT`|Whole numbers|
|`DECIMAL(p,s)`|Exact numbers (money)|
|`FLOAT`, `DOUBLE`|Approximate numbers|
|`CHAR(n)`|Fixed-length string|
|`VARCHAR(n)`|Variable-length string|
|`TEXT`|Large text|
|`DATE`|Date only|
|`TIME`|Time only|
|`DATETIME`|Date + time|
|`TIMESTAMP`|Auto timestamp|
|`BOOLEAN`|True/False (TINYINT)|
|`JSON`|JSON data|
|`BLOB`|Binary data|

---

## **7. Altering Tables**

### **7.1 Adding a Column**

```sql
ALTER TABLE <table_name>
ADD <column_name> <data_type>;
```

---

### **7.2 Deleting a Column**

```sql
ALTER TABLE <table_name>
DROP COLUMN <column_name>;
```

---

### **7.3 Modifying Column Type**

```sql
ALTER TABLE <table_name>
MODIFY <column_name> <new_data_type>;
```

---

### **7.4 Renaming Column**

```sql
ALTER TABLE <table_name>
RENAME COLUMN <old_name> TO <new_name>;
```

---

### **7.5 Dropping a Table**

```sql
DROP TABLE <table_name>;
```

---

## **8. Constraints in MySQL**

### **8.1 Types of Constraints**

|Constraint|Description|Syntax Example|
|---|---|---|
|`NOT NULL`|Column must have a value|`Name VARCHAR(50) NOT NULL`|
|`CHECK`|Condition must be true|`CHECK (Age >= 18)`|
|`DEFAULT`|Default value|`DEFAULT CURRENT_TIMESTAMP`|
|`UNIQUE`|Unique values only|`Email VARCHAR(100) UNIQUE`|
|`PRIMARY KEY`|Unique + NOT NULL|`ID INT PRIMARY KEY`|
|`FOREIGN KEY`|Reference another table|`FOREIGN KEY (col) REFERENCES table(col)`|

---

### **8.2 Creating Table with Constraints**

```sql
CREATE TABLE Students (
    StudentID INT AUTO_INCREMENT PRIMARY KEY,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    Age INT CHECK (Age >= 18),
    Email VARCHAR(100) UNIQUE,
    CreatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

### **8.3 CHECK Constraint Example**

```sql
CREATE TABLE Products (
    ProductID INT PRIMARY KEY,
    Price DECIMAL(10,2) CHECK (Price > 0),
    Quantity INT CHECK (Quantity >= 0)
);
```

---

### **8.4 DEFAULT Constraint Example**

```sql
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    OrderDate DATETIME DEFAULT CURRENT_TIMESTAMP,
    Status VARCHAR(20) DEFAULT 'Pending'
);
```

---

### **8.5 UNIQUE Constraint Example**

```sql
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    Email VARCHAR(100) UNIQUE
);
```

---

### **8.6 PRIMARY KEY Constraint**

- Uniquely identifies each row
- Cannot be NULL
- Only one per table (can be composite)

```sql
PRIMARY KEY (column1, column2)
```

---

## **9. AUTO_INCREMENT (MySQL Specific)**

### **What is AUTO_INCREMENT:**

- Automatically generates unique values
    

```sql
StudentID INT AUTO_INCREMENT PRIMARY KEY
```

### **Reset AUTO_INCREMENT**

```sql
ALTER TABLE Students AUTO_INCREMENT = 1;
```

---

## **10. Relationships Between Tables**

### **10.1 One-to-One**

```sql
CREATE TABLE Users (
    UserID INT PRIMARY KEY
);

CREATE TABLE Profiles (
    ProfileID INT PRIMARY KEY,
    UserID INT UNIQUE,
    FOREIGN KEY (UserID) REFERENCES Users(UserID)
);
```

---

### **10.2 One-to-Many**

```sql
CREATE TABLE Classes (
    ClassID INT PRIMARY KEY
);

CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    ClassID INT,
    FOREIGN KEY (ClassID) REFERENCES Classes(ClassID)
);
```

---

### **10.3 Many-to-Many**

```sql
CREATE TABLE StudentCourses (
    StudentID INT,
    CourseID INT,
    PRIMARY KEY (StudentID, CourseID),
    FOREIGN KEY (StudentID) REFERENCES Students(StudentID),
    FOREIGN KEY (CourseID) REFERENCES Courses(CourseID)
);
```

---

## **11. Indexes (MySQL Important Feature)**

### **What is an Index:**

- Improves query performance

---

### **Create Index**

```sql
CREATE INDEX idx_location ON users(location);
```

---

### **Create Unique Index**

```sql
CREATE UNIQUE INDEX idx_email ON users(email);
```

---

### **Drop Index**

```sql
DROP INDEX idx_location ON users;
```

---

## **12. Constraint Modification**

### **12.1 Adding Constraints**

```sql
ALTER TABLE Students
ADD CONSTRAINT unique_email UNIQUE (Email);
```

---

### **12.2 Dropping Constraints**

```sql
ALTER TABLE Students
DROP INDEX unique_email;
```

---

## **13. Database Schema**

- Logical grouping of database objects
- MySQL uses database as schema

---

## **14. Execution & Tools**

### **14.1 Selecting Database**

```sql
USE <database_name>;
```

---

### **14.2 Running Queries**

- Execute full script or selected lines
- Tools:
    - MySQL Workbench
    - phpMyAdmin
    - CLI

---

## **15. SQL Comments**

### **Single-Line**

```sql
-- comment
```

### **Multi-Line**

```sql
/* comment */
```

---

## **16. Key Concepts Summary**

### **AUTO_INCREMENT vs IDENTITY**
- MySQL: `AUTO_INCREMENT`
- SQL Server: `IDENTITY`
---

### **Case Sensitivity**
- SQL keywords are case-insensitive
- Table names may be case-sensitive (OS dependent)
---

### **Preconditions**
1. Cannot drop table with foreign key references
2. Cannot drop database in use
3. Constraints must be removed before dropping columns

---

## **✅ Quick Reference: DDL Commands**

| Command         | Syntax                           | Example                                   |
| --------------- | -------------------------------- | ----------------------------------------- |
| CREATE DATABASE | `CREATE DATABASE name`           | `CREATE DATABASE SchoolDB`                |
| DROP DATABASE   | `DROP DATABASE name`             | `DROP DATABASE SchoolDB`                  |
| CREATE TABLE    | `CREATE TABLE name (...)`        | `CREATE TABLE Students`                   |
| ALTER TABLE     | `ALTER TABLE name ADD/MODIFY`    | `ALTER TABLE Students ADD Email`          |
| DROP TABLE      | `DROP TABLE name`                | `DROP TABLE Students`                     |
| TRUNCATE        | `TRUNCATE TABLE name`            | `TRUNCATE TABLE Students`                 |
| CREATE INDEX    | `CREATE INDEX idx ON table(col)` | `CREATE INDEX idx_loc ON users(location)` |

---

## **📋 Best Practices**
1. Define primary keys for all tables
2. Use `AUTO_INCREMENT` for IDs
3. Index frequently queried columns
4. Use foreign keys for relationships
5. Normalize your schema
6. Backup before major DDL operations