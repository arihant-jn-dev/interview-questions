# SQL Interview Questions & Guide

This document provides a structured set of SQL interview questions, complete with answers, examples, and potential follow-up questions. It is designed to help assess a candidate's understanding of database concepts and their practical SQL skills.

---

## Part 1: Core Database Concepts

### 1. Data Normalization

**Question:** What is Data Normalization and why is it important?

**Answer:**
Data Normalization is the process of organizing columns and tables in a relational database to minimize data redundancy and improve data integrity. The primary goals are to eliminate undesirable characteristics like Insertion, Update, and Deletion Anomalies. This is achieved by dividing larger tables into smaller, well-structured tables and defining relationships between them.

**Follow-up Question 1:** "Can you explain the first three normal forms (1NF, 2NF, 3NF) with a simple example?"

**Explanation & Example:**
Let's start with an unnormalized table of student course data:

**Unnormalized Table:**
| StudentID | StudentName | Courses                               |
|-----------|-------------|---------------------------------------|
| 1         | John Doe    | 'CS101: Intro to CS', 'MA203: Calculus' |
| 2         | Jane Smith  | 'EN101: English Comp'                 |

-   **First Normal Form (1NF):** A table is in 1NF if all its columns contain atomic (indivisible) values, and each row is unique. Our table violates 1NF because the `Courses` column contains multiple values.

    **To achieve 1NF**, we separate the courses into individual rows:
    | StudentID | StudentName | CourseID | CourseName      |
    |-----------|-------------|----------|-----------------|
    | 1         | John Doe    | CS101    | Intro to CS     |
    | 1         | John Doe    | MA203    | Calculus        |
    | 2         | Jane Smith  | EN101    | English Comp    |
    *Problem:* We now have redundant data (`StudentName` is repeated). This leads to update anomalies.

-   **Second Normal Form (2NF):** A table is in 2NF if it is in 1NF and every non-primary-key attribute is fully functionally dependent on the primary key. This rule applies to tables with composite primary keys. Let's assume our primary key is `(StudentID, CourseID)`. `StudentName` depends only on `StudentID`, not the full composite key.

    **To achieve 2NF**, we split the table into two:
    **Students Table:**
    | StudentID | StudentName |
    |-----------|-------------|
    | 1         | John Doe    |
    | 2         | Jane Smith  |

    **Enrollment Table:**
    | StudentID | CourseID | CourseName   |
    |-----------|----------|--------------|
    | 1         | CS101    | Intro to CS  |
    | 1         | MA203    | Calculus     |
    | 2         | EN101    | English Comp |
    *Problem:* `CourseName` depends on `CourseID`, which is not the primary key of the Enrollment table. This is a transitive dependency.

-   **Third Normal Form (3NF):** A table is in 3NF if it is in 2NF and has no transitive dependencies (where a non-key attribute depends on another non-key attribute).

    **To achieve 3NF**, we split the `Enrollment` table:
    **Enrollment Table:**
    | StudentID | CourseID |
    |-----------|----------|
    | 1         | CS101    |
    | 1         | MA203    |
    | 2         | EN101    |

    **Courses Table:**
    | CourseID | CourseName   |
    |----------|--------------|
    | CS101    | Intro to CS  |
    | MA203    | Calculus     |
    | EN101    | English Comp |

Now our database is in 3NF. Redundancy is minimized, and data integrity is improved.

**Follow-up Question 2:** "What are the potential disadvantages of over-normalizing a database?"

**Explanation:**
Over-normalization (e.g., moving to 4NF, 5NF) can lead to a large number of tables. While this reduces redundancy to an absolute minimum, it means that simple queries might require complex joins across many tables. This can degrade query performance and make the database schema harder to understand and maintain. It's a trade-off between data integrity and performance.

---

### 2. Data Integrity

**Question:** What is Data Integrity? Can you explain the different types?

**Answer:**
Data Integrity refers to the accuracy, consistency, and reliability of data stored in a database. It ensures that data is protected from accidental or intentional corruption. There are several types:

