# DBMS_Placement_Interview_Notes

# DBMS Placement & Interview Notes

## 1. What exactly is a DBMS?

A **Database Management System** is software that allows applications and users to store, retrieve, modify, secure, and manage structured data.

A database is the data itself. The DBMS is the software responsible for managing that data.

Examples include MySQL, PostgreSQL, Oracle Database, SQL Server, and MongoDB, although MongoDB uses a document-oriented model rather than the relational model.

A relational database represents data mainly using:

- **Relation** → table
- **Tuple** → row
- **Attribute** → column
- **Schema** → structure of the database
- **Instance** → actual data stored at a particular moment

### Why use a DBMS instead of files?

Imagine two applications modifying the same bank-account file simultaneously.

With ordinary file handling, you would have to solve problems such as:

- synchronization
- duplicate data
- authorization
- crash recovery
- searching large files
- consistency between related records

A DBMS provides mechanisms for all of these.

### Interview answer

> A DBMS provides structured storage along with efficient querying, concurrency control, security, integrity constraints, and crash recovery.
> 

---

# 2. Relational Model

A relational database organizes information into tables.

For example:

```
STUDENT

student_id | name   | department
-----------|--------|-----------
101        | Ravi   | CSE
102        | Ankit  | ECE
```

The **schema** describes the structure:

```
STUDENT(student_id, name, department)
```

while the actual rows at any moment form the **database instance**.

---

# 3. Keys

Keys are among the most frequently tested DBMS topics.

Suppose:

```
STUDENT(
    roll_no,
    email,
    name,
    phone
)
```

and `roll_no`, `email`, and `phone` are individually unique.

## Super Key

Any set of attributes capable of uniquely identifying a row.

Examples:

```
{roll_no}
{email}
{roll_no, name}
{email, phone}
```

The last two contain unnecessary attributes but still uniquely identify rows.

## Candidate Key

A **minimal super key**.

It uniquely identifies a row, but if you remove even one attribute, it loses that property.

Possible candidate keys:

```
{roll_no}
{email}
{phone}
```

### Memory relationship

```
Candidate key = minimal super key
```

## Primary Key

One candidate key chosen as the primary identifier.

```
PRIMARY KEY (roll_no)
```

It must be:

- unique
- NOT NULL

## Alternate Key

Candidate keys that were not selected as the primary key.

If `roll_no` is primary:

```
email
phone
```

are alternate keys.

## Composite Key

A key containing multiple attributes.

Example:

```
ENROLLMENT(student_id, course_id)
```

Neither attribute alone identifies the enrollment, but together they may.

## Foreign Key

An attribute in one table that refers to a key in another table.

```
STUDENT
student_id PK

ENROLLMENT
student_id FK
course_id
```

Foreign keys establish relationships between tables and enforce **referential integrity**.

---

# 4. Integrity Constraints

Integrity constraints prevent invalid database states.

## Entity Integrity

A primary key cannot be NULL.

Otherwise, the row cannot be uniquely identified.

## Referential Integrity

A foreign key must either:

- reference an existing parent row
- or be NULL, if NULL is allowed

## Domain Integrity

Column values must satisfy their allowed domain.

```sql
age INT CHECK (age >= 0)
```

## User-defined constraints

Business-specific rules.

```sql
salary > 0
```

---

# 5. ER Model

For placement interviews, understand the concepts rather than memorizing every diagram symbol.

## Entity

A real-world object whose data we store.

Examples:

```
Student
Employee
Course
Order
```

## Strong Entity

Can be uniquely identified using its own attributes.

## Weak Entity

Cannot be uniquely identified independently. It depends on an owner entity.

Example:

```
EMPLOYEE
employee_id

DEPENDENT
name
```

`name` alone may not identify a dependent.

Instead:

```
(employee_id, dependent_name)
```

can identify it.

---

# 6. Cardinality

Cardinality describes how entities participate in a relationship.

## One-to-One

```
Person ↔ Passport
```

## One-to-Many

```
Department → Employees
```

One department may have many employees. Each employee belongs to one department.

## Many-to-Many

```
Students ↔ Courses
```

A student may take many courses and a course may contain many students.

In relational databases, this normally becomes a junction table:

```
ENROLLMENT
student_id
course_id
```

---

# 7. Functional Dependencies

Functional dependencies form the basis of normalization.

If:

