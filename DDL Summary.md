# **Ultimate DDL Reference for SQL Server / SSMS**

## **1. What is DDL?**

**DDL (Data Definition Language)** is the subset of SQL used to **define, modify, and remove database structures** such as:
- Databases
- Tables
- Indexes
- Constraints
- File structures

### **What DDL is Used For:**
- Creating and managing database schemas
- Defining table structures with columns and data types
- Establishing relationships between tables
- Setting up constraints to enforce data integrity
- Managing database files and storage

**Common DDL Commands:**
- `CREATE` - Create database, table, file, etc.
- `ALTER` - Modify database, table, or object
- `DROP` - Delete database, table, or object
- `TRUNCATE` - Remove all data from a table

---

## **2. Database File Types in SQL Server**

| File Type | Description / Usage |
|-----------|-------------------|
| **Primary (.mdf)** | Main data file of the database, contains schema and data. Every database has one primary file. |
| **Secondary (.ndf)** | Optional additional data file to spread data across multiple disks. Used for performance and storage management. |
| **Transaction Log (.ldf)** | Stores all transactions for recovery and rollback purposes. Essential for database recovery. |

---

## **3. Creating a Database in SSMS**

### **3.1 Without Coding (GUI Method)**

**Steps:**
1. Right-click **Databases** in Object Explorer
2. Select **New Database…**
3. Enter **Database Name**
4. Configure:
   - **Initial Size**: Starting size of database files
   - **Autogrowth**: How much the file grows when full
   - **Max Size**: Maximum size limit
   - **File Type**: MDF (data) or LDF (log)
   - **Filegroup**: Logical grouping of files
5. Click **OK**

**Initial Size, Autogrowth, MaxSize in SSMS:**
- **Initial Size**: Starting file size (e.g., 10MB)
- **Autogrowth**: Automatic expansion amount (e.g., 5MB or 10%)
- **Max Size**: Maximum allowed file size (e.g., 100MB or UNLIMITED)
- **File Type**: Primary data file (.mdf) or log file (.ldf)
- **File Group**: Logical container for organizing database files

### **3.2 With SQL Coding**

**Syntax:**
```sql
CREATE DATABASE <database_name>
ON PRIMARY (
    NAME = '<logical_name>',
    FILENAME = '<path\filename.mdf>',
    SIZE = <size>,
    FILEGROWTH = <growth_amount>,
    MAXSIZE = <max_size>
)
LOG ON (
    NAME = '<logical_name_log>',
    FILENAME = '<path\filename_log.ldf>',
    SIZE = <size>,
    FILEGROWTH = <growth_amount>,
    MAXSIZE = <max_size>
);
```

**Example:**
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
- `ON PRIMARY` specifies the primary data file
- `LOG ON` specifies the transaction log file
- You can specify multiple data files and log files

---

## **4. Managing Databases / Files**

### **4.1 Viewing Database Files**

**Syntax:**
```sql
SELECT * FROM sys.database_files;
SELECT name, physical_name FROM sys.database_files;
```

**Example:**
```sql
-- All details about database files
SELECT * FROM sys.database_files;

-- Only names and physical paths
SELECT name, physical_name FROM sys.database_files;
```

### **4.2 Adding a New File**

**Syntax:**
```sql
ALTER DATABASE <database_name>
ADD FILE (
    NAME = '<logical_name>',
    FILENAME = '<path\filename.ndf>',
    SIZE = <size>,
    FILEGROWTH = <growth_amount>,
    MAXSIZE = <max_size>
);
```

**Example:**
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

**Syntax:**
```sql
ALTER DATABASE <database_name>
MODIFY FILE (
    NAME = '<logical_name>',
    SIZE = <new_size>,
    MAXSIZE = <new_maxsize>,
    FILEGROWTH = <new_growth>
);
```

**Example:**
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

**Precondition**: If the file contains data, first empty it using `DBCC SHRINKFILE`