1.  **Domain Integrity:** Ensures that all values in a column are from a specified domain. This is enforced using data types (`INT`, `VARCHAR`), `NOT NULL` constraints, and `CHECK` constraints.
2.  **Entity Integrity:** Ensures that every table has a primary key, and that the primary key column(s) cannot contain `NULL` values. This guarantees that every row is uniquely identifiable.
3.  **Referential Integrity:** Ensures that relationships between tables remain consistent. When one table has a foreign key to another table, the value of the foreign key must either be `NULL` or match a value in the primary key of the referenced table. This prevents "orphan" records.
4.  **User-Defined Integrity:** Business rules that don't fall into the other categories, often enforced by triggers or stored procedures.

**Follow-up Question 1:** "How would you enforce Domain Integrity for an email address column to ensure it follows a specific format?"

**Explanation & Example:**
You can use a `CHECK` constraint with a regular expression or pattern matching.

```sql
-- SQL Server Example
CREATE TABLE Users (
    UserID INT PRIMARY KEY,
    Email VARCHAR(255) NOT NULL
        CHECK (Email LIKE '%_@__%.__%') -- Basic pattern check
);

-- Attempting to insert an invalid email will fail
INSERT INTO Users (UserID, Email) VALUES (1, 'invalid-email');
-- ERROR: The INSERT statement conflicted with the CHECK constraint.
```

**Follow-up Question 2:** "What happens if you try to delete a record from a primary table that is referenced by a foreign key in another table?"

**Explanation:**
The default behavior is to block the deletion to maintain referential integrity. However, you can define specific actions using `ON DELETE`:
-   `ON DELETE NO ACTION` or `ON DELETE RESTRICT`: (Default) The deletion is rejected.
-   `ON DELETE CASCADE`: Deleting the parent row will automatically delete all matching rows in the child table.
-   `ON DELETE SET NULL`: Deleting the parent row will set the foreign key column(s) in the child table to `NULL`.

**Example:**
```sql
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    EmployeeName VARCHAR(100),
    DepartmentID INT,
    FOREIGN KEY (DepartmentID) REFERENCES Departments(DepartmentID)
        ON DELETE SET NULL -- If a department is deleted, the employee's department is set to NULL
);
```

---

### 3. Keys (Primary, Foreign, Unique)

**Question:** Explain the difference between a Primary Key, a Foreign Key, and a Unique Key.

**Answer:**
-   **Primary Key:** A constraint that uniquely identifies each record in a table. A primary key must contain unique values and cannot contain `NULL` values. A table can have only one primary key.
-   **Unique Key:** A constraint that ensures all values in a column (or a set of columns) are unique. Unlike a primary key, a unique key can allow one `NULL` value. A table can have multiple unique keys.
-   **Foreign Key:** A key used to link two tables together. It is a field (or collection of fields) in one table that refers to the Primary Key in another table. It enforces referential integrity.

**Follow-up Question 1:** "Can a primary key be composed of more than one column? What is that called?"

**Explanation & Example:**
Yes, this is called a **Composite Primary Key**. It is used when a single column is not sufficient to uniquely identify a row. This is common in linking tables (many-to-many relationships).

```sql
-- A student can enroll in many courses, and a course can have many students.
CREATE TABLE Student_Courses (
    StudentID INT,
    CourseID INT,
    Grade CHAR(1),
    PRIMARY KEY (StudentID, CourseID), -- Composite Key
    FOREIGN KEY (StudentID) REFERENCES Students(StudentID),
    FOREIGN KEY (CourseID) REFERENCES Courses(CourseID)
);
```
Here, the combination of `StudentID` and `CourseID` must be unique.

**Follow-up Question 2:** "Why would you use a Unique Key instead of a Primary Key?"

**Explanation:**
You would use a `UNIQUE` key when you want to enforce uniqueness on a column (or columns) that is not the primary identifier for the table. For example, in an `Employees` table, `EmployeeID` would be the `PRIMARY KEY`. You might also want to ensure that every employee has a unique `EmailAddress` and a unique `SocialSecurityNumber`. These would be excellent candidates for `UNIQUE` constraints. This allows you to have multiple columns with uniqueness guarantees in a single table.