```
A → B
```

then whenever two rows have the same `A`, they must also have the same `B`.

Example:

```
student_id → student_name
```

Knowing `student_id` determines the student’s name.

---

# 8. Attribute Closure

The closure of `X`, written:

```
X+
```

contains every attribute that can be derived from `X`.

Suppose:

```
A → B
B → C
C → D
```

Then:

```
A+ = {A, B, C, D}
```

If `A+` contains **every attribute in the relation**, then `A` is a super key.

### Candidate-key method

1. Compute closure.
2. Check whether it contains all attributes.
3. Remove unnecessary attributes.
4. If no attribute can be removed, you have a candidate key.

---

# 9. Armstrong’s Axioms

Three fundamental rules:

### Reflexivity

If `B ⊆ A`, then:

```
A → B
```

### Augmentation

If:

```
A → B
```

then:

```
AC → BC
```

### Transitivity

If:

```
A → B
B → C
```

then:

```
A → C
```

Useful derived rules include:

```
Union
Decomposition
Pseudo-transitivity
```

---

# 10. Normalization

Normalization reduces unnecessary redundancy and the anomalies caused by it.

## Why normalize?

Suppose:

```
student_id | student_name | course_id | course_name | teacher
```

Course information gets duplicated for every enrolled student.

This creates three classic anomalies.

### Update anomaly

Changing one course’s teacher may require modifying dozens of rows. Miss one row and your database becomes inconsistent.

### Insertion anomaly

You may be unable to add a new course until a student enrolls.

### Deletion anomaly

Deleting the final student enrolled in a course could accidentally remove the only stored information about that course.

---

# 11. First Normal Form: 1NF

Every attribute should contain an atomic value.

Bad:

```
phone = "99999, 88888"
```

Better:

```
STUDENT_PHONE
student_id | phone
```

> **1NF → atomic values**
> 

---

# 12. Second Normal Form: 2NF

Must already satisfy 1NF.

Additionally:

> No non-prime attribute should depend on only part of a candidate key.
> 

This matters mainly when candidate keys are composite.

Suppose:

```
ENROLLMENT(
    student_id,
    course_id,
    student_name
)
```

with key:

```
(student_id, course_id)
```

but:

```
student_id → student_name
```

`student_name` depends only on part of the key. That is a **partial dependency**.

> **2NF → no partial dependency**
> 

---

# 13. Third Normal Form: 3NF

Must satisfy 2NF and eliminate problematic transitive dependencies.

Example:

```
student_id → department_id
department_id → department_name
```

Therefore:

```
student_id → department_name
```

indirectly.

For every FD `X → A`, at least one must hold:

- `X` is a super key
- `A` is a prime attribute

> **3NF → no problematic non-key → non-key dependency**
> 

---

# 14. BCNF

BCNF is stricter than 3NF.

For every non-trivial functional dependency:

```
X → Y
```

`X` must be a super key.

> **BCNF → every determinant must be a super key**
> 

Every BCNF relation is in 3NF. Every 3NF relation is **not necessarily** BCNF.

---

# 15. Lossless Join and Dependency Preservation

## Lossless decomposition

Joining decomposed tables should reconstruct the original information without introducing spurious tuples.

For binary decomposition `R → R1, R2`, a common textbook test is whether:

```
R1 ∩ R2
```

functionally determines either `R1` or `R2`.

## Dependency preservation

Dependencies should preferably remain enforceable without joining decomposed relations.

> 3NF decomposition can preserve both losslessness and dependencies, whereas BCNF decomposition may sacrifice dependency preservation.
> 

---

# 16. Transactions

A **transaction** is a logical unit of database work.

Example:

```
Transfer ₹500 from A to B

A = A - 500
B = B + 500
```

Both changes must succeed together.

---

# 17. ACID Properties

## Atomicity

All operations happen or none happen.

> **All or nothing**
> 

## Consistency

A transaction must preserve database rules and constraints.

## Isolation

Concurrent transactions should behave according to the chosen isolation guarantees without producing unacceptable interference.

## Durability

Once committed, changes survive crashes.

```
A → All or nothing
C → Constraints remain valid
I → Independent execution semantics
D → Durable committed data
```

---

# 18. Transaction States

Typical flow:

```
Active
  ↓
Partially Committed
  ↓
Committed
  ↓
Terminated
```

Failure:

