# **Ultimate DDL Reference for SQL / SSMS**

## **1. What is DDL?**

**DDL (Data Definition Language)** is the subset of SQL used to **define, modify, and remove database structures**, such as:

- Databases
- Tables
- Indexes
- Constraints
- File structures

**Common DDL commands:**

- `CREATE` — create database, table, file, etc.
- `ALTER` — modify database, table, or object
- `DROP` — delete database, table, or object
- `TRUNCATE` — remove all data from a table

---

## **2. Database File Types in SQL Server**

| File Type                  | Description / Usage                                                 |
| -------------------------- | ------------------------------------------------------------------- |
| **Primary (.mdf)**         | Main data file of the database, contains schema and data.           |
| **Secondary (.ndf)**       | Optional additional data file to spread data across multiple disks. |
| **Transaction Log (.ldf)** | Stores all transactions for recovery and rollback purposes.         |

---

## **3. Creating a Database in SSMS**

### **3.1 Without Coding (GUI)**

1. Right-click **Databases** → **New Database…**
2. Enter **Database Name**
3. Configure **Initial Size**, **Autogrowth**, **Max Size**, **File Type**, **Filegroup**
4. Click **OK**

### **3.2 With SQL Coding**

```sql
CREATE DATABASE SchoolDB
ON PRIMARY (
    NAME = 'SchoolDB_data',
    FILENAME = 'C:\SQLData\SchoolDB.mdf',
    SIZE = 10MB,
    FILEGROWTH = 5MB,
    MAXSIZE = 100MB
)
LOG ON (
    NAME = 'SchoolDB_log',
    FILENAME = 'C:\SQLData\SchoolDB_log.ldf',
    SIZE = 5MB,
    FILEGROWTH = 2MB,
    MAXSIZE = 50MB
);
```

**Notes:**

- `PRIMARY` → primary data file
- `LOG ON` → transaction log
- You can specify multiple data files and log files.

---

## **4. Managing Databases / Files**

### **4.1 Viewing Database Files**

```sql
-- All details
SELECT * FROM sys.database_files;

-- Only names and physical paths
SELECT name, physical_name FROM sys.database_files;
```

### **4.2 Adding a New File**

```sql
ALTER DATABASE SchoolDB
ADD FILE (
    NAME = 'SchoolDB_Data2',
    FILENAME = 'C:\SQLData\SchoolDB_Data2.ndf',
    SIZE = 10MB,
    FILEGROWTH = 5MB,
    MAXSIZE = 50MB
);
```

### **4.3 Modifying an Existing File**

```sql
ALTER DATABASE SchoolDB
MODIFY FILE (
    NAME = 'SchoolDB_data',
    SIZE = 20MB,
    MAXSIZE = 200MB,
    FILEGROWTH = 10MB
);
```

### **4.4 Removing a File**

> If the file contains data, first move it using `DBCC SHRINKFILE`:

```sql
DBCC SHRINKFILE('SchoolDB_Data2', EMPTYFILE);
ALTER DATABASE SchoolDB
REMOVE FILE SchoolDB_Data2;
```

### **4.5 Deleting a Database**

**Precondition:** You **cannot be using the database**.

```sql
USE master;
DROP DATABASE SchoolDB;
```

---

## **5. Creating Tables**

### **5.1 Basic Table Syntax**

```sql
CREATE TABLE Students (
    StudentID INT,
    FirstName NVARCHAR(50),
    LastName NVARCHAR(50),
    BirthDate DATE,
    Gender CHAR(1)
);
```

---

## **5.2 Expanded Data Types Overview**