---

### 4. Indexes

**Question:** What is an index in SQL, and what are its main advantages?

**Answer:**
An index is a special lookup table that the database search engine can use to speed up data retrieval. An index is a pointer to data in a table. It works like an index in the back of a book; instead of scanning the entire book (table), you can go to the index to find the page number (data location) for a specific topic. The main advantage is significantly faster query performance for `SELECT` queries with `WHERE` clauses.

**Question:** What are two disadvantages of using an index?

**Answer:**
1.  **Slower Data Modification:** Indexes slow down `INSERT`, `UPDATE`, and `DELETE` operations because the index itself must also be updated every time the data in the table changes.
2.  **Storage Space:** Indexes take up additional disk space. For very large tables, the space consumed by multiple indexes can be substantial.

**Follow-up Question 1:** "What is the difference between a clustered and a non-clustered index?"

**Explanation:**
-   **Clustered Index:** Determines the physical order of data in a table. Because of this, a table can have only **one** clustered index. The leaf nodes of a clustered index contain the actual data pages. A primary key constraint automatically creates a clustered index by default.
-   **Non-Clustered Index:** Has a structure separate from the data rows. The leaf nodes of a non-clustered index contain pointers back to the data rows in the underlying table. A table can have multiple non-clustered indexes.

**Analogy:** A clustered index is like a dictionary, where the words are physically sorted alphabetically. A non-clustered index is like the index at the back of a textbook; it's a separate list of topics with page numbers pointing you to the actual content.

**Follow-up Question 2:** "On which columns would you typically create an index?"

**Explanation:**
You should create indexes on columns that are:
1.  **Frequently used in `WHERE` clauses:** To speed up filtering.
    -   `SELECT * FROM Orders WHERE CustomerID = 123;` (Index on `CustomerID`)
2.  **Used in `JOIN` conditions:** To speed up joining tables.
    -   `SELECT * FROM Orders o JOIN Customers c ON o.CustomerID = c.CustomerID;` (Index on `o.CustomerID` and `c.CustomerID`)
3.  **Frequently used in `ORDER BY` clauses:** To return sorted data more quickly.
    -   `SELECT * FROM Orders ORDER BY OrderDate DESC;` (Index on `OrderDate`)

---

### 5. Joins

**Question:** Explain the difference between a `LEFT JOIN` and a `RIGHT JOIN`.

**Answer:**
-   **`LEFT JOIN`**: Returns all records from the left table (the first one mentioned), and the matched records from the right table. If there is no match, the result is `NULL` on the right side.
-   **`RIGHT JOIN`**: Returns all records from the right table, and the matched records from the left table. If there is no match, the result is `NULL` on the left side.

**Example:**
`Customers` Table: (1, 'Alice'), (2, 'Bob')
`Orders` Table: (101, 1), (102, 1)
```sql
-- LEFT JOIN: Find all customers and their orders, if any.
SELECT c.Name, o.OrderID FROM Customers c LEFT JOIN Orders o ON c.ID = o.CustomerID;
-- Result: ('Alice', 101), ('Alice', 102), ('Bob', NULL) -- Bob has no orders.

-- RIGHT JOIN: Find all orders and the customer who placed them.
SELECT c.Name, o.OrderID FROM Customers c RIGHT JOIN Orders o ON c.ID = o.CustomerID;
-- Result: ('Alice', 101), ('Alice', 102) -- Bob is excluded as he has no orders.
```

**Question:** What is the difference between a `CROSS JOIN` and a `FULL OUTER JOIN`?

**Answer:**
-   **`CROSS JOIN`**: Returns the Cartesian product of the two tables. It combines every row from the first table with every row from the second table. It does not require a join condition.
-   **`FULL OUTER JOIN`**: Returns all records when there is a match in either the left or the right table. It combines the results of both `LEFT JOIN` and `RIGHT JOIN`. If there is no match, the missing side will have `NULL` values.

**Follow-up Question:** "When would a `CROSS JOIN` be useful?"

