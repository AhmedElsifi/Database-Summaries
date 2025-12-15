# **Complete DQL (Data Query Language) Reference for SQL Server**

## **1. What is DQL?**

**DQL (Data Query Language)** is the subset of SQL used to **retrieve, query, and fetch data** from databases. While sometimes considered part of DML, DQL specifically focuses on reading data using the `SELECT` statement and its various clauses.

**Primary DQL Command:**

- `SELECT` - Retrieve data from one or more tables

**DQL vs DML vs DDL:**

- **DDL**: Define/modify structure (CREATE, ALTER, DROP)
- **DML**: Manipulate data (INSERT, UPDATE, DELETE)
- **DQL**: Query/retrieve data (SELECT)

---

## **2. Basic SELECT Statement**

### **2.1 Selecting from One Table**

**Syntax:**

```sql
SELECT <column1>, <column2>, <column3>, ...
FROM <table_name>;

-- Select all columns
SELECT * FROM <table_name>;
```

**Example:**

```sql
-- Select specific columns
SELECT FirstName, LastName, Email
FROM Students;

-- Select all columns
SELECT * FROM Students;
```

**Database Context:**
Always specify which database to use before executing queries:

**Method 1: Using USE statement**

```sql
USE SchoolDB;
SELECT * FROM Students;
```

**Method 2: Using GUI dropdown** in SSMS to select database

---

## **3. Joining Multiple Tables**

### **3.1 JOIN Fundamentals**

**Rules for JOIN:**

1. **Only for connected tables**: Tables must have a relationship
2. **Common column required**: Must have at least one column in common

**JOIN Types:**

- **INNER JOIN**: Returns matching records from both tables
- **LEFT JOIN**: Returns all from left table + matching from right
- **RIGHT JOIN**: Returns all from right table + matching from left
- **FULL OUTER JOIN**: Returns all from both tables

### **3.2 INNER JOIN**

**What it does**: Returns only rows with matching values in both tables

**Syntax:**

```sql
SELECT <columns>
FROM <table1>
INNER JOIN <table2>
ON <table1.column> = <table2.column>;

-- INNER JOIN is the same as JOIN
SELECT <columns>
FROM <table1>
JOIN <table2>
ON <table1.column> = <table2.column>;
```

**Example:**

```sql
-- Get students with their class information
SELECT Students.FirstName, Students.LastName, Classes.ClassName
FROM Students
INNER JOIN Classes ON Students.ClassID = Classes.ClassID;
```

**Visual Example:**

```
Students Table           Classes Table
ID | Name | ClassID     ClassID | ClassName
1  | John | 101         101     | Math
2  | Jane | 102         102     | Science
3  | Bob  | 103         103     | History

INNER JOIN Result:
Name | ClassName
John | Math
Jane | Science
Bob  | History
```

### **3.3 LEFT JOIN (LEFT OUTER JOIN)**

**What it does**: Returns all rows from left table + matching rows from right table (NULL if no match)

**Syntax:**

```sql
SELECT <columns>
FROM <left_table>
LEFT JOIN <right_table>
ON <left_table.column> = <right_table.column>;
```

**Example:**

```sql
-- Get all students, even if they don't have a class
SELECT Students.FirstName, Students.LastName, Classes.ClassName
FROM Students
LEFT JOIN Classes ON Students.ClassID = Classes.ClassID;
```

**Visual Example:**

```
Students Table           Classes Table
ID | Name | ClassID     ClassID | ClassName
1  | John | 101         101     | Math
2  | Jane | 102         102     | Science
3  | Bob  | NULL        103     | History

LEFT JOIN Result:
Name | ClassName
John | Math
Jane | Science
Bob  | NULL
```

### **3.4 RIGHT JOIN (RIGHT OUTER JOIN)**

**What it does**: Returns all rows from right table + matching rows from left table (NULL if no match)

**Syntax:**

```sql
SELECT <columns>
FROM <left_table>
RIGHT JOIN <right_table>
ON <left_table.column> = <right_table.column>;
```

**Example:**

```sql
-- Get all classes, even if no students are enrolled
SELECT Students.FirstName, Students.LastName, Classes.ClassName
FROM Students
RIGHT JOIN Classes ON Students.ClassID = Classes.ClassID;
```

**Visual Example:**