**Syntax:**
```sql
DBCC SHRINKFILE('<logical_name>', EMPTYFILE);
ALTER DATABASE <database_name>
REMOVE FILE <logical_name>;
```

**Example:**
```sql
DBCC SHRINKFILE('SchoolDB_Data2', EMPTYFILE);
ALTER DATABASE SchoolDB
REMOVE FILE SchoolDB_Data2;
```

### **4.5 Deleting a Database**

**Precondition**: You **cannot be using the database** you want to delete.

**Syntax:**
```sql
USE master;
DROP DATABASE <database_name>;
```

**Example:**
```sql
USE master;
DROP DATABASE SchoolDB;
```

---

## **5. Creating Tables**

### **5.1 Basic Table Syntax**

**Syntax:**
```sql
CREATE TABLE <table_name> (
    <column_name> <data_type>,
    <column_name> <data_type>,
    <column_name> <data_type>
);
```

**Example:**
```sql
CREATE TABLE Students (
    StudentID INT,
    FirstName NVARCHAR(50),
    LastName NVARCHAR(50),
    BirthDate DATE,
    Gender CHAR(1)
);
```

### **5.2 SQL Data Types Reference**

| Data Type | Storage Size | Range / Notes | Usage Example | When/Why to Use |
|-----------|--------------|---------------|---------------|-----------------|
| `INT` | 4 bytes | -2,147,483,648 to 2,147,483,647 | `Age INT` | Most common integer type for IDs, counts |
| `BIGINT` | 8 bytes | -9.22e18 to 9.22e18 | `Population BIGINT` | Very large numbers, big primary keys |
| `SMALLINT` | 2 bytes | -32,768 to 32,767 | `Score SMALLINT` | Smaller integers to save space |
| `TINYINT` | 1 byte | 0 to 255 | `Rating TINYINT` | Small ranges like ratings, flags |
| `DECIMAL(p,s)` | Varies | Fixed precision, `p`=total digits, `s`=decimal places | `Price DECIMAL(10,2)` | Exact decimal values like money |
| `FLOAT` | 4/8 bytes | Approximate floating-point | `Temperature FLOAT` | Scientific calculations, approximate values |
| `REAL` | 4 bytes | Approximate floating-point | `Measurement REAL` | Less precision than FLOAT, saves space |
| `CHAR(n)` | n bytes | Fixed-length string | `Gender CHAR(1)` | Fixed-length strings like codes |
| `VARCHAR(n)` | Variable | Variable-length string | `Name VARCHAR(50)` | Most common for variable text |
| `NVARCHAR(n)` | Variable | Unicode string | `Address NVARCHAR(100)` | International text, multiple languages |
| `TEXT` | Up to 2GB | Large text data | `Description TEXT` | Deprecated - use VARCHAR(MAX) instead |
| `NTEXT` | Up to 2GB | Unicode large text | `Notes NTEXT` | Deprecated - use NVARCHAR(MAX) instead |
| `VARCHAR(MAX)` | Up to 2GB | Large text data | `Content VARCHAR(MAX)` | Replace TEXT, stores large documents |
| `NVARCHAR(MAX)` | Up to 2GB | Unicode large text | `TextData NVARCHAR(MAX)` | Replace NTEXT, large international text |
| `DATE` | 3 bytes | 0001-01-01 to 9999-12-31 | `BirthDate DATE` | Date only values |
| `TIME` | 5 bytes | 00:00:00.0000000 to 23:59:59.9999999 | `StartTime TIME` | Time only values |
| `DATETIME` | 8 bytes | 1753-01-01 to 9999-12-31 | `CreatedAt DATETIME` | Legacy date+time (less precision) |
| `DATETIME2` | 6-8 bytes | 0001-01-01 to 9999-12-31 | `UpdatedAt DATETIME2(3)` | Modern date+time (more precise) |
| `SMALLDATETIME` | 4 bytes | 1900-01-01 to 2079-06-06 | `MeetingTime SMALLDATETIME` | Less precise, smaller storage |
| `BIT` | 1 bit | 0, 1, or NULL | `IsActive BIT` | Boolean flags (true/false) |
| `MONEY` | 8 bytes | -922 trillion to +922 trillion | `Salary MONEY` | Currency values (high precision) |
| `SMALLMONEY` | 4 bytes | -214,748.3648 to 214,748.3647 | `Bonus SMALLMONEY` | Smaller currency range |
| `BINARY(n)` | n bytes | Fixed-length binary | `ImageHash BINARY(16)` | Hashes, fixed binary data |
| `VARBINARY(n)` | Variable | Variable-length binary | `Photo VARBINARY(MAX)` | Images, files, variable binary |
| `XML` | Variable | XML data | `Settings XML` | Hierarchical data, configuration |
| `UNIQUEIDENTIFIER` | 16 bytes | Globally unique ID | `UserID UNIQUEIDENTIFIER` | GUIDs, distributed systems |
| `HIERARCHYID` | Variable | Hierarchical data | `OrgNode HIERARCHYID` | Tree structures, organizational charts |
| `GEOGRAPHY` | Variable | Spatial data (earth) | `Location GEOGRAPHY` | GPS coordinates, maps |
| `GEOMETRY` | Variable | Spatial data (plane) | `Shape GEOMETRY` | 2D geometric shapes |