```
Active
  ↓
Failed
  ↓
Aborted
  ↓
Restart / Terminate
```

### Active

Transaction is executing.

### Partially committed

Final statement has executed, but commit is not yet guaranteed durable.

### Committed

Transaction succeeds.

### Failed

Transaction can no longer continue.

### Aborted

Changes are rolled back.

---

# 19. Concurrency Problems

## Dirty Read

A transaction reads another transaction’s **uncommitted** value.

```
T1 writes X = 500
T2 reads X = 500
T1 rolls back
```

T2 used data that never officially existed.

## Non-repeatable Read

A transaction reads the same row twice but receives different values.

```
T1 reads salary = 50k

T2 updates salary = 60k
T2 commits

T1 reads again → 60k
```

## Phantom Read

The same query returns a different **set of rows**.

```sql
SELECT * FROM Employee WHERE salary > 50000;
```

If another transaction inserts a qualifying employee and commits, rerunning the query may return an additional row.

## Lost Update

Two transactions read the same old value and then overwrite each other’s work.

```
X = 100

T1 reads 100
T2 reads 100

T1 writes 110
T2 writes 120
```

T1’s update is lost.

---

# 20. Isolation Levels

| Isolation Level | Dirty Read | Non-repeatable Read | Phantom |
| --- | --- | --- | --- |
| Read Uncommitted | Possible | Possible | Possible |
| Read Committed | Prevented | Possible | Possible |
| Repeatable Read | Prevented | Prevented | Traditionally possible |
| Serializable | Prevented | Prevented | Prevented |

```
Read Uncommitted
       ↓
Read Committed
       ↓
Repeatable Read
       ↓
Serializable
```

Higher isolation generally means stronger consistency guarantees but potentially lower concurrency.

---

# 21. Shared and Exclusive Locks

## Shared Lock: S

Used for reading.

```
S + S → allowed
```

## Exclusive Lock: X

Used when writing.

```
S + X → blocked
X + S → blocked
X + X → blocked
```

---

# 22. Two-Phase Locking: 2PL

### Growing Phase

Locks may be acquired. Locks cannot be released.

### Shrinking Phase

Locks may be released. No new locks may be acquired.

Basic 2PL guarantees **conflict serializability**, but deadlocks can still occur.

### Strict 2PL

Exclusive locks are held until commit/abort.

### Rigorous 2PL

All locks are held until commit/abort.

### Conservative 2PL

Acquire all required locks before execution. This prevents deadlocks but reduces concurrency.

---

# 23. Deadlock

```
T1 holds A and waits for B
T2 holds B and waits for A
```

Neither can proceed.

A **wait-for graph** cycle indicates deadlock.

Possible handling:

- deadlock detection + victim rollback
- timeout
- lock ordering
- timestamp-based prevention

---

# 24. Serializability

A concurrent schedule is desirable when it produces the same result as some correct serial execution.

## Conflict Serializability

Two operations conflict if:

1. they belong to different transactions
2. they access the same data item
3. at least one operation is a write

```
R-R → no conflict
R-W → conflict
W-R → conflict
W-W → conflict
```

Build a **precedence graph**.

```
No cycle → conflict serializable
Cycle    → not conflict serializable
```

For normal placements, know conflict serializability well. **View serializability is lower priority** unless the interview is DBMS-heavy.

---

# 25. MVCC

**Multi-Version Concurrency Control** maintains multiple versions of database records.

```
T1 → reads an appropriate older committed version
T2 → writes a newer version
```

> Readers can often obtain a consistent snapshot while writers continue modifying newer versions.
> 

---

# 26. Indexing

An index is an auxiliary data structure that allows the DBMS to locate rows without scanning every record.

```
Book without index:
scan every page

Book with index:
find topic → jump to page
```

Indexes improve reads but introduce:

- additional storage
- additional insert cost
- additional delete cost
- additional update cost

because indexes must be maintained whenever indexed data changes.

---

# 27. Primary Index

Using textbook terminology, a primary index is created when the data file is ordered according to its key field.

Typically sparse:

```
100 → Block 1
200 → Block 2
300 → Block 3
```

Each index entry can point to a block because records inside the block are already ordered.

---

# 28. Clustering Index

A clustering index is built when records are physically ordered by a **non-key attribute**.

```
department = CSE
department = CSE
department = CSE
department = ECE
department = ECE
```

