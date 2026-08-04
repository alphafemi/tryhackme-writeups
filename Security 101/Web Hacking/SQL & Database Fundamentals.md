# SQL & Database Fundamentals
 
## Overview
 
---
 
### What is a Database?
 
A database is an **organised collection of structured information** that can be stored, accessed, manipulated, and analysed. Databases underpin virtually every digital service — authentication systems, social media, e-commerce, SIEM tools, malware analysis platforms, and more.
 
---
 
### Relational vs. Non-Relational Databases
 
| Type | Also Known As | Data Format | Best For |
|---|---|---|---|
| **Relational** | SQL | Structured — fixed columns and rows in tables | Consistent data, accuracy matters (e.g., e-commerce transactions) |
| **Non-relational** | NoSQL | Non-tabular — documents, key-value pairs, etc. | Variable/flexible data formats (e.g., social media content) |
 
**Non-relational example (document format):**
```json
{
  "_id": ObjectId("4556712cd2b2397ce1b47661"),
  "name": { "first": "Thomas", "last": "Anderson" },
  "date_of_birth": "1964-09-02",
  "occupation": ["The One"],
  "steps_taken": 4738947387743977493
}
```
 
---
 
### Relational Database Structure
 
#### Tables, Rows, and Columns
 
| Component | Description |
|---|---|
| **Table** | A named collection of related records (e.g., `books`, `authors`) |
| **Column** | A defined field type within the table (e.g., `name`, `published_date`) |
| **Row** | A single record in the table — one complete data entry |
 
> Each column has a defined **data type** — inserting the wrong type is rejected by the database.
 
**Common data types:**
 
| Type | Description |
|---|---|
| `INT` | Whole numbers |
| `VARCHAR(n)` | Variable-length text up to `n` characters |
| `DATE` | Date values |
| `FLOAT` / `DECIMAL` | Numbers with decimal points |
| `BOOLEAN` | True/false |
 
#### Primary Keys & Foreign Keys
 
| Key | Description | Uniqueness |
|---|---|---|
| **Primary Key** | Uniquely identifies each row in a table | Must be unique per table; only one per table |
| **Foreign Key** | A column in one table that references the primary key of another — creates a relationship | Can appear multiple times; multiple per table |
 
```
Books table                    Authors table
┌────────────────────┐         ┌──────────────────┐
│ book_id (PK)       │         │ author_id (PK)   │
│ book_name          │         │ author_name      │
│ author_id (FK) ────┼────────▶│                  │
└────────────────────┘         └──────────────────┘
```
 
---
 
### SQL (Structured Query Language)
 
SQL is the standard language for interacting with relational databases — querying, creating, updating, and deleting data. It is **plain-English in style** and widely used across:
- Offensive security (SQL injection exploitation)
- Defensive security (SIEM queries, log investigation, access control)
**DBMS examples:** MySQL, PostgreSQL, MariaDB, Oracle Database, Microsoft SQL Server
 
---
 
### Database Statements
 
```sql
-- Create a new database
CREATE DATABASE thm_bookmarket_db;
 
-- List all databases
SHOW DATABASES;
 
-- Switch to a database
USE thm_bookmarket_db;
 
-- Delete a database
DROP DATABASE thm_bookmarket_db;
```
 
---
 
### Table Statements
 
```sql
-- Create a table
CREATE TABLE book_inventory (
    book_id INT AUTO_INCREMENT PRIMARY KEY,
    book_name VARCHAR(255) NOT NULL,
    publication_date DATE,
    page_count INT
);
 
-- List tables in current database
SHOW TABLES;
 
-- Describe table structure
DESCRIBE book_inventory;
 
-- Add a column
ALTER TABLE book_inventory ADD page_count INT;
 
-- Delete a table
DROP TABLE book_inventory;
```
 
**`DESCRIBE` output example:**
```
+------------------+--------------+------+-----+---------+----------------+
| Field            | Type         | Null | Key | Default | Extra          |
+------------------+--------------+------+-----+---------+----------------+
| book_id          | int          | NO   | PRI | NULL    | auto_increment |
| book_name        | varchar(255) | NO   |     | NULL    |                |
| publication_date | date         | YES  |     | NULL    |                |
+------------------+--------------+------+-----+---------+----------------+
```
 
---
 
### CRUD Operations
 
| Operation | SQL Statement | Purpose |
|---|---|---|
| **Create** | `INSERT INTO` | Add a new record |
| **Read** | `SELECT` | Retrieve records |
| **Update** | `UPDATE` | Modify existing records |
| **Delete** | `DELETE` | Remove records |
 
#### INSERT
 
```sql
INSERT INTO books (id, name, published_date, description)
VALUES (1, "Android Security Internals", "2014-10-14", "An In-Depth Guide to Android's Security Architecture");
```
 
#### SELECT
 
```sql
-- All columns
SELECT * FROM books;
 
-- Specific columns
SELECT name, description FROM books;
```
 
#### UPDATE
 
```sql
UPDATE books
SET description = "An In-Depth Guide to Android's Security Architecture."
WHERE id = 1;
```
 