---

## **6. Altering Tables**

### **6.1 Adding a Column**

**Syntax:**
```sql
ALTER TABLE <table_name>
ADD <column_name> <data_type>;
```

**Example:**
```sql
ALTER TABLE Students
ADD Email NVARCHAR(100);
```

### **6.2 Deleting a Column**

**Syntax:**
```sql
ALTER TABLE <table_name>
DROP COLUMN <column_name>;
```

**Example:**
```sql
ALTER TABLE Students
DROP COLUMN Email;
```

### **6.3 Modifying Column Type**

**Syntax:**
```sql
ALTER TABLE <table_name>
ALTER COLUMN <column_name> <new_data_type>;
```

**Example:**
```sql
ALTER TABLE Students
ALTER COLUMN FirstName NVARCHAR(100);
```

### **6.4 Dropping a Table**

**Syntax:**
```sql
DROP TABLE <table_name>;
```

**Example:**
```sql
DROP TABLE Students;
```

---

## **7. Constraints in SQL**

### **7.1 Types of Constraints**

| Constraint | Description | Syntax Example |
|------------|-------------|----------------|
| `NOT NULL` | Column must have a value | `FirstName NVARCHAR(50) NOT NULL` |
| `CHECK` | Column value must satisfy condition | `CHECK (Age >= 18)` |
| `DEFAULT` | Column has default value if not specified | `DEFAULT GETDATE()` |
| `UNIQUE` | Column value must be unique across table | `Email NVARCHAR(100) UNIQUE` |
| `PRIMARY KEY` | Uniquely identifies a row (NOT NULL + UNIQUE) | `StudentID INT PRIMARY KEY` |
| `FOREIGN KEY` | Reference to primary key in another table | `FOREIGN KEY(ClassID) REFERENCES Classes(ClassID)` |

### **7.2 Creating Table with Constraints**

**Syntax with Column-Level Constraints:**
```sql
CREATE TABLE <table_name> (
    <column_name> <data_type> CONSTRAINT <constraint_name> <constraint_type> <constraint_details>,
    <column_name> <data_type> <constraint_type> <constraint_details>
);
```

**Example:**
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

**Why Use Constraint Names:**
- Makes error messages more meaningful
- Easier to drop/modify constraints later
- Better documentation and maintenance
- SQL Server auto-generates names if not provided (e.g., `DF__Students__Enrol__123456`)

### **7.3 CHECK Constraint Examples**