The index may point to the beginning of each group:

```
CSE → first CSE block
ECE → first ECE block
```

It can therefore be sparse.

---

# 29. Secondary Index

A secondary index provides an access path on an attribute that does **not determine the physical ordering of the file**.

Because matching records may be scattered, the index must be dense enough to locate them.

```
Primary    → ordered key      → usually sparse
Clustering → ordered non-key  → usually sparse
Secondary  → non-ordering     → generally dense
```

---

# 30. B-Tree vs B+ Tree

## Why not use an ordinary BST?

A binary search tree has only two children per node. A B+ tree may have hundreds of children per internal node.

That gives it a very high **fan-out**.

```
High fan-out
    ↓
Small tree height
    ↓
Fewer disk-page reads
```

## B+ Tree

Internal nodes mainly store:

```
keys + child pointers
```

Record pointers/data entries live at the leaves.

Leaves are linked:

```
Leaf 1 → Leaf 2 → Leaf 3 → Leaf 4
```

Therefore B+ trees are excellent for both exact and range queries.

---

# 31. Hash Index

Hash indexes excel at equality lookup.

```sql
WHERE user_id = 125
```

Conceptually:

```
hash(125)
    ↓
bucket
    ↓
record location
```

Hashing does not naturally preserve ordering, so range queries are generally much better suited to B/B+ tree indexes.

```
Hash   → equality
B+Tree → equality + range + ordering
```

---

# 32. Composite Indexes

Suppose:

```sql
INDEX(A, B, C)
```

The index is ordered first by `A`, then by `B` inside the same `A`, then by `C`.

It naturally supports prefixes:

```
A
A,B
A,B,C
```

This is the **leftmost-prefix rule**.

### Efficient

```sql
WHERE A = ?
```

```sql
WHERE A = ? AND B = ?
```

```sql
WHERE A = ? AND B = ? AND C = ?
```

### Usually not efficient through this index alone

```sql
WHERE B = ?
```

### Important nuance

```sql
WHERE A = ? AND C = ?
```

can still use the `A` prefix and then filter using `C`.

---

# 33. Why might the optimizer ignore an index?

A table scan may be cheaper when:

- most rows will be returned
- the table is tiny
- the index has poor selectivity
- the compound-index prefix doesn’t match
- an expression prevents straightforward index lookup
- optimizer statistics estimate scanning to be cheaper

---

# 34. Query Execution

```
SQL Query
   ↓
Parser
   ↓
Logical representation
   ↓
Optimizer
   ↓
Execution Plan
   ↓
Execution Engine
   ↓
Result
```

The optimizer may choose:

- which index to use
- join order
- join algorithm
- whether to scan
- whether sorting is necessary

---

# SQL: Complete Placement Revision

# 35. SQL Command Categories

## DDL

```sql
CREATE
ALTER
DROP
TRUNCATE
```

## DML

```sql
INSERT
UPDATE
DELETE
```

`SELECT` is sometimes separately classified as DQL depending on convention.

## DCL

```sql
GRANT
REVOKE
```

## TCL

```sql
COMMIT
ROLLBACK
SAVEPOINT
```

---

# 36. SELECT

```sql
SELECT name, salary
FROM Employee;
```

All columns:

```sql
SELECT *
FROM Employee;
```

Remove duplicate output rows:

```sql
SELECT DISTINCT department
FROM Employee;
```

---

# 37. WHERE

```sql
SELECT *
FROM Employee
WHERE salary > 50000;
```

Multiple conditions:

```sql
WHERE salary > 50000
AND department = 'CSE'
```

---

# 38. ORDER BY

```sql
SELECT *
FROM Employee
ORDER BY salary DESC;
```

Multiple levels:

```sql
ORDER BY department ASC, salary DESC;
```

---

# 39. SQL Logical Execution Order

```
FROM
↓
WHERE
↓
GROUP BY
↓
HAVING
↓
SELECT
↓
ORDER BY
↓
LIMIT
```

`WHERE` occurs **before grouping**, while `HAVING` operates **after grouping**.

---

# 40. Aggregate Functions

```sql
COUNT()
SUM()
AVG()
MIN()
MAX()
```

Example:

```sql
SELECT AVG(salary)
FROM Employee;
```

---

# 41. GROUP BY

```sql
SELECT department, AVG(salary)
FROM Employee
GROUP BY department;
```