#### DELETE
 
```sql
DELETE FROM books WHERE id = 1;
```
 
> ⚠️ Always use `WHERE` with `UPDATE` and `DELETE` — omitting it affects **every row** in the table.
 
---
 
### Clauses
 
Clauses filter, sort, and group query results.
 
#### WHERE
 
Filters rows based on a condition:
 
```sql
SELECT * FROM books WHERE category = "Offensive Security";
```
 
#### DISTINCT
 
Returns only unique values — removes duplicates:
 
```sql
SELECT DISTINCT name FROM books;
```
 
#### ORDER BY
 
Sorts results ascending (`ASC`, default) or descending (`DESC`):
 
```sql
SELECT * FROM books ORDER BY published_date ASC;
SELECT * FROM books ORDER BY published_date DESC;
```
 
#### GROUP BY
 
Groups rows that share a value, used with aggregate functions:
 
```sql
SELECT name, COUNT(*)
FROM books
GROUP BY name;
```
 
#### HAVING
 
Filters grouped results (used after `GROUP BY`, unlike `WHERE` which filters before aggregation):
 
```sql
SELECT name, COUNT(*)
FROM books
GROUP BY name
HAVING name LIKE '%Hack%';
```
 
---
 
### Operators
 
#### Logical Operators
 
| Operator | Behavior | Example |
|---|---|---|
| `LIKE` | Pattern match using `%` (any chars) or `_` (one char) | `WHERE name LIKE '%Security%'` |
| `AND` | All conditions must be true | `WHERE category = "X" AND name = "Y"` |
| `OR` | At least one condition must be true | `WHERE name LIKE "%Android%" OR name LIKE "%iOS%"` |
| `NOT` | Reverses the condition | `WHERE NOT description LIKE "%guide%"` |
| `BETWEEN` | Value within a range (inclusive) | `WHERE id BETWEEN 2 AND 4` |
 
#### Comparison Operators
 
| Operator | Meaning | Example |
|---|---|---|
| `=` | Equal to | `WHERE name = "Ethical Hacking"` |
| `!=` | Not equal to | `WHERE category != "Offensive Security"` |
| `<` | Less than | `WHERE published_date < "2020-01-01"` |
| `>` | Greater than | `WHERE published_date > "2020-01-01"` |
| `<=` | Less than or equal to | `WHERE published_date <= "2021-11-15"` |
| `>=` | Greater than or equal to | `WHERE published_date >= "2021-11-02"` |
 
---
 
### Functions
 
#### String Functions
 
| Function | Purpose | Example |
|---|---|---|
| `CONCAT(a, b, ...)` | Join strings together | `SELECT CONCAT(name, " — ", category) FROM books;` |
| `GROUP_CONCAT(col SEPARATOR ", ")` | Concatenate multiple rows into one string | Used with `GROUP BY` |
| `SUBSTRING(col, start, length)` | Extract part of a string | `SUBSTRING(published_date, 1, 4)` → year only |
| `LENGTH(col)` | Count characters in a string | `SELECT LENGTH(name) FROM books;` |
 
#### Aggregate Functions
 
| Function | Purpose | Example |
|---|---|---|
| `COUNT(*)` | Number of rows matching query | `SELECT COUNT(*) FROM books;` |
| `SUM(col)` | Total of all values in column | `SELECT SUM(price) FROM books;` |
| `MAX(col)` | Highest value in column | `SELECT MAX(published_date) FROM books;` |
| `MIN(col)` | Lowest value in column | `SELECT MIN(published_date) FROM books;` |
 
---
 
### MySQL Quick Start
 
```bash
# Connect to MySQL as root
mysql -u root -p
 
# Enter password when prompted
```
 
```sql
-- Common startup sequence
CREATE DATABASE mydb;
USE mydb;
SHOW TABLES;
```
 
---
 
### SQL Quick Reference
 
| Task | Statement |
|---|---|
| Create database | `CREATE DATABASE name;` |
| List databases | `SHOW DATABASES;` |
| Select database | `USE name;` |
| Drop database | `DROP DATABASE name;` |
| Create table | `CREATE TABLE name (col type, ...);` |
| List tables | `SHOW TABLES;` |
| Describe table | `DESCRIBE tablename;` |
| Add column | `ALTER TABLE name ADD col type;` |
| Drop table | `DROP TABLE name;` |
| Insert record | `INSERT INTO table (cols) VALUES (vals);` |
| Read all | `SELECT * FROM table;` |
| Read filtered | `SELECT * FROM table WHERE condition;` |
| Update record | `UPDATE table SET col=val WHERE condition;` |
| Delete record | `DELETE FROM table WHERE condition;` |
| Sort results | `ORDER BY col ASC\|DESC` |
| Remove duplicates | `SELECT DISTINCT col FROM table;` |
| Group results | `GROUP BY col` |
| Filter groups | `HAVING condition` |
| Count rows | `SELECT COUNT(*) FROM table;` |
| Find max/min | `SELECT MAX(col), MIN(col) FROM table;` |
 
---