**Explanation & Example:**
A `CROSS JOIN` is useful for generating a complete set of possible combinations. For example, if you have a `Sizes` table ('S', 'M', 'L') and a `Colors` table ('Red', 'Blue'), a cross join would produce all possible product variants: (S, Red), (S, Blue), (M, Red), (M, Blue), (L, Red), (L, Blue). This is great for creating test data or populating a master list of product combinations.

---

### 6. `TRUNCATE` vs. `DELETE`

**Question:** What is the difference between `TRUNCATE` and `DELETE`?

**Answer:**
-   **`DELETE`**: Is a DML (Data Manipulation Language) command. It removes rows one by one and records an entry in the transaction log for each deleted row. It can be rolled back. `DELETE` statements can also use a `WHERE` clause to remove specific rows.
-   **`TRUNCATE`**: Is a DDL (Data Definition Language) command. It quickly removes all rows from a table by deallocating the data pages. It cannot be rolled back (in most SQL versions) and does not fire `DELETE` triggers. It is much faster than `DELETE` for clearing a table.

**Follow-up Question:** "If you need to remove all records from a large table and reset an identity column, which command would you use and why?"

**Explanation:**
You would use `TRUNCATE TABLE`. `TRUNCATE` is much faster for large tables because it doesn't log individual row deletions. Crucially, it also resets any `IDENTITY` columns back to their starting seed value, which is often the desired behavior when clearing a table for reuse. `DELETE` would remove the rows but leave the identity counter at its last value.

---

### 7. Stored Procedures vs. Functions

**Question:** What are the main differences between a Stored Procedure and a Function?

**Answer:**
-   **Return Value:** A function must return a single scalar value or a table. A stored procedure is not required to return any value but can return values through `OUTPUT` parameters.
-   **Calling:** A function can be called from within a `SELECT` statement. A stored procedure cannot; it must be executed using the `EXEC` or `EXECUTE` command.
-   **Operations:** Stored procedures can perform DML operations (`INSERT`, `UPDATE`, `DELETE`). Functions are generally restricted to read-only operations.

**Follow-up Question:** "Provide a scenario where you would choose to use a function over a stored procedure."

**Explanation & Example:**
Use a function when you need a reusable piece of logic that calculates a value and you want to use that value directly in a query's column list or `WHERE` clause.

**Scenario:** You want to calculate the full name of an employee from `FirstName` and `LastName` columns and use it in multiple queries.

```sql
-- Create the Function
CREATE FUNCTION dbo.GetFullName (@FirstName VARCHAR(50), @LastName VARCHAR(50))
RETURNS VARCHAR(101)
AS
BEGIN
    RETURN @FirstName + ' ' + @LastName;
END;
GO

-- Use the Function in a SELECT statement
SELECT
    dbo.GetFullName(FirstName, LastName) AS FullName,
    Email
FROM Employees
WHERE DepartmentID = 5;
```
This is cleaner and more reusable than writing `FirstName + ' ' + LastName` in every query. A stored procedure could not be used in this way.

---

### 8. Views

**Question:** What is a View? Can a View be used inside a Stored Procedure?

**Answer:**
A View is a virtual table based on the result-set of an SQL statement. It contains rows and columns, just like a real table, but it does not store the data itself. Views are used to simplify complex queries, restrict access to data, and present data in a specific format.

Yes, a View can absolutely be used inside a Stored Procedure. It is common practice to query a view from within a procedure to simplify the logic and promote code reuse.

**Follow-up Question:** "What are the benefits of using a view to grant data access to a user instead of giving them direct table access?"

**Explanation & Example:**
Using a view for security provides two main benefits:
1.  **Column-Level Security:** You can expose only specific, non-sensitive columns to the user.
2.  **Row-Level Security:** You can filter the rows that the user is allowed to see.

**Scenario:** You want to allow an analyst to see employee names and departments, but not their salaries or personal information.

```sql
-- Create a secure view
CREATE VIEW vw_EmployeeDirectory AS
SELECT
    EmployeeName,
    DepartmentName
FROM Employees e
JOIN Departments d ON e.DepartmentID = d.DepartmentID
WHERE e.IsActive = 1; -- Only show active employees

-- Grant access to the view, not the underlying tables
GRANT SELECT ON vw_EmployeeDirectory TO AnalystUser;
```
The analyst can now query `vw_EmployeeDirectory` but has no access to the `Employees` or `Departments` tables directly, protecting sensitive data like salary.

---

## Part 2: SQL Query Problems

### Query 1: Pivot Data

**Problem:**
Given the table below, write a query to find `BookType`-wise sales where `SalesYear` is displayed as column names.

| BookType   | SalesYear | BookSales |
|------------|-----------|-----------|
| Fiction    | 2013      | 10436     |
| Fiction    | 2013      | 9346      |
| Nonfiction | 2014      | 7214      |
| Nonfiction | 2014      | 5800      |

**Answer:**
```sql
SELECT BookType, [2013], [2014]
FROM (
    SELECT BookType, SalesYear, BookSales
    FROM Sales
) AS SourceTable
PIVOT (
    SUM(BookSales)
    FOR SalesYear IN ([2013], [2014])
) AS PivotTable;
```
**Result:**
| BookType   | 2013  | 2014  |
|------------|-------|-------|
| Fiction    | 19782 | NULL  |
| Nonfiction | NULL  | 13014 |

**Follow-up Question:** "What if you wanted to add a new year, 2015, to the columns, but you don't know the years in advance? How would you modify this query? What is this approach called?"

**Explanation:**
This requires **Dynamic SQL**. Since the `PIVOT` clause requires hardcoded column names, you must first construct the query as a string and then execute it.

**Example (SQL Server):**
```sql
DECLARE @columns NVARCHAR(MAX), @sql NVARCHAR(MAX);

-- 1. Get a comma-separated list of the years
SET @columns = STUFF((SELECT DISTINCT ',' + QUOTENAME(SalesYear)
                      FROM Sales
                      FOR XML PATH(''), TYPE).value('.', 'NVARCHAR(MAX)'), 1, 1, '');

-- 2. Construct the full dynamic query string
SET @sql = N'
SELECT BookType, ' + @columns + N'
FROM (
    SELECT BookType, SalesYear, BookSales
    FROM Sales
) AS SourceTable
PIVOT (
    SUM(BookSales)
    FOR SalesYear IN (' + @columns + N')
) AS PivotTable;';

-- 3. Execute the query
EXEC sp_executesql @sql;
```

---

### Query 2: Optimizing `NOT IN`

**Problem:**
How can you optimize the below query if both tables `T1` and `T2` have millions of records?
`SELECT * FROM T1 WHERE T1_ID NOT IN (SELECT T2_ID FROM T2)`

**Answer:**
The `NOT IN` operator can perform poorly, especially if the subquery (`SELECT T2_ID FROM T2`) returns any `NULL` values, which would cause the entire query to return no results. Better alternatives are `LEFT JOIN` or `NOT EXISTS`.

**Option 1: Using `LEFT JOIN`**
This is often the most performant method.
```sql
SELECT T1.*
FROM T1
LEFT JOIN T2 ON T1.T1_ID = T2.T2_ID
WHERE T2.T2_ID IS NULL;
```

**Option 2: Using `NOT EXISTS`**
This is also highly efficient and clearly expresses the intent.
```sql
SELECT T1.*
FROM T1
WHERE NOT EXISTS (
    SELECT 1
    FROM T2
    WHERE T2.T2_ID = T1.T1_ID
);
```
For both solutions, performance is greatly improved by having an index on the `T2_ID` column of table `T2`.

**Follow-up Question:** "Why is the `LEFT JOIN / IS NULL` pattern often faster than `NOT IN`?"

**Explanation:**
The SQL query optimizer is generally better at optimizing `JOIN` operations than subqueries. The `LEFT JOIN / IS NULL` pattern allows the optimizer to use more efficient join algorithms (like hash or merge joins) and leverage indexes effectively. Furthermore, `NOT IN` has a critical flaw: if any value returned by the subquery is `NULL`, the entire outer query returns an empty set because the condition `T1_ID NOT IN (val1, val2, NULL)` evaluates to "unknown," which is treated as false. `NOT EXISTS` and `LEFT JOIN` do not have this problem.

---

### Query 3: Common Records

**Problem:**
Write a query to fetch common records between two tables, `EmployeeSalary` and `ManagerSalary`, based on `EmpId`.

**Answer:**
```sql
-- Using INNER JOIN (most common and readable)
SELECT ES.*
FROM EmployeeSalary ES
INNER JOIN ManagerSalary MS ON ES.EmpId = MS.EmpId;

-- Using IN (as provided)
SELECT *
FROM EmployeeSalary
WHERE EmpId IN (SELECT EmpId FROM ManagerSalary);

-- Using EXISTS (often more performant than IN for large subqueries)
SELECT *
FROM EmployeeSalary ES
WHERE EXISTS (SELECT 1 FROM ManagerSalary MS WHERE ES.EmpId = MS.EmpId);
```

---

### Query 4: Employees Who Are Also Managers

**Problem:**
Write a query to fetch all employees who are also managers from the `EmployeeDetails` table. Assume the table has `EmpID` and `ManagerID` columns.

**Answer:**
```sql
SELECT DISTINCT M.*
FROM EmployeeDetails E
INNER JOIN EmployeeDetails M ON E.ManagerID = M.EmpID;
```
This query joins the `EmployeeDetails` table to itself. It finds employees whose `EmpID` appears in the `ManagerID` column, meaning they manage at least one other person.

---

### Query 5: Records from Either Table

**Problem:**
Write a query to fetch all `EmpIds` that are present in either `EmployeeDetails` or `EmployeeSalary`.

**Answer:**
```sql
-- Using UNION
SELECT EmpId FROM EmployeeDetails
UNION
SELECT EmpId FROM EmployeeSalary;
```
`UNION` combines the results and automatically removes duplicate `EmpIds`. If you wanted to keep duplicates (i.e., an `EmpId` appears in both tables and you want it listed twice), you would use `UNION ALL`.

---

### Query 6: Remove Duplicates

**Problem:**
Write a query to remove duplicate records from `EmployeeDetails` without using a temporary table, based on `FullName`.

**Answer:**
You can use a Common Table Expression (CTE) with the `ROW_NUMBER()` window function.

```sql
WITH CTE AS (
    SELECT
        EmpId,
        FullName,
        ROW_NUMBER() OVER(PARTITION BY FullName ORDER BY EmpId) AS RowNum
    FROM EmployeeDetails
)
DELETE FROM CTE WHERE RowNum > 1;
```
**Explanation:**
1.  `PARTITION BY FullName`: This groups rows with the same `FullName` together.
2.  `ROW_NUMBER() ... ORDER BY EmpId`: This assigns a sequential number to each row within its group. The `ORDER BY` determines which row gets `RowNum = 1` (this will be the one that is kept).
3.  `DELETE FROM CTE WHERE RowNum > 1`: This deletes all rows that are not the first one in their group, effectively removing the duplicates.

---

### Query 7: String Manipulation

**Problem:**
1.  Write a query to fetch employee full names and replace the space with ‘-’.
2.  Write a query to fetch only the first name from the `FullName` column.
3.  Write a query to find the count of the character ‘n’ in the `FullName` field for each employee.

**Answers:**
```sql
-- 1. Replace space with '-'
SELECT REPLACE(FullName, ' ', '-') AS DashedName
FROM EmployeeDetails;

-- 2. Fetch first name (assuming 'FirstName LastName' format)
SELECT SUBSTRING(FullName, 1, CHARINDEX(' ', FullName) - 1) AS FirstName
FROM EmployeeDetails;

-- 3. Count occurrences of 'n'
SELECT FullName, (LEN(FullName) - LEN(REPLACE(LOWER(FullName), 'n', ''))) AS CountOfN
FROM EmployeeDetails;
```
*Note on #3: It's good practice to use `LOWER()` to make the search case-insensitive, counting both 'n' and 'N'.*