Conceptually:

```
CSE employees → average
ECE employees → average
IT employees  → average
```

---

# 42. WHERE vs HAVING

## WHERE

Filters **rows before grouping**.

## HAVING

Filters **groups after grouping**.

```sql
SELECT department, AVG(salary)
FROM Employee
GROUP BY department
HAVING AVG(salary) > 60000;
```

```
WHERE  → rows
HAVING → groups
```

---

# 43. Joins

Suppose:

```
EMPLOYEE
employee_id
name
department_id
```

and:

```
DEPARTMENT
department_id
department_name
```

## INNER JOIN

Returns matching rows from both tables.

```sql
SELECT e.name, d.department_name
FROM Employee e
JOIN Department d
ON e.department_id = d.department_id;
```

---

# 44. LEFT JOIN

Returns every row from the left table. If there is no match on the right, right-side columns become NULL.

```sql
SELECT e.name, d.department_name
FROM Employee e
LEFT JOIN Department d
ON e.department_id = d.department_id;
```

---

# 45. RIGHT JOIN

Same idea as LEFT JOIN but preserves the right table.

Most queries can be rewritten by reversing tables and using LEFT JOIN.

---

# 46. FULL OUTER JOIN

Returns:

```
matches
+
unmatched left rows
+
unmatched right rows
```

Not every database supports `FULL OUTER JOIN` directly.

---

# 47. CROSS JOIN

Cartesian product:

```sql
SELECT *
FROM A
CROSS JOIN B;
```

If A has 5 rows and B has 10 rows, the result has 50 rows.

---

# 48. SELF JOIN

Classic employee-manager example:

```sql
SELECT
    e.name AS employee,
    m.name AS manager
FROM Employee e
LEFT JOIN Employee m
ON e.manager_id = m.id;
```

---

# 49. Subqueries

```sql
SELECT *
FROM Employee
WHERE salary > (
    SELECT AVG(salary)
    FROM Employee
);
```

The inner query calculates the average. The outer query retrieves employees above it.

---

# 50. Correlated Subquery

A correlated subquery depends on the current row of the outer query.

```sql
SELECT e1.*
FROM Employee e1
WHERE salary > (
    SELECT AVG(e2.salary)
    FROM Employee e2
    WHERE e2.department = e1.department
);
```

For every employee, compare their salary against the average of **their own department**.

---

# 51. IN vs EXISTS

## IN

```sql
WHERE department_id IN (1, 2, 3)
```

## EXISTS

Checks whether at least one matching row exists.

```sql
SELECT *
FROM Employee e
WHERE EXISTS (
    SELECT 1
    FROM Project p
    WHERE p.employee_id = e.id
);
```

---

# 52. NOT EXISTS

Useful for absence questions.

```sql
SELECT *
FROM Employee e
WHERE NOT EXISTS (
    SELECT 1
    FROM Project p
    WHERE p.employee_id = e.id
);
```

---

# 53. CTE

A **Common Table Expression** gives a query result a temporary name.

```sql
WITH HighEarners AS (
    SELECT *
    FROM Employee
    WHERE salary > 50000
)
SELECT *
FROM HighEarners;
```

CTEs are especially useful for readability and structuring complex queries.

---

# 54. CTE vs Subquery

Subquery:

```sql
SELECT *
FROM (
    SELECT *
    FROM Employee
    WHERE salary > 50000
) x;
```

CTE:

```sql
WITH x AS (
    SELECT *
    FROM Employee
    WHERE salary > 50000
)
SELECT *
FROM x;
```

Do not automatically claim that a CTE is faster. The optimizer may inline or materialize it depending on the database.

---

# 55. Window Functions

A window function calculates something across related rows **without collapsing the rows**.

`GROUP BY` typically reduces rows into group-level results. Window functions can preserve each original row.

---

# 56. PARTITION BY

```sql
ROW_NUMBER() OVER (
    PARTITION BY department
    ORDER BY salary DESC
)
```

`PARTITION BY department` divides rows into department groups, and the window calculation runs independently inside each group.

---

# 57. ROW_NUMBER()

```sql
ROW_NUMBER() OVER (
    PARTITION BY department
    ORDER BY salary DESC
)
```

Example:

```
CSE:
90k → 1
80k → 2
70k → 3

ECE:
85k → 1
75k → 2
```