**Syntax:**
```sql
CREATE TABLE <table_name> (
    <column_name> <data_type> CONSTRAINT <name> CHECK(condition),
    <column_name> <data_type> CHECK(condition),
    CONSTRAINT <name> CHECK(<column> condition)
);
```

**Example:**
```sql
CREATE TABLE Products (
    ProductID INT PRIMARY KEY,
    ProductName NVARCHAR(100),
    Price DECIMAL(10,2) CONSTRAINT CHK_Price CHECK(Price > 0),
    Quantity INT CHECK(Quantity >= 0),
    Category NVARCHAR(50),
    CONSTRAINT CHK_Category CHECK(Category IN ('Electronics', 'Clothing', 'Books'))
);
```

### **7.4 DEFAULT Constraint Examples**

**Syntax:**
```sql
CREATE TABLE <table_name> (
    <column_name> <data_type> CONSTRAINT <name> DEFAULT(value),
    <column_name> <data_type> DEFAULT(value)
);
```

**Example:**
```sql
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    OrderDate DATETIME CONSTRAINT DF_OrderDate DEFAULT GETDATE(),
    Status NVARCHAR(20) DEFAULT 'Pending',
    TotalAmount DECIMAL(10,2)
);
```

### **7.5 UNIQUE Constraint Examples**

**Syntax:**
```sql
CREATE TABLE <table_name> (
    <column_name> <data_type> CONSTRAINT <name> UNIQUE,
    <column_name> <data_type> UNIQUE,
    CONSTRAINT <name> UNIQUE(column_name)
);
```

**Example:**
```sql
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    Email NVARCHAR(100) CONSTRAINT UQ_Email UNIQUE,
    Phone NVARCHAR(20) UNIQUE,
    SSN NVARCHAR(11),
    CONSTRAINT UQ_SSN UNIQUE(SSN)
);
```

### **7.6 PRIMARY KEY Constraint**

**What is a Primary Key:**
- Uniquely identifies each row in a table
- Cannot contain NULL values
- Only one primary key per table (can be composite)
- Automatically creates a clustered index

**Column-Level vs Table-Level Constraints:**
- **Column-Level**: Applied directly to a single column
- **Table-Level**: Applied after column definitions, can involve multiple columns

**Syntax:**
```sql
CREATE TABLE <table_name> (
    <column_name> <data_type> CONSTRAINT <name> PRIMARY KEY,
    <column_name> <data_type> PRIMARY KEY,
    CONSTRAINT <name> PRIMARY KEY(column_name)
);
```

**Example:**
```sql
CREATE TABLE Customers (
    CustomerID INT CONSTRAINT PK_CustomerID PRIMARY KEY,
    FirstName NVARCHAR(50),
    LastName NVARCHAR(50),
    Email NVARCHAR(100)
);
```

**What Happens Without Constraint Names:**
SQL Server generates automatic names like:
- `PK__Students__3214EC27456E1A6F`
- `DF__Students__Enrol__45F365D3`
- `UQ__Students__A9D10534427E3C1C`

---

## **8. Relationships Between Tables**

### **8.1 Identifying Relationships**

**Steps to Identify Relationships:**
1. Check if tables have connections
2. Determine relationship type:
   - One-to-One (1:1)
   - One-to-Many (1:M)
   - Many-to-Many (M:N)
3. Analyze: "How many tuples does Table A relate to in Table B per tuple?"

**Examples:**
- Does each student have a contact? How many contacts per student?
- Does each class have students? How many students per class?

### **8.2 Relationship Types Implementation**

#### **One-to-One Relationship**
- Take primary key of first table → foreign key in second table
- **Example**: Student ↔ StudentContact (each student has one contact)

**Syntax:**
```sql
CREATE TABLE Students (
    StudentID INT PRIMARY KEY,
    Name NVARCHAR(100)
);

CREATE TABLE StudentContacts (
    ContactID INT PRIMARY KEY,
    StudentID INT UNIQUE, -- Ensures one-to-one
    Phone NVARCHAR(20),
    Email NVARCHAR(100),
    FOREIGN KEY (StudentID) REFERENCES Students(StudentID)
);
```