| Data Type                       | Storage Size         | Range / Notes                                             | Usage Example                             | Notes / Tips                                 |
| ------------------------------- | -------------------- | --------------------------------------------------------- | ----------------------------------------- | -------------------------------------------- |
| `INT`                           | 4 bytes              | -2,147,483,648 to 2,147,483,647                           | `Age INT`                                 | Most common integer type                     |
| `BIGINT`                        | 8 bytes              | -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807   | `Population BIGINT`                       | For very large numbers                       |
| `SMALLINT`                      | 2 bytes              | -32,768 to 32,767                                         | `Score SMALLINT`                          | Use for smaller integers to save space       |
| `TINYINT`                       | 1 byte               | 0 to 255                                                  | `Rating TINYINT`                          | Good for flags or small counts               |
| `DECIMAL(p,s)` / `NUMERIC(p,s)` | Varies               | Fixed precision, `p` = total digits, `s` = decimal places | `Price DECIMAL(10,2)`                     | Use for money, financial calculations        |
| `FLOAT`                         | 4 or 8 bytes         | Approximate floating-point                                | `Temperature FLOAT`                       | Not exact; for scientific calculations       |
| `REAL`                          | 4 bytes              | Approximate floating-point                                | `Measurement REAL`                        | Less precision than `FLOAT`                  |
| `CHAR(n)`                       | n bytes              | Fixed-length string, n characters                         | `Gender CHAR(1)`                          | Always stores n characters, pads with spaces |
| `VARCHAR(n)`                    | Variable             | Variable-length string, max n characters                  | `Name VARCHAR(50)`                        | Most common text type                        |
| `NVARCHAR(n)`                   | Variable             | Unicode string (supports multiple languages)              | `Address NVARCHAR(100)`                   | Use for international text                   |
| `TEXT`                          | Variable, up to 2 GB | Large text data                                           | `Description TEXT`                        | Deprecated; use `VARCHAR(MAX)` instead       |
| `NTEXT`                         | Variable, up to 2 GB | Unicode large text                                        | `Notes NTEXT`                             | Deprecated; use `NVARCHAR(MAX)` instead      |
| `DATE`                          | 3 bytes              | 0001-01-01 to 9999-12-31                                  | `BirthDate DATE`                          | Stores date only                             |
| `TIME`                          | 5 bytes              | 00:00:00.0000000 to 23:59:59.9999999                      | `StartTime TIME`                          | Stores time only                             |
| `DATETIME`                      | 8 bytes              | 1753-01-01 to 9999-12-31, 3.33ms precision                | `CreatedAt DATETIME`                      | Old format, combines date + time             |
| `DATETIME2(p)`                  | 6-8 bytes            | 0001-01-01 to 9999-12-31, user-defined fractional seconds | `UpdatedAt DATETIME2(3)`                  | More precise than `DATETIME`                 |
| `SMALLDATETIME`                 | 4 bytes              | 1900-01-01 to 2079-06-06, 1 min precision                 | `MeetingTime SMALLDATETIME`               | Less precise, less storage                   |
| `BIT`                           | 1 bit                | 0, 1, or NULL                                             | `IsActive BIT`                            | Perfect for true/false flags                 |
| `MONEY`                         | 8 bytes              | -922,337,203,685,477.5808 to 922,337,203,685,477.5807     | `Salary MONEY`                            | Stores currency values exactly               |
| `SMALLMONEY`                    | 4 bytes              | -214,748.3648 to 214,748.3647                             | `Bonus SMALLMONEY`                        | Smaller money range                          |
| `BINARY(n)`                     | n bytes              | Fixed-length binary data                                  | `ImageHash BINARY(16)`                    | Stores binary data like hashes               |
| `VARBINARY(n)`                  | Variable             | Variable-length binary                                    | `Photo VARBINARY(MAX)`                    | Use for images, files                        |
| `XML`                           | Variable             | Stores XML data                                           | `Settings XML`                            | Useful for hierarchical data                 |
| `UNIQUEIDENTIFIER`              | 16 bytes             | Globally unique ID                                        | `UserID UNIQUEIDENTIFIER DEFAULT NEWID()` | Use for GUIDs, PKs across systems            |

---

## **6. Altering Tables**

### **6.1 Adding a Column**

```sql
ALTER TABLE Students
ADD Email NVARCHAR(100);
```

### **6.2 Deleting a Column**

```sql
ALTER TABLE Students
DROP COLUMN Email;
```

### **6.3 Modifying Column Type**

```sql
ALTER TABLE Students
ALTER COLUMN FirstName NVARCHAR(100);
```

### **6.4 Dropping a Table**

```sql
DROP TABLE Students;
```

---

## **7. Constraints in SQL**

### **7.1 Types and Usage**

| Constraint    | Description                           | Syntax Example                                     |
| ------------- | ------------------------------------- | -------------------------------------------------- |
| `NOT NULL`    | Column must have a value              | `FirstName NVARCHAR(50) NOT NULL`                  |
| `CHECK`       | Column value must satisfy a condition | `CHECK (Age >= 18)`                                |
| `DEFAULT`     | Column has default value              | `DEFAULT GETDATE()`                                |
| `UNIQUE`      | Column value must be unique           | `Email NVARCHAR(100) UNIQUE`                       |
| `PRIMARY KEY` | Uniquely identifies a row             | `StudentID INT PRIMARY KEY`                        |
| `FOREIGN KEY` | Reference another table               | `FOREIGN KEY(ClassID) REFERENCES Classes(ClassID)` |

**Column-level vs Table-level constraints**

- **Column-level:** Applied directly to the column
- **Table-level:** Applied to one or more columns after column definitions

---

### **7.2 Creating Table with Multiple Constraints**

```sql
CREATE TABLE Students (
    StudentID INT CONSTRAINT PK_StudentID PRIMARY KEY,
    FirstName NVARCHAR(50) NOT NULL,
    LastName NVARCHAR(50) NOT NULL,
    Age INT CONSTRAINT CHK_Age CHECK(Age >= 18),
    Email NVARCHAR(100) UNIQUE,
    EnrollmentDate DATE DEFAULT GETDATE()
);
```

**Notes:**

- Naming constraints makes **error messages more meaningful**.
- If no name is given, SQL Server generates one automatically.

---

## **8. Relationships Between Tables**

### **8.1 Types of Relationships**

1. **One-to-One:** Primary key in Table A → foreign key in Table B
2. **One-to-Many:** Primary key in Table A → foreign key in Table B (many rows)
3. **Many-to-Many:** Create a junction table containing **both primary keys** as a composite primary key

### **8.2 Example: One-to-Many**

```sql
CREATE TABLE Classes (
    ClassID INT PRIMARY KEY,
    ClassName NVARCHAR(50)
);

CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    FirstName NVARCHAR(50),
    ClassID INT,
    CONSTRAINT FK_Students_Classes FOREIGN KEY (ClassID)
        REFERENCES Classes(ClassID)
);
```

### **8.3 Example: Many-to-Many**

```sql
CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    FirstName NVARCHAR(50)
);

CREATE TABLE Courses (
    CourseID INT PRIMARY KEY,
    CourseName NVARCHAR(50)
);

CREATE TABLE StudentCourses (
    StudentID INT,
    CourseID INT,
    CONSTRAINT PK_StudentCourses PRIMARY KEY (StudentID, CourseID),
    CONSTRAINT FK_StudentCourses_Students FOREIGN KEY (StudentID)
        REFERENCES Students(StudentID),
    CONSTRAINT FK_StudentCourses_Courses FOREIGN KEY (CourseID)
        REFERENCES Courses(CourseID)
);
```

---

## **9. SQL Server Management Studio (SSMS) Tips**

- **New Query button:** Open a query editor to run SQL scripts
- **Specify database to apply query:**

  - GUI: Select database in dropdown
  - SQL: `USE <DatabaseName>;`

- **Execute specific line(s):** Highlight lines → Press **F5** or **Execute**
- **Edit table in GUI:** Right-click → **Design**
- **Execute SQL option in edit page:** Apply changes directly from GUI

---

## **10. SQL Comments**

```sql
-- Single-line comment
/* Multi-line
   comment */
```

---

## ✅ **Quick Summary: DDL Commands**

| Command           | Usage                                   |
| ----------------- | --------------------------------------- |
| `CREATE DATABASE` | Create a new database                   |
| `ALTER DATABASE`  | Modify files, filegroups, options       |
| `DROP DATABASE`   | Delete a database                       |
| `CREATE TABLE`    | Create a table                          |
| `ALTER TABLE`     | Add, drop, modify columns & constraints |
| `DROP TABLE`      | Delete a table                          |
| `DBCC SHRINKFILE` | Empty a file before removing            |