Numbering restarts for each partition.

---

# 58. RANK vs DENSE_RANK vs ROW_NUMBER

Suppose:

```
100
100
90
80
```

## ROW_NUMBER

```
100 → 1
100 → 2
90  → 3
80  → 4
```

## RANK

```
100 → 1
100 → 1
90  → 3
80  → 4
```

## DENSE_RANK

```
100 → 1
100 → 1
90  → 2
80  → 3
```

```
ROW_NUMBER → unique numbering
RANK       → ties + gaps
DENSE_RANK → ties + no gaps
```

---

# 59. Top-N Per Group

Find top 3 earners from every department:

```sql
WITH ranked AS (
    SELECT
        name,
        department,
        salary,
        ROW_NUMBER() OVER (
            PARTITION BY department
            ORDER BY salary DESC
        ) AS rn
    FROM Employee
)
SELECT *
FROM ranked
WHERE rn <= 3;
```

```
PARTITION
↓
ORDER
↓
NUMBER
↓
FILTER
```

---

# 60. Second Highest Salary

Using `MAX`:

```sql
SELECT MAX(salary)
FROM Employee
WHERE salary < (
    SELECT MAX(salary)
    FROM Employee
);
```

Using ranking:

```sql
WITH ranked AS (
    SELECT
        salary,
        DENSE_RANK() OVER (
            ORDER BY salary DESC
        ) AS rnk
    FROM Employee
)
SELECT salary
FROM ranked
WHERE rnk = 2;
```

---

# 61. Nth Highest Salary

```sql
WITH ranked AS (
    SELECT
        salary,
        DENSE_RANK() OVER (
            ORDER BY salary DESC
        ) AS rnk
    FROM Employee
)
SELECT salary
FROM ranked
WHERE rnk = N;
```

---

# 62. Find Duplicate Values

```sql
SELECT email, COUNT(*)
FROM Users
GROUP BY email
HAVING COUNT(*) > 1;
```

---

# 63. Employees Earning More Than Department Average

```sql
SELECT e.*
FROM Employee e
WHERE salary > (
    SELECT AVG(e2.salary)
    FROM Employee e2
    WHERE e2.department = e.department
);
```

---

# 64. Running Total

```sql
SELECT
    date,
    amount,
    SUM(amount) OVER (
        ORDER BY date
    ) AS running_total
FROM Sales;
```

Example:

```
100 → 100
200 → 300
50  → 350
```

---

# 65. CASE

```sql
SELECT
    name,
    salary,
    CASE
        WHEN salary >= 100000 THEN 'High'
        WHEN salary >= 50000 THEN 'Medium'
        ELSE 'Low'
    END AS salary_band
FROM Employee;
```

---

# 66. NULL

NULL does **not** mean 0, empty string, or false.

Wrong:

```sql
WHERE salary = NULL
```

Correct:

```sql
WHERE salary IS NULL
```

or:

```sql
WHERE salary IS NOT NULL
```

---

# 67. COALESCE

Returns the first non-NULL expression.

```sql
SELECT COALESCE(phone, alternate_phone, 'Not available')
FROM Users;
```

---

# 68. COUNT(*) vs COUNT(column)

`COUNT(*)` counts rows.

`COUNT(column)` counts **non-NULL values** of that column.

---

# 69. UNION vs UNION ALL

## UNION

Combines result sets and removes duplicates.

```sql
SELECT city FROM Customers
UNION
SELECT city FROM Suppliers;
```

## UNION ALL

Keeps duplicates.

```sql
SELECT city FROM Customers
UNION ALL
SELECT city FROM Suppliers;
```

`UNION ALL` is generally cheaper when duplicate elimination is unnecessary.

---

# 70. DELETE vs TRUNCATE vs DROP

## DELETE

Removes selected or all rows.

```sql
DELETE FROM Employee
WHERE department = 'CSE';
```

- supports `WHERE`
- table remains

## TRUNCATE

Removes all rows.

```sql
TRUNCATE TABLE Employee;
```

- no normal row-selection `WHERE`
- table structure remains

## DROP

Removes the database object itself.

```sql
DROP TABLE Employee;
```

```
DELETE   → selected/all data
TRUNCATE → all data
DROP     → table itself
```

Exact transactional behavior of `TRUNCATE` varies by DBMS.

---

