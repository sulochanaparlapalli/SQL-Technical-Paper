# SQL Technical Paper

---

# Table of Contents

1. ACID Properties
2. CAP Theorem
3. Joins
4. Aggregations and Filters in Queries
5. Normalization
6. Indexes
7. Transactions
8. Locking Mechanisms
9. Database Isolation Levels
10. Triggers

---

# 1. ACID Properties

## Definition

**ACID** is a set of four properties that ensures database transactions are executed reliably and that data remains correct even if errors, crashes, or concurrent access occur.

ACID stands for:

* **Atomicity** – All operations succeed or none do.
* **Consistency** – Data always follows defined rules and constraints.
* **Isolation** – Concurrent transactions do not interfere with each other.
* **Durability** – Committed changes are permanently stored.

## Why ACID Matters

ACID is essential in systems where data accuracy is critical, such as:

* Banking systems
* E-commerce applications
* Inventory management
* Healthcare systems

## Real-World Example

When transferring ₹1,000 from one bank account to another:

1. Deduct ₹1,000 from Account A.
2. Add ₹1,000 to Account B.

Both steps must succeed together.

## SQL Example

```sql
BEGIN;
UPDATE accounts SET balance = balance - 1000 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 1000 WHERE account_id = 2;
COMMIT;
```

---

# 2. CAP Theorem

## Definition

The **CAP Theorem** states that a distributed database system can guarantee only two of the following three properties when a network partition occurs:

* **Consistency (C)** – All nodes return the same latest data.
* **Availability (A)** – Every request receives a response.
* **Partition Tolerance (P)** – The system continues operating despite network failures.

## Why CAP Matters

Distributed databases must choose which property to sacrifice during communication failures.

## Real-World Example

If two servers become disconnected:

* Choose **Consistency** → Some requests are rejected until data is synchronized.
* Choose **Availability** → Requests are served even if data may be stale.

## Common Databases

| Model | Prioritizes                               | Examples               |
| ----- | ----------------------------------------- | ---------------------- |
| CP    | Consistency + Partition Tolerance         | HBase, ZooKeeper       |
| AP    | Availability + Partition Tolerance        | Cassandra, DynamoDB    |
| CA    | Consistency + Availability (no partition) | Single-node PostgreSQL |

---

# 3. Joins

## Definition

A **JOIN** is a SQL operation used to combine rows from two or more tables based on a related column, usually a primary key and foreign key relationship.

## Why Joins Matter

Data is typically stored in separate tables to reduce redundancy. Joins allow you to retrieve related data together in a single query.

## Example Tables

### employees

| emp_id | name    | dept_id | manager_id |
| -----: | ------- | ------: | ---------: |
|      1 | Alice   |      10 |       NULL |
|      2 | Bob     |      20 |          1 |
|      3 | Charlie |      10 |          1 |
|      4 | David   |    NULL |          2 |

### departments

| dept_id | dept_name |
| ------: | --------- |
|      10 | HR        |
|      20 | IT        |
|      30 | Finance   |

---

## INNER JOIN

Returns only rows that have matching values in both tables.

```sql
SELECT e.emp_id,
       e.name,
       d.dept_name
FROM employees e
INNER JOIN departments d
ON e.dept_id = d.dept_id;
```

---

## LEFT JOIN (LEFT OUTER JOIN)

Returns all rows from the left table and matching rows from the right table. If no match exists, columns from the right table are `NULL`.

```sql
SELECT e.emp_id,
       e.name,
       d.dept_name
FROM employees e
LEFT JOIN departments d
ON e.dept_id = d.dept_id;
```

---

## RIGHT JOIN (RIGHT OUTER JOIN)

Returns all rows from the right table and matching rows from the left table. If no match exists, columns from the left table are `NULL`.

```sql
SELECT e.emp_id,
       e.name,
       d.dept_name
FROM employees e
RIGHT JOIN departments d
ON e.dept_id = d.dept_id;
```

---

## FULL OUTER JOIN

Returns all rows from both tables. Matching rows are combined; non-matching rows contain `NULL` values.

```sql
SELECT e.emp_id,
       e.name,
       d.dept_name
FROM employees e
FULL OUTER JOIN departments d
ON e.dept_id = d.dept_id;
```

---

## CROSS JOIN

Returns the Cartesian product: every row in the first table combined with every row in the second table.

```sql
SELECT e.name,
       d.dept_name
FROM employees e
CROSS JOIN departments d;
```

> If `employees` has 4 rows and `departments` has 3 rows, the result contains `4 × 3 = 12` rows.

---

## SELF JOIN

Joins a table to itself. Commonly used for hierarchical relationships such as employees and managers.

```sql
SELECT e.name AS employee,
       m.name AS manager
FROM employees e
LEFT JOIN employees m
ON e.manager_id = m.emp_id;
```

---

# 4. Aggregations and Filters in Queries

## Definition

**Aggregation** summarizes data from multiple rows into a single result.

**Filtering** restricts which rows or groups are included in the result.

## Aggregate Functions

* `COUNT()` – Counts rows
* `SUM()` – Adds values
* `AVG()` – Calculates average
* `MIN()` – Finds smallest value
* `MAX()` – Finds largest value

## Example

```sql
SELECT dept_id, AVG(salary) AS avg_salary
FROM employees
GROUP BY dept_id;
```