```
Students Table           Classes Table
ID | Name | ClassID     ClassID | ClassName
1  | John | 101         101     | Math
2  | Jane | 102         102     | Science
3  | NULL | NULL        103     | History
                        104     | Art

RIGHT JOIN Result:
Name  | ClassName
John  | Math
Jane  | Science
NULL  | History
NULL  | Art
```

### **3.5 FULL OUTER JOIN**

**What it does**: Returns all rows from both tables (NULL where no match)

**Syntax:**

```sql
SELECT <columns>
FROM <table1>
FULL OUTER JOIN <table2>
ON <table1.column> = <table2.column>;
```

**Example:**

```sql
-- Get all students and all classes
SELECT Students.FirstName, Students.LastName, Classes.ClassName
FROM Students
FULL OUTER JOIN Classes ON Students.ClassID = Classes.ClassID;
```

**Visual Example:**

```
Students Table           Classes Table
ID | Name | ClassID     ClassID | ClassName
1  | John | 101         101     | Math
2  | Jane | 102         102     | Science
3  | Bob  | NULL        103     | History
                        104     | Art

FULL OUTER JOIN Result:
Name  | ClassName
John  | Math
Jane  | Science
Bob   | NULL
NULL  | History
NULL  | Art
```

### **3.6 JOIN Comparison Summary**

| JOIN Type           | Returns                   | Common Use                    |
| ------------------- | ------------------------- | ----------------------------- |
| **INNER JOIN**      | Only matching rows        | Get related data              |
| **LEFT JOIN**       | All left + matching right | Include all main records      |
| **RIGHT JOIN**      | All right + matching left | Include all reference records |
| **FULL OUTER JOIN** | All from both tables      | Complete merge                |

---

## **4. SELECT Clauses and Keywords**

### **4.1 WHERE Clause**

**Purpose**: Filters rows based on conditions

**Syntax:**

```sql
SELECT <columns>
FROM <table>
WHERE <condition>;
```

**Examples:**

```sql
-- Simple condition
SELECT * FROM Students WHERE Grade > 80;

-- Multiple conditions
SELECT * FROM Students WHERE Age >= 18 AND Grade > 70;

-- Using with JOIN
SELECT FirstName, LastName, StudentID
FROM Students
INNER JOIN Classes ON Students.ClassID = Classes.ClassID
WHERE Department = 'IS';

SELECT FirstName, LastName
FROM Students
INNER JOIN Classes ON Students.ClassID = Classes.ClassID
WHERE Number_of_Students > 10;
```

### **4.2 ORDER BY Clause**

**Purpose**: Sorts result set

**Syntax:**

```sql
SELECT <columns>
FROM <table>
[WHERE <condition>]
ORDER BY <column> [ASC|DESC];
```

**Notes:**

- **ASC** = Ascending (A-Z, 1-10) - **DEFAULT**
- **DESC** = Descending (Z-A, 10-1)
- Can order by numbers, strings, dates
- Can order by multiple columns
- **WHERE must come before ORDER BY**

**Examples:**

```sql
-- Single column
SELECT * FROM Students ORDER BY LastName ASC;

-- Multiple columns
SELECT * FROM Students ORDER BY Grade DESC, LastName ASC;

-- With WHERE
SELECT * FROM Students WHERE Age > 17 ORDER BY Grade DESC;
```

### **4.3 AS (Alias)**

**Purpose**: Temporarily rename columns/tables (query-scoped only)

**Syntax:**

```sql
-- Column alias
SELECT <column> AS <alias> FROM <table>;

-- Table alias
SELECT <columns> FROM <table> AS <alias>;

-- Combined example
SELECT Student.FirstName AS FN, Student.LastName AS LN
FROM Student AS Std
INNER JOIN Student_Contact AS Std_Ct ON Std.ID = Std_Ct.Student_ID
WHERE Age > 17
ORDER BY Grade DESC;
```

**Important**: Aliases are **query-scoped** - cannot be used in other queries

### **4.4 IN Operator**

**Purpose**: Filter with multiple specific values

**Syntax:**

```sql
SELECT <columns>
FROM <table>
WHERE <column> IN (value1, value2, value3, ...);
```

**Example:**

```sql
SELECT * FROM Students
INNER JOIN Classes ON Students.ClassID = Classes.ClassID
WHERE Department IN ('IS', 'CS', 'IT', 'SW');
```

**Equivalent to multiple OR conditions:**

```sql
WHERE Department = 'IS' OR Department = 'CS' OR Department = 'IT' OR Department = 'SW';
```

### **4.5 BETWEEN Operator**

**Purpose**: Filter within a range (inclusive)

**Syntax:**

```sql
SELECT <columns>
FROM <table>
WHERE <column> BETWEEN <value1> AND <value2>;
```

**Examples:**

```sql
-- Numeric range
SELECT * FROM Classes WHERE number_of_students BETWEEN 10 AND 20;

-- Date range
SELECT * FROM Orders WHERE OrderDate BETWEEN '2024-01-01' AND '2024-01-31';

-- Character range (alphabetical)
SELECT * FROM Students WHERE LastName BETWEEN 'A' AND 'M';
```

### **4.6 LIKE Operator**

**Purpose**: Pattern matching with wildcards

**Wildcards:**

- `%` = Any sequence of characters (0 or more)
- `_` = Single character

**Syntax:**

```sql
SELECT <columns>
FROM <table>
WHERE <column> LIKE '<pattern>';
```

**Pattern Examples:**

| Pattern  | Matches                           | Example                     |
| -------- | --------------------------------- | --------------------------- |
| `'a%'`   | Starts with "a"                   | "apple", "ant", "a"         |
| `'%a'`   | Ends with "a"                     | "banana", "pizza", "a"      |
| `'%or%'` | Contains "or"                     | "orange", "fork", "storage" |
| `'_r%'`  | Second letter is "r"              | "art", "cry", "brave"       |
| `'a_%'`  | Starts with "a", at least 2 chars | "at", "apple", "ant"        |
| `'a__%'` | Starts with "a", at least 3 chars | "ant", "apple", "april"     |
| `'a%o'`  | Starts with "a", ends with "o"    | "apollo", "ado", "audio"    |

**Examples:**

```sql
-- Contains "gmail" anywhere
SELECT * FROM Students
INNER JOIN Student_Contact ON Students.ID = Student_Contact.Student_ID
WHERE Email LIKE '%gmail%';

-- Starts with "jo"
SELECT * FROM Students WHERE FirstName LIKE 'jo%';

-- Exactly 4 characters ending with "son"
SELECT * FROM Students WHERE LastName LIKE '__son';
```

### **4.7 NOT Operator**

**Purpose**: Negates/inverts a condition

**Syntax:**

```sql
SELECT <columns>
FROM <table>
WHERE NOT <condition>;
```

**Examples:**

```sql
-- NOT LIKE
SELECT * FROM Students
INNER JOIN Student_Contact ON Students.ID = Student_Contact.Student_ID
WHERE Email NOT LIKE '%gmail%';

-- NOT IN
SELECT * FROM Students WHERE Department NOT IN ('IS', 'CS');

-- NOT BETWEEN
SELECT * FROM Classes WHERE number_of_students NOT BETWEEN 10 AND 20;

-- NOT with complex condition
SELECT * FROM Students WHERE NOT (Age > 18 AND Grade > 70);
```

---

## **5. Subqueries**

### **5.1 What are Subqueries?**

- **Nested queries** inside main query
- Used instead of JOINs or with IN, LIKE, BETWEEN, EXISTS
- Inner query executes first, results used in outer query

### **5.2 Subquery with IN**

**Syntax:**

```sql
SELECT <columns>
FROM <table>
WHERE <column> IN (SELECT <column> FROM <table> WHERE <condition>);
```

**Examples:**

**Using JOIN:**

```sql
SELECT FirstName, LastName
FROM Students
INNER JOIN Classes ON Students.ClassID = Classes.ClassID
WHERE number_of_students > 12;
```

**Equivalent with Subquery:**

```sql
SELECT FirstName, LastName
FROM Students
WHERE ClassID IN (
    SELECT ClassID
    FROM Classes
    WHERE number_of_students > 12
);
```

### **5.3 Subquery Types**

**Single-Row Subquery**: Returns one row

```sql
SELECT * FROM Employees
WHERE Salary > (SELECT AVG(Salary) FROM Employees);
```

**Multi-Row Subquery**: Returns multiple rows (used with IN, ANY, ALL)

```sql
SELECT * FROM Products
WHERE CategoryID IN (
    SELECT CategoryID
    FROM Categories
    WHERE IsActive = 1
);
```

**Correlated Subquery**: References outer query

```sql
SELECT e1.* FROM Employees e1
WHERE Salary > (
    SELECT AVG(Salary)
    FROM Employees e2
    WHERE e1.DepartmentID = e2.DepartmentID
);
```

### **5.4 EXISTS Operator with Subqueries**

**Purpose**: Check if subquery returns any rows

**Syntax:**

```sql
SELECT <columns>
FROM <table>
WHERE EXISTS (SELECT 1 FROM <table> WHERE <condition>);
```

**Example:**

```sql
-- Students who have enrolled in at least one course
SELECT * FROM Students s
WHERE EXISTS (
    SELECT 1
    FROM Enrollments e
    WHERE e.StudentID = s.StudentID
);
```

---

## **6. Aggregation Functions**

### **6.1 What are Aggregation Functions?**

- Functions that **return single values** from multiple rows
- Always return **one row** of result
- Used with GROUP BY for grouped calculations

### **6.2 COUNT()**

**Purpose**: Count number of rows or non-NULL values

**Syntax:**

```sql
-- Count all rows
SELECT COUNT(*) FROM <table>;

-- Count non-NULL values in column
SELECT COUNT(<column>) FROM <table>;
```

**Examples:**

```sql
SELECT COUNT(*) AS Number_Of_Students FROM Students;

SELECT COUNT(DISTINCT Department) AS Unique_Departments FROM Classes;

SELECT COUNT(Email) AS Students_With_Email FROM Students;
```

### **6.3 SUM()**

**Purpose**: Calculate total sum of numeric column

**Syntax:**

```sql
SELECT SUM(<numeric_column>) FROM <table> [WHERE <condition>];
```

**Examples:**

```sql
SELECT SUM(Number_of_Students) AS Total_Students FROM Classes;

SELECT SUM(Salary) AS Total_Payroll FROM Employees WHERE Department = 'IT';

SELECT SUM(Quantity * Price) AS Total_Sales FROM OrderDetails;
```

### **6.4 AVG()**

**Purpose**: Calculate average (mean) of numeric column

**Syntax:**

```sql
SELECT AVG(<numeric_column>) FROM <table> [WHERE <condition>];
```

**Examples:**

```sql
SELECT AVG(Number_of_Students) AS Avg_Class_Size FROM Classes;

SELECT AVG(Grade) AS Average_Grade FROM Students WHERE Age > 18;

SELECT AVG(DATEDIFF(day, OrderDate, ShippedDate)) AS Avg_Shipping_Days FROM Orders;
```

### **6.5 MIN()**

**Purpose**: Find smallest value in column

**Syntax:**

```sql
SELECT MIN(<column>) FROM <table> [WHERE <condition>];
```

**Examples:**

```sql
SELECT MIN(Number_of_Students) AS Smallest_Class FROM Classes;

SELECT MIN(BirthDate) AS Oldest_Student FROM Students;

SELECT MIN(Salary) AS Lowest_Salary FROM Employees;
```

### **6.6 MAX()**

**Purpose**: Find largest value in column

**Syntax:**

```sql
SELECT MAX(<column>) FROM <table> [WHERE <condition>];
```

**Examples:**

```sql
SELECT MAX(Number_of_Students) AS Largest_Class FROM Classes;

SELECT MAX(Grade) AS Highest_Grade FROM Students;

SELECT MAX(OrderDate) AS Most_Recent_Order FROM Orders;
```

### **6.7 Aggregation Function Summary**

| Function  | Returns               | Works With          | NULL Handling                     |
| --------- | --------------------- | ------------------- | --------------------------------- |
| `COUNT()` | Number of rows/values | Any data type       | Excludes NULLs (except COUNT(\*)) |
| `SUM()`   | Total sum             | Numeric only        | Treats NULL as 0                  |
| `AVG()`   | Average               | Numeric only        | Excludes NULLs from calculation   |
| `MIN()`   | Smallest value        | Any comparable type | Excludes NULLs                    |
| `MAX()`   | Largest value         | Any comparable type | Excludes NULLs                    |

---

## **7. GROUP BY Clause**

### **7.1 What is GROUP BY?**

- Groups rows with same values into summary rows
- Used with aggregation functions
- **Keyword trigger**: When question says "for each", "per", "by"

**Syntax:**

```sql
SELECT <aggregation_function>, <grouping_column>
FROM <table>
[WHERE <condition>]
GROUP BY <grouping_column>
[HAVING <condition>];
```

### **7.2 Basic GROUP BY Examples**

**Example 1: Simple grouping**

```sql
-- Count students per department
SELECT COUNT(*) AS Student_Count, Department
FROM Students
INNER JOIN Classes ON Students.ClassID = Classes.ClassID
GROUP BY Department;
```

**Example 2: Multiple aggregations**

```sql
-- Statistics per department
SELECT
    Department,
    COUNT(*) AS Student_Count,
    AVG(Grade) AS Average_Grade,
    MIN(Grade) AS Minimum_Grade,
    MAX(Grade) AS Maximum_Grade
FROM Students
INNER JOIN Classes ON Students.ClassID = Classes.ClassID
GROUP BY Department;
```

**Example 3: Group by multiple columns**

```sql
-- Students per department and class year
SELECT
    Department,
    YEAR(EnrollmentDate) AS Enrollment_Year,
    COUNT(*) AS Student_Count
FROM Students
INNER JOIN Classes ON Students.ClassID = Classes.ClassID
GROUP BY Department, YEAR(EnrollmentDate);
```

### **7.3 HAVING Clause**

**Purpose**: Filter grouped results (WHERE filters before grouping, HAVING filters after)

**Syntax:**

```sql
SELECT <aggregation_function>, <grouping_column>
FROM <table>
GROUP BY <grouping_column>
HAVING <condition_on_aggregation>;
```

**Examples:**

```sql
-- Departments with more than 50 students
SELECT Department, COUNT(*) AS Student_Count
FROM Students
INNER JOIN Classes ON Students.ClassID = Classes.ClassID
GROUP BY Department
HAVING COUNT(*) > 50;

-- Classes with average grade > 70
SELECT ClassName, AVG(Grade) AS Average_Grade
FROM Students
INNER JOIN Classes ON Students.ClassID = Classes.ClassID
GROUP BY ClassName
HAVING AVG(Grade) > 70;
```

### **7.4 WHERE vs HAVING Comparison**

| Aspect                    | WHERE                                 | HAVING                         |
| ------------------------- | ------------------------------------- | ------------------------------ |
| **Applied When**          | Before grouping                       | After grouping                 |
| **Filters**               | Individual rows                       | Grouped results                |
| **Can Use Aggregations?** | No (except in subqueries)             | **Yes**                        |
| **Performance**           | Faster (reduces rows before grouping) | Slower (groups all rows first) |

**Example showing both:**

```sql
-- Students older than 18, grouped by department, showing only departments with >10 such students
SELECT Department, COUNT(*) AS Adult_Students
FROM Students
INNER JOIN Classes ON Students.ClassID = Classes.ClassID
WHERE Age > 18                    -- Filter rows BEFORE grouping
GROUP BY Department
HAVING COUNT(*) > 10;            -- Filter groups AFTER grouping
```

---

## **8. Complete Query Examples**

### **8.1 Complex Query with All Clauses**

```sql
SELECT
    c.Department,
    YEAR(s.EnrollmentDate) AS Enrollment_Year,
    COUNT(*) AS Total_Students,
    AVG(s.Grade) AS Average_Grade,
    MIN(s.Grade) AS Minimum_Grade,
    MAX(s.Grade) AS Maximum_Grade
FROM Students s
INNER JOIN Classes c ON s.ClassID = c.ClassID
WHERE s.Age >= 18                    -- Only adult students
    AND c.Department IN ('IS', 'CS', 'IT')  -- Specific departments
    AND s.EnrollmentDate BETWEEN '2023-01-01' AND '2023-12-31'  -- 2023 enrollments
    AND s.Email LIKE '%@university.edu'      -- Valid university emails
GROUP BY c.Department, YEAR(s.EnrollmentDate)
HAVING COUNT(*) > 5                     -- Only groups with >5 students
    AND AVG(s.Grade) >= 70              -- Only groups with average grade >= 70
ORDER BY c.Department ASC, Enrollment_Year DESC, Average_Grade DESC;
```

### **8.2 Query Execution Order**

SQL Server processes queries in this order:

1. **FROM**: Identify tables
2. **JOIN**: Combine tables
3. **WHERE**: Filter rows
4. **GROUP BY**: Group rows
5. **HAVING**: Filter groups
6. **SELECT**: Choose columns
7. **ORDER BY**: Sort results

**Visual Flow:**

```
FROM/JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
```

---

## **9. Advanced SELECT Features**

### **9.1 DISTINCT Keyword**

**Purpose**: Remove duplicate rows from result

**Syntax:**

```sql
SELECT DISTINCT <column1>, <column2>, ...
FROM <table>;
```

**Examples:**

```sql
-- Unique departments
SELECT DISTINCT Department FROM Classes;

-- Unique combinations
SELECT DISTINCT Department, ClassName FROM Classes;
```

### **9.2 TOP / FETCH FIRST**

**Purpose**: Limit number of rows returned

**Syntax:**

```sql
-- SQL Server (TOP)
SELECT TOP <number> <columns> FROM <table> [ORDER BY <column>];

-- Standard SQL (FETCH FIRST)
SELECT <columns> FROM <table>
ORDER BY <column>
FETCH FIRST <number> ROWS ONLY;
```

**Examples:**

```sql
-- Top 10 students by grade
SELECT TOP 10 * FROM Students ORDER BY Grade DESC;

-- Top 5% of products by sales
SELECT TOP 5 PERCENT * FROM Products ORDER BY Sales DESC;
```

### **9.3 UNION / UNION ALL**

**Purpose**: Combine results from multiple SELECT statements

**Syntax:**

```sql
-- Remove duplicates
SELECT <columns> FROM <table1>
UNION
SELECT <columns> FROM <table2>;

-- Keep all rows (faster)
SELECT <columns> FROM <table1>
UNION ALL
SELECT <columns> FROM <table2>;
```

**Example:**

```sql
-- All contacts (students + employees)
SELECT FirstName, LastName, Email, 'Student' AS Type FROM Students
UNION
SELECT FirstName, LastName, Email, 'Employee' AS Type FROM Employees
ORDER BY LastName, FirstName;
```

---

## **10. Performance Tips**

### **10.1 Query Optimization**

1. **Use specific columns** instead of `SELECT *`
2. **Add WHERE clauses** to limit data
3. **Use appropriate JOIN types**
4. **Index frequently queried columns**
5. **Avoid functions on indexed columns in WHERE**

**Bad:**

```sql
SELECT * FROM Students WHERE YEAR(BirthDate) = 2000;
```

**Good:**

```sql
SELECT FirstName, LastName FROM Students
WHERE BirthDate BETWEEN '2000-01-01' AND '2000-12-31';
```

### **10.2 Common Mistakes to Avoid**

1. **Missing JOIN conditions** (Cartesian product)
2. **Confusing WHERE and HAVING**
3. **Using GROUP BY without aggregation**
4. **Alias scope errors**
5. **Case sensitivity issues** (depends on collation)

---

## **11. Quick Reference Cheat Sheet**

### **11.1 JOIN Types Quick Guide**

| Need to...                             | Use JOIN Type                        |
| -------------------------------------- | ------------------------------------ |
| Get only matching records              | INNER JOIN                           |
| Get all from main table + matches      | LEFT JOIN                            |
| Get all from reference table + matches | RIGHT JOIN                           |
| Get all from both tables               | FULL OUTER JOIN                      |
| Get unmatched records only             | LEFT JOIN WHERE right.column IS NULL |

### **11.2 Operator Summary**

| Operator          | Purpose           | Example                         |
| ----------------- | ----------------- | ------------------------------- |
| `=`               | Equal to          | `WHERE Age = 18`                |
| `<>` or `!=`      | Not equal         | `WHERE Department <> 'IT'`      |
| `>` `<` `>=` `<=` | Comparisons       | `WHERE Grade >= 70`             |
| `IN`              | Match list        | `WHERE ID IN (1,2,3)`           |
| `BETWEEN`         | Range             | `WHERE Age BETWEEN 18 AND 25`   |
| `LIKE`            | Pattern match     | `WHERE Name LIKE 'A%'`          |
| `IS NULL`         | Check NULL        | `WHERE Email IS NULL`           |
| `AND` `OR` `NOT`  | Logical operators | `WHERE Age > 18 AND Grade > 70` |

### **11.3 Aggregation Functions**

| Need to...              | Use Function |
| ----------------------- | ------------ |
| Count rows/values       | `COUNT()`    |
| Calculate total         | `SUM()`      |
| Find average            | `AVG()`      |
| Find smallest           | `MIN()`      |
| Find largest            | `MAX()`      |
| Find standard deviation | `STDEV()`    |
| Find variance           | `VAR()`      |

### **11.4 Clause Order Template**

```sql
SELECT
FROM
[JOIN]
[WHERE]
[GROUP BY]
[HAVING]
[ORDER BY]
[TOP/FETCH];
```