# 71. PRIMARY KEY vs UNIQUE

## PRIMARY KEY

Main row identifier.

```
unique
+
NOT NULL
```

Only one primary-key constraint per table, though it may contain multiple columns.

## UNIQUE

Provides another uniqueness constraint.

A table may contain multiple UNIQUE constraints.

NULL behavior for UNIQUE constraints can vary between database systems.

---

# 72. View

A view is a saved query presented like a virtual table.

```sql
CREATE VIEW HighEarners AS
SELECT *
FROM Employee
WHERE salary > 100000;
```

Benefits include:

- abstraction
- reusable queries
- exposing only selected columns/rows
- simplifying complex queries

---

# 73. Transactions in SQL

```sql
BEGIN;

UPDATE Account
SET balance = balance - 500
WHERE id = 1;

UPDATE Account
SET balance = balance + 500
WHERE id = 2;

COMMIT;
```

If something fails:

```sql
ROLLBACK;
```

---

# 74. SAVEPOINT

```sql
BEGIN;

UPDATE A ...;

SAVEPOINT s1;

UPDATE B ...;

ROLLBACK TO s1;

COMMIT;
```

A savepoint allows partial rollback without discarding the entire transaction.

---

# 75. Common SQL Interview Traps

### Trap 1: WHERE with aggregate

Wrong:

```sql
WHERE AVG(salary) > 50000
```

For grouped aggregate filtering:

```sql
HAVING AVG(salary) > 50000
```

### Trap 2: forgetting GROUP BY

```sql
SELECT department, AVG(salary)
FROM Employee;
```

is not valid standard grouping logic because `department` is neither aggregated nor grouped.

Use:

```sql
GROUP BY department
```

### Trap 3: LEFT JOIN accidentally becoming INNER JOIN

```sql
FROM Employee e
LEFT JOIN Department d
ON e.department_id = d.id
WHERE d.location = 'Delhi'
```

Rows with no department produce `NULL` on `d.location`, so the `WHERE` condition eliminates them.

Depending on intended semantics, the condition may belong in the `ON` clause instead.

### Trap 4: `NOT IN` and NULL

`NOT IN` can behave unexpectedly when NULL appears in the subquery.

For many “does not exist” interview questions, `NOT EXISTS` is easier and safer to reason about.

---

# 76. The SQL Patterns You Should Know Cold

1. second / Nth highest salary
2. top N salaries per department
3. find duplicates
4. employees earning above department average
5. customers with no orders
6. GROUP BY + HAVING
7. INNER / LEFT / SELF joins
8. CTE
9. `ROW_NUMBER`, `RANK`, `DENSE_RANK`
10. running total
11. `EXISTS` / `NOT EXISTS`
12. NULL handling
13. `DELETE` vs `TRUNCATE` vs `DROP`
14. composite indexes and leftmost-prefix rule

---

# 77. Final Placement Revision Sheet

Before an interview, you should be able to explain these without notes:

## DBMS fundamentals

```
DBMS vs file system
Keys
Primary vs foreign key
Integrity constraints
ER cardinality
Functional dependency
Closure
1NF / 2NF / 3NF / BCNF
Lossless vs dependency preserving
```

## Transactions

```
ACID
Transaction states
Dirty read
Non-repeatable read
Phantom read
Lost update
Isolation levels
S/X locks
2PL
Deadlock
Conflict serializability
MVCC
```

## Indexing

```
Why indexes help
Why writes become expensive
Primary index
Clustering index
Secondary index
Dense vs sparse
B-tree vs B+ tree
Hash vs B+ tree
Composite indexes
Leftmost-prefix rule
Why optimizer may skip an index
```

## SQL

```
SELECT / WHERE
GROUP BY / HAVING
ORDER BY
Joins
Subqueries
Correlated subqueries
EXISTS
CTE
Window functions
ROW_NUMBER
RANK
DENSE_RANK
Top-N per group
NULL
CASE
Aggregates
UNION / UNION ALL
DELETE / TRUNCATE / DROP
Transactions
```

## What to deprioritize for normal placements

Lower-return topics for most software placement interviews include:

- detailed schedule-counting formulas
- detailed B-tree order calculations
- Thomas Write Rule
- full view-serializability mechanics
- relational calculus notation
- extensive storage-block arithmetic

Study these after the placement-focused material above unless a company specifically tests DBMS academically.