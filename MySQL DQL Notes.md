## **1. What is DQL?**
**DQL (Data Query Language)** is the subset of SQL used to **retrieve, query, and fetch data** from databases. While sometimes considered part of DML, DQL specifically focuses on reading data using the `SELECT` statement and its various clauses.
**Primary DQL Command:**
- `SELECT` - Retrieve data from one or more tables
**DQL vs DML vs DDL:**
- **DDL**: Define/modify structure (`CREATE`, `ALTER`, `DROP`)
- **DML**: Manipulate data (`INSERT`, `UPDATE`, `DELETE`)
- **DQL**: Query/retrieve data (`SELECT`)

---
## **2. Basic SELECT Statement**
### **2.1 Selecting from One Table**
**Syntax:**
```sql
SELECT column1, column2, column3, ...
FROM table_name;

-- Select all columns
SELECT * FROM table_name;
```
**Example:**
```sql
SELECT FirstName, LastName, Email
FROM Students;

SELECT * FROM Students;
```
---
### **2.2 Database Context (MySQL)**
```sql
USE SchoolDB;
SELECT * FROM Students;
```
---
## **3. Joining Multiple Tables**
### **3.1 JOIN Fundamentals**
**Rules:**
1. Tables should be logically related
2. Use a matching column (usually PK ↔ FK)
**JOIN Types in MySQL:**
- `INNER JOIN`
- `LEFT JOIN`
- `RIGHT JOIN`
---
### **3.2 INNER JOIN**
```sql
SELECT columns
FROM table1
INNER JOIN table2
ON table1.column = table2.column;
```

```sql
SELECT s.FirstName, s.LastName, c.ClassName
FROM Students s
INNER JOIN Classes c ON s.ClassID = c.ClassID;
```
---
### **3.3 LEFT JOIN**
```sql
SELECT columns
FROM table1
LEFT JOIN table2
ON table1.column = table2.column;
```

```sql
SELECT s.FirstName, c.ClassName
FROM Students s
LEFT JOIN Classes c ON s.ClassID = c.ClassID;
```
---
### **3.4 RIGHT JOIN**
```sql
SELECT columns
FROM table1
RIGHT JOIN table2
ON table1.column = table2.column;
```
---
### **3.5 JOIN Summary**

| JOIN Type  | Description         |
| ---------- | ------------------- |
| INNER JOIN | Matching rows only  |
| LEFT JOIN  | All left + matches  |
| RIGHT JOIN | All right + matches |

---
## **4. SELECT Clauses and Keywords**
### **4.1 WHERE**
```sql
SELECT * FROM Students
WHERE Grade > 80;
```

```sql
SELECT * FROM Students
WHERE Age >= 18 AND Grade > 70;
```
---
### **4.2 ORDER BY**
```sql
SELECT * FROM Students
ORDER BY Grade DESC, LastName ASC;
```
- `ASC` default
- `DESC` for descending
---
### **4.3 AS (Alias)**
```sql
SELECT FirstName AS FN FROM Students;

SELECT s.FirstName, c.ClassName
FROM Students AS s
JOIN Classes AS c ON s.ClassID = c.ClassID;
```
---
### **4.4 IN**
```sql
SELECT * FROM Students
WHERE Department IN ('CS', 'IT', 'IS');
```
---
### **4.5 BETWEEN**
```sql
SELECT * FROM Students
WHERE Age BETWEEN 18 AND 25;
```
---
### **4.6 LIKE**
```sql
SELECT * FROM Students
WHERE FirstName LIKE 'A%';
```
**Wildcards:**
- `%` → any characters
- `_` → single character
---
### **4.7 NOT**
```sql
SELECT * FROM Students
WHERE NOT Grade > 70;
```
---
## **5. Subqueries**
### **5.1 IN Subquery**
```sql
SELECT FirstName
FROM Students
WHERE ClassID IN (
    SELECT ClassID
    FROM Classes
    WHERE Number_of_Students > 10
);
```
---
### **5.2 EXISTS**
```sql
SELECT *
FROM Students s
WHERE EXISTS (
    SELECT 1
    FROM Enrollments e
    WHERE e.StudentID = s.StudentID
);
```
---
### **5.3 Correlated Subquery**