#### **One-to-Many Relationship**
- Take primary key of "one" table → foreign key in "many" table
- **Example**: Class ↔ Students (one class has many students)

**Syntax:**
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

#### **Many-to-Many Relationship**
- Create junction/association table
- Take both primary keys as foreign keys
- Both foreign keys together form composite primary key
- **Example**: Students ↔ Courses (students take many courses, courses have many students)

**Syntax:**
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
    EnrollmentDate DATE DEFAULT GETDATE(),
    CONSTRAINT PK_StudentCourses PRIMARY KEY (StudentID, CourseID),
    CONSTRAINT FK_StudentCourses_Students FOREIGN KEY (StudentID)
        REFERENCES Students(StudentID),
    CONSTRAINT FK_StudentCourses_Courses FOREIGN KEY (CourseID)
        REFERENCES Courses(CourseID)
);
```

### **8.3 Composite Primary Key**

**Syntax:**
```sql
CREATE TABLE <table_name> (
    <column1> <type>,
    <column2> <type>,
    PRIMARY KEY(column1, column2)
);
```

**Example:**
```sql
CREATE TABLE OrderDetails (
    OrderID INT,
    ProductID INT,
    Quantity INT,
    Price DECIMAL(10,2),
    PRIMARY KEY(OrderID, ProductID)
);
```

### **8.4 FOREIGN KEY Constraint**

**Syntax:**
```sql
CREATE TABLE <table_name> (
    <column_name> <type> CONSTRAINT <name> REFERENCES <ref_table>(<ref_column>),
    <column_name> <type> REFERENCES <ref_table>(<ref_column>),
    CONSTRAINT <name> FOREIGN KEY (column) REFERENCES <ref_table>(<ref_column>)
);
```

**Example:**
```sql
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT,
    OrderDate DATETIME,
    CONSTRAINT FK_Orders_Customers FOREIGN KEY (CustomerID)
        REFERENCES Customers(CustomerID)
);
```

---

## **9. Database Schema**

**What is a Database Schema:**
- Logical container for database objects (tables, views, procedures)
- Provides namespace and security boundary
- Default schema: `dbo` (database owner)

**Using Reserved Keywords as Names:**
```sql
-- Use square brackets for reserved keywords
CREATE TABLE [Address] (
    AddressID INT PRIMARY KEY,
    [Street] NVARCHAR(100),  -- Street is not reserved but shows syntax
    [City] NVARCHAR(50)
);
```

---

## **10. SSMS (SQL Server Management Studio) Tips**

### **10.1 New Query Button**
- Opens a query editor for writing SQL scripts
- Located on the toolbar or via File → New → Query with Current Connection

### **10.2 Specifying Database Context**

**GUI Method:**
- Use dropdown in toolbar to select database
- Shows currently connected database

**SQL Method:**
```sql
USE <database_name>;
```

**Example:**
```sql
USE SchoolDB;
-- All subsequent commands apply to SchoolDB
```

### **10.3 Executing Specific Lines in SSMS**
- Highlight the lines you want to execute
- Press **F5** or click **Execute** button
- Or right-click → **Execute**
- Use **Ctrl+E** as shortcut

### **10.4 Edit Table in GUI**
- Right-click table → **Design**
- Modify columns, data types, constraints visually
- **Execute SQL Option**: Apply changes directly from GUI

### **10.5 Schema Execution Order**
SQL Server processes:
1. **Schema Validation**: Checks syntax and object existence
2. **Constraint Checking**: Validates constraints can be created
3. **Object Creation**: Creates the objects
4. **Dependencies**: Handles dependent objects

---

## **11. Constraint Modification**

### **11.1 Adding Constraints to Existing Table**

**Syntax:**
```sql
ALTER TABLE <table_name>
ADD CONSTRAINT <constraint_name> <constraint_type> <constraint_details>;
```

**Examples:**

**ADD CHECK Constraint:**
```sql
ALTER TABLE Students
ADD CONSTRAINT CHK_Age CHECK(Age >= 18);
```

**ADD DEFAULT Constraint:**
```sql
ALTER TABLE Students
ADD CONSTRAINT DF_EnrollmentDate DEFAULT GETDATE() FOR EnrollmentDate;
```

**ADD UNIQUE Constraint:**
```sql
ALTER TABLE Students
ADD CONSTRAINT UQ_Email UNIQUE(Email);
```

**ADD PRIMARY KEY Constraint:**
```sql
ALTER TABLE Students
ADD CONSTRAINT PK_StudentID PRIMARY KEY(StudentID);
```

**ADD FOREIGN KEY Constraint:**
```sql
ALTER TABLE Students
ADD CONSTRAINT FK_Students_Classes 
FOREIGN KEY (ClassID) REFERENCES Classes(ClassID);
```

### **11.2 Dropping Constraints**

**Syntax:**
```sql
ALTER TABLE <table_name>
DROP CONSTRAINT <constraint_name>;
```

**Example:**
```sql
ALTER TABLE Students
DROP CONSTRAINT CHK_Age;
```

---

## **12. SQL Comments**

**Single-Line Comments:**
```sql
-- This is a single-line comment
SELECT * FROM Students; -- Comment after code
```

**Multi-Line Comments:**
```sql
/* This is a 
   multi-line 
   comment */
SELECT * FROM Students;
```

---

## **13. Key Concepts Summary**

### **Logical Name vs Actual Name:**
- **Logical Name**: Name used in SQL queries (e.g., `SchoolDB_data`)
- **Actual Name**: Physical filename on disk (e.g., `C:\SQLData\SchoolDB.mdf`)

### **SQL Case Insensitivity:**
- SQL keywords are case-insensitive: `SELECT`, `select`, `Select` are all valid
- Object names depend on server collation settings
- Best practice: Use consistent casing for readability

### **Preconditions for Database Operations:**
1. **Deleting Database**: Must not be in use, switch to another database first
2. **Removing Files**: File must be empty (use `DBCC SHRINKFILE` first)
3. **Dropping Tables**: No foreign key constraints referencing the table

---

## **✅ Quick Reference: DDL Commands**

| Command | Syntax | Example |
|---------|--------|---------|
| **CREATE DATABASE** | `CREATE DATABASE <name> ON PRIMARY (...) LOG ON (...)` | `CREATE DATABASE SchoolDB...` |
| **ALTER DATABASE** | `ALTER DATABASE <name> ADD/MODIFY/REMOVE FILE` | `ALTER DATABASE SchoolDB ADD FILE...` |
| **DROP DATABASE** | `DROP DATABASE <name>` | `DROP DATABASE SchoolDB` |
| **CREATE TABLE** | `CREATE TABLE <name> (col type, col type)` | `CREATE TABLE Students (...)` |
| **ALTER TABLE** | `ALTER TABLE <name> ADD/DROP/ALTER COLUMN` | `ALTER TABLE Students ADD Email...` |
| **DROP TABLE** | `DROP TABLE <name>` | `DROP TABLE Students` |
| **ADD CONSTRAINT** | `ALTER TABLE <name> ADD CONSTRAINT` | `ALTER TABLE Students ADD PK...` |
| **DBCC SHRINKFILE** | `DBCC SHRINKFILE(<name>, EMPTYFILE)` | `DBCC SHRINKFILE('File2', EMPTYFILE)` |

---

## **📋 Best Practices**

1. **Always name your constraints** for easier management
2. **Use appropriate data types** to optimize storage
3. **Define primary keys** for all tables
4. **Use foreign keys** to enforce referential integrity
5. **Test constraints** before applying to production
6. **Back up databases** before major DDL operations
7. **Use transactions** for multiple related DDL operations
8. **Document your schema** with comments and documentation