## WHERE

Filters rows before grouping.

```sql
SELECT * FROM employees
WHERE salary > 50000;
```

## HAVING

Filters groups after aggregation.

```sql
SELECT dept_id, COUNT(*)
FROM employees
GROUP BY dept_id
HAVING COUNT(*) > 5;
```

---

# 5. Normalization

## Definition

**Normalization** is the process of organizing data into well-structured tables to reduce redundancy and improve data integrity.

## Why Normalization Matters

Without normalization, we get problems like:

* Data duplication (redundancy)
* Update anomalies
* Insert anomalies
* Delete anomalies

Normalization solves all these issues.

## Normal Forms

### 1NF (First Normal Form)

Each column contains a single value.

### 2NF (Second Normal Form)

Every non-key column depends on the entire primary key.

### 3NF (Third Normal Form)

Non-key columns depend only on the primary key.

### BCNF (Boyce-Codd Normal Form)

Every determinant is a candidate key.

## Example

Instead of storing customer and order details in one table, split them into:

* `customers`
* `orders`

---

# 6. Indexes

## Definition

An **index** is a data structure that allows the database to find rows faster.

## Why Indexes Matter

Without an index, the database may scan every row. With an index, it can jump directly to matching rows.

## Example

```sql
CREATE INDEX idx_employee_name
ON employees(name);
```

## Types of Indexes

* Single-column index
* Composite index
* Unique index
* Partial index

## Composite Index Example

```sql
CREATE INDEX idx_dept_salary
ON employees(dept_id, salary);
```

---

# 7. Transactions

## Definition

A **transaction** is a group of SQL statements executed as one logical unit of work.

## Commands

* `BEGIN` – Starts a transaction
* `COMMIT` – Saves changes permanently
* `ROLLBACK` – Undoes changes
* `SAVEPOINT` – Creates an intermediate rollback point

## Example

```sql
BEGIN;
UPDATE products SET stock = stock - 1 WHERE product_id = 10;
INSERT INTO orders(customer_id, product_id) VALUES (101, 10);
COMMIT;
```

---

# 8. Locking Mechanisms

## Definition

**Locks** are used by the database to control concurrent access to data and prevent conflicts.

## Types of Locks

### Shared Lock

Allows multiple transactions to read the same data.

### Exclusive Lock

Allows one transaction to modify data while blocking others.

### Row-Level Lock

Locks specific rows.

```sql
SELECT * FROM accounts
WHERE account_id = 1
FOR UPDATE;
```

### Table-Level Lock

Locks an entire table.

```sql
LOCK TABLE employees IN EXCLUSIVE MODE;
```

---

# 9. Database Isolation Levels

## Definition

**Isolation levels** define how and when changes made by one transaction become visible to other transactions.

## Concurrency Problems

* Dirty Read
* Non-Repeatable Read
* Phantom Read

## Isolation Levels

### 1. Read Uncommitted (Lowest Isolation)

Transactions can see uncommitted changes of others.

**Problem:**
- Dirty reads allowed

**Example:**
- Transaction A updates salary but does NOT commit
- Transaction B still reads that updated value (unsafe)

**Not used in critical systems**

### 2. Read Committed (Most commonly used)

Only committed data is visible.

**Prevents:**
- Dirty reads

**Allows:**
- Non-repeatable reads
- Phantom reads

Default in many systems like PostgreSQL

### 3. Repeatable Read

If you read a row once, you will get the same value throughout the transaction.

**Prevents:**
- Dirty reads
- Non-repeatable reads

**Allows:**
- Phantom reads

### 4. Serializable (Highest Isolation)

Strictest level. Transactions execute as if they run one after another (serially).

**Prevents:**
- Dirty reads
- Non-repeatable reads
- Phantom reads

**Most safe but slowest**

## Example

```sql
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

---

# 10. Triggers

## Definition

A **trigger** is a database object that automatically executes a function when a specified event occurs.

Events include:

* `INSERT`
* `UPDATE`
* `DELETE`

## Real-World Example

Automatically updating an `updated_at` timestamp whenever a row is modified.

## Trigger Function

```sql
CREATE OR REPLACE FUNCTION set_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

## Create Trigger

```sql
CREATE TRIGGER trg_set_updated_at
BEFORE UPDATE ON employees
FOR EACH ROW
EXECUTE FUNCTION set_updated_at(); 
```

---

# Quick Summary Table

| Concept          | Purpose                                    |
| ---------------- | ------------------------------------------ |
| ACID             | Ensures reliable transactions              |
| CAP Theorem      | Explains distributed system trade-offs     |
| Joins            | Combines related data from multiple tables |
| Aggregations     | Summarizes data                            |
| Normalization    | Reduces redundancy                         |
| Indexes          | Speeds up queries                          |
| Transactions     | Groups operations into one unit            |
| Locking          | Prevents concurrent conflicts              |
| Isolation Levels | Controls visibility between transactions   |
| Triggers         | Automates actions on table events          |

---

# Conclusion

These concepts form the foundation of database design, SQL query optimization, concurrency control, and distributed systems. A strong understanding of them is essential for software engineering interviews and real-world application development.

---

## References

- PostgreSQL Documentation – https://www.postgresql.org/docs/