```sql
SELECT *
FROM Employees e1
WHERE Salary > (
    SELECT AVG(Salary)
    FROM Employees e2
    WHERE e1.DepartmentID = e2.DepartmentID
);
```
---
## **6. Aggregation Functions**

### **6.1 COUNT**

```sql
SELECT COUNT(*) FROM Students;

SELECT COUNT(DISTINCT Department) FROM Classes;
```
---
### **6.2 SUM**

```sql
SELECT SUM(Salary) FROM Employees;
```
---
### **6.3 AVG**

```sql
SELECT AVG(Grade) FROM Students;
```
---
### **6.4 MIN / MAX**

```sql
SELECT MIN(Grade), MAX(Grade)
FROM Students;
```
---
## **7. GROUP BY**

### **7.1 Basic**
```sql
SELECT Department, COUNT(*) AS Student_Count
FROM Students
GROUP BY Department;
```
---
### **7.2 HAVING**
```sql
SELECT Department, COUNT(*) AS Student_Count
FROM Students
GROUP BY Department
HAVING COUNT(*) > 10;
```
---

### **7.3 WHERE vs HAVING**

|WHERE|HAVING|
|---|---|
|Before grouping|After grouping|
|Filters rows|Filters groups|
|No aggregation|Supports aggregation|

---
## **8. Complete Query Example**
```sql
SELECT
    c.Department,
    YEAR(s.EnrollmentDate) AS Year,
    COUNT(*) AS Total_Students,
    AVG(s.Grade) AS Avg_Grade
FROM Students s
JOIN Classes c ON s.ClassID = c.ClassID
WHERE s.Age >= 18
    AND c.Department IN ('CS', 'IT')
GROUP BY c.Department, YEAR(s.EnrollmentDate)
HAVING COUNT(*) > 5
ORDER BY Avg_Grade DESC;
```
---

## **9. Advanced SELECT Features (MySQL)**
### **9.1 DISTINCT**
```sql
SELECT DISTINCT Department FROM Classes;
```
---
### **9.2 LIMIT (MySQL alternative to TOP)**

```sql
SELECT * FROM Students
ORDER BY Grade DESC
LIMIT 10;
```

```sql
-- Pagination
SELECT * FROM Students
LIMIT 10 OFFSET 20;
```

---
### **9.3 UNION / UNION ALL**

```sql
SELECT Name FROM Students
UNION
SELECT Name FROM Employees;
```
---

## **10. Query Execution Order (MySQL)**

```
FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```
---

## **11. Performance Tips**

- Avoid `SELECT *`
- Use indexes on filtered columns
- Filter early with `WHERE`
- Avoid functions on indexed columns

---

## **12. Quick Cheat Sheet**
### **JOIN Guide**

|Goal|Use|
|---|---|
|Matching only|INNER JOIN|
|Keep all left|LEFT JOIN|
|Keep all right|RIGHT JOIN|
|Full merge|UNION workaround|

---
### **Operators**

|Operator|Example|
|---|---|
|`=`|`Age = 18`|
|`!=`|`Age != 18`|
|`IN`|`ID IN (1,2,3)`|
|`BETWEEN`|`Age BETWEEN 18 AND 25`|
|`LIKE`|`Name LIKE 'A%'`|
|`IS NULL`|`Email IS NULL`|

---
### **Aggregation**

|Task|Function|
|---|---|
|Count|`COUNT()`|
|Sum|`SUM()`|
|Avg|`AVG()`|
|Min|`MIN()`|
|Max|`MAX()`|

---
### **Clause Order Template**

```sql
SELECT
FROM
[JOIN]
WHERE
GROUP BY
HAVING
ORDER BY
LIMIT;
```