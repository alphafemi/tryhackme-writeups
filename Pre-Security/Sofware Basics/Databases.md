Databases & SQL Fundamentals
 
### What Is a Database?
 
A **database** is an organized digital store of information — conceptually similar to a notebook, but searchable, sortable, and countable in seconds rather than manually, regardless of how much data it holds.
 
#### The Problem Databases Solve
 
```
Paper Notebook                        Database
───────────────                       ────────
Manual record-keeping            →    Structured, queryable storage
Slow to search/count             →    Instant search, sort, filter
Becomes unmanageable at scale    →    Scales to millions of records
```
 
> **Example:** A café owner tracking orders in a notebook can answer "How many coffees were sold today?" only by manually counting through pages. A database answers the same question instantly, regardless of order volume.
 
---
 
### Tables, Columns, and Rows
 
Data inside a database is organized into **tables**, structured like a spreadsheet.
 
| Element | Description |
|---|---|
| **Column** | Defines a *type* of information (e.g., `drink`, `price`, `time`) |
| **Row** | A single complete record (e.g., one café order) |
| **Table** | The full collection of rows sharing the same columns |
 
#### Example: Café Orders Table
 
| id | drink | price | time |
|---|---|---|---|
| 1 | Coffee | 2.50 | 09:14 |
| 2 | Latte | 3.20 | 09:21 |
| 3 | Tea | 2.00 | 09:30 |
 
- Each **column** stores one category of data (all prices live in the `price` column)
- Each **row** stores one complete order's worth of data
- Adding an order → adds one row
- Removing an order → removes only that row; the rest of the table is unaffected
---
 
### SQL (Structured Query Language)
 
**SQL** is the language used to ask questions of — or "query" — a database. Crucially, a standard query **never modifies data**; it only retrieves and displays it.
 
| Plain-English Question | Becomes a SQL Query |
|---|---|
| "Show me all orders" | `SELECT * FROM Orders;` |
| "Show me only coffee orders" | `SELECT * FROM Orders WHERE drink = 'Coffee';` |
| "Show me the cheapest drink" | `SELECT * FROM Orders ORDER BY price;` |
 
---
 
### Core SQL Keywords
 
| Keyword | Purpose |
|---|---|
| `SELECT` | Specifies which column(s) to retrieve |
| `FROM` | Specifies which table to query |
| `WHERE` | Filters rows based on a condition |
| `ORDER BY` | Sorts the result set by a specified column |
 
---
 
### Worked Examples
 
#### 1. Retrieve All Columns — `SELECT *` / `FROM`
 
The `*` wildcard means "all columns."
 
```sql
SELECT * FROM Orders;
```
 
> Returns every row, with every column, from the `Orders` table.
 
#### 2. Retrieve Specific Columns
 
List only the columns needed, separated by commas.
 
```sql
SELECT drink, price FROM Orders;
```
 
> Returns only the `drink` and `price` columns for every row — other columns (e.g., `id`, `time`) are excluded from the result.
 
#### 3. Filter Rows — `WHERE`
 
`WHERE` restricts results to rows matching a specific condition.
 
```sql
SELECT * FROM Orders WHERE drink = 'Coffee';
```
 
> Returns only rows where the `drink` column equals `'Coffee'`.
 
> **Tip:** Unsure what values exist in a column? Query the reference table directly:
> ```sql
> SELECT * FROM Menu;
> ```
 
#### 4. Sort Results — `ORDER BY`
 
By default, `ORDER BY` sorts in **ascending** order (lowest → highest).
 
```sql
SELECT * FROM Orders ORDER BY price;
```
 
To reverse the sort order (highest → lowest), append `DESC`:
 
```sql
SELECT * FROM Orders ORDER BY price DESC;
```
 
| Modifier | Sort Direction |
|---|---|
| *(none)* | Ascending (default) — lowest to highest |
| `DESC` | Descending — highest to lowest |
| `ASC` | Ascending (explicit) — equivalent to default |
 
#### 5. Combining Filtering and Sorting
 
Most real-world queries combine multiple clauses together.
 
```sql
SELECT * FROM Orders WHERE drink = 'Coffee' ORDER BY price DESC;
```
 
> Returns only `Coffee` orders, sorted from the most expensive to the least expensive.
 
---
 
### SQL Clause Order Reference
 
While clauses can be combined, they generally follow this structural order:
 
```sql
SELECT  column1, column2      -- which columns to return
FROM    table_name            -- which table to query
WHERE   condition             -- which rows to include
ORDER BY column [ASC|DESC];   -- how to sort the results
```
 
| Clause | Required? | Purpose |
|---|---|---|
| `SELECT` | Yes | Defines output columns |
| `FROM` | Yes | Defines the source table |
| `WHERE` | No | Filters rows (omit to include all rows) |
| `ORDER BY` | No | Sorts the result set (omit for unspecified/default order) |
 
---
