# STRUCTURE QUERY LANGUAGE 
## dual table
**Snowflake and Databricks, there is no DUAL table like in Oracle. The equivalent way is simply to run a SELECT statement without a FROM clause.**

###Snowflake Equivalent
~~~~
-- Snowflake does not need DUAL
SELECT CURRENT_DATE;
SELECT 2+2;
~~~~~~~~
 If you need multiple rows, you can use TABLE(GENERATOR(...)):
 ~~~~~~~~~
SELECT SEQ4() AS n FROM TABLE(GENERATOR(ROWCOUNT => 5));
~~~~~~~~~~
###Databricks Equivalent
~~~~~~~!
-- Databricks Spark SQL also does not need DUAL
SELECT current_date();
SELECT 2+2;
~~~~~~~~
For generating sequences (like Oracle’s recursive queries), you can use
~~~~~~~~~
SELECT explode(sequence(1,5)) AS n;
~~~~~~~~~
## NULL

### NULL in Snowflake

**Snowflake and Databricks, NULL means a value that is undefined, unassigned, or unknown. It’s not zero, not an empty string, and not false — it simply represents “no value.”**
###NULL in Snowflake and Databricks are same 
Cannot be compared directly (= NULL won’t work, you must use IS NULL).
Functions to handle it:
IFNULL(expr, replacement) → replaces NULL with a given value.
COALESCE(expr1, expr2, ...) → returns the first non‑NULL value
~~~~~~~~~~~
SELECT IFNULL(comm, 0) AS commission FROM employees;
SELECT COALESCE(phone, email, 'No Contact') AS contact FROM users;
~~~~~~~~~~~~
But to make migration easier for teams moving from Oracle to Databricks, they also support NVL

## BLOB, CLOB, LONG

**CLOB = Character Large Object
Used to store very large text data (documents, XML, JSON, etc.)🔹 BLOB
BLOB = Binary Large Object
Used to store large binary data (images, audio, video, PDFs).**
###Snowflake Equivalent
CLOB/NCLOB/LONG → Stored as VARCHAR (UTF‑8 encoded).

Snowflake supports very large VARCHAR (up to 16 MB per cell).

BLOB/RAW → Stored as BINARY.
Binary data is preserved without UTF‑8 conversion.
XMLTYPE → No direct equivalent; usually stored as text (VARCHAR).
Databricks Equivalent
CLOB → Stored as STRING (up to ~50,000 characters).

For larger text, use Delta Lake with partitioning or external storage.
BLOB → Stored as BINARY.
Handles raw bytes (images, PDFs, etc.).
Concatenation/aggregation of large text → Use concat() or collect_list() functions.
##Views
Definition: A view is a saved SQL query that behaves like a table.

Characteristics:

No data storage (results computed on demand).

Always reflects the latest data.

UseTypes in Snowflake:

Standard Views (non‑materialized).

Secure Views (hide underlying query logic for security).ul for simplifying complex queries, enforcing security, or providing consistent interfaces.
##Materialized Views
Definition: A materialized view stores the results of a query physically, like a cached table.

Characteristics:

Faster query performance (results pre‑computed).

Requires storage and compute resources to refresh.

Best for repeated queries on large/complex datasets.
Snowflake:

Maintained automatically in the background.

Refreshes incrementally when base tables change.

Available only in Enterprise Edition
Types in Databricks:

Standard SQL views (virtual tables).
Databricks:

Managed via Unity Catalog.

Can be refreshed manually or on a schedule.

Supports incremental refresh via Lakeflow pipelines
Advantages
Views:

Simple, lightweight, no storage cost.

Always up‑to‑date.

Materialized Views:

Faster query performance.

Ideal for repeated, complex queries.

Reduce compute cost for frequent queries.

🔹 Disadvantages
Views:

Slower for complex queries (recomputed each time).

Materialized Views:

Consume storage.

Require compute resources for refresh.

May lag behind base table until refreshed.

More complex to manage.

**SQL commands are grouped into five categories — DDL, DML, DQL, DCL, and TCL — and both Snowflake and Databricks support them with slight syntax differences. Snowflake emphasizes cloud‑native features like CREATE WAREHOUSE and CLONE, while Databricks focuses on Delta Lake operations and Unity Catalog security**
## DDL (Data Definition Language)
Definition: Commands used to define or change database objects (tables, schemas, views).
CREATE → make new objects.
ALTER → modify existing objects.
DROP → remove objects.
TRUNCATE → quickly delete all rows in a table (faster than DELETE).
~~~~~~~~~~~~
-- Create a table
CREATE TABLE users (id INT, name VARCHAR);

-- Alter table: add column
ALTER TABLE users ADD COLUMN email VARCHAR;

-- Drop table
DROP TABLE users;

-- Truncate table
TRUNCATE TABLE users;
~~~~~~~~~~~~~~
both are same 

## DML Definition
Definition: DML commands are used to manipulate data inside tables (insert, update, delete, merge).
They don’t change the structure of the table — only the contents.
INSERT → add new rows.
UPDATE → modify existing rows.
DELETE → remove rows.
MERGE → upsert (insert or update depending on condition).

~~~~~~~~~
-- Insert
INSERT INTO users (id, name) VALUES (1, 'Thanigai');
-- Update
UPDATE users SET name = 'Thani' WHERE id = 1;
-- Delete
DELETE FROM users WHERE id = 1;
-- Merge (Upsert)
MERGE INTO users u
USING staging s
ON u.id = s.id
WHEN MATCHED THEN UPDATE SET u.name = s.name
WHEN NOT MATCHED THEN INSERT (id, name) VALUES (s.id, s.name);
~~~~~~~~~

## DQL Definition
~~~~~~~~
-- Simple select
SELECT * FROM users;

-- Filtered query
SELECT id, name FROM users WHERE id = 1;

-- Aggregation
SELECT product_id, SUM(amount) AS total_sales
FROM orders
GROUP BY product_id
ORDER BY total_sales DESC;

-- Join
SELECT u.id, u.name, o.amount
FROM users u
JOIN orders o ON u.id = o.user_id;
~~~~~~~~
## DCL Definition
Data Control Language (DCL) → commands used to control access and permissions in the database.
GRANT → give privileges (like SELECT, INSERT, UPDATE).
REVOKE → remove privileges.
These commands don’t change data or structure — they control who can do what.
**Snowflake DCL Examples**
Snowflake uses roles and warehouses for access control.
~~~~~~~
-- Grant SELECT privilege on a table to a role
GRANT SELECT ON TABLE users TO ROLE analyst;

-- Grant usage on a warehouse
GRANT USAGE ON WAREHOUSE my_wh TO ROLE analyst;

-- Revoke privilege
REVOKE SELECT ON TABLE users FROM ROLE analyst;
~~~~~~~~
**databricks in example**
Databricks uses Unity Catalog for fine‑grained permissions.
~~~~~~~~~
-- Grant SELECT privilege on a table to a user/group
GRANT SELECT ON TABLE users TO `analyst`;

-- Grant usage on a catalog
GRANT USAGE ON CATALOG sales TO `analyst`;

-- Revoke privilege
REVOKE SELECT ON TABLE users FROM `analyst`;
~~~~~~~~
## Transaction Control Language
manages transactions in SQL.
**Snowflake → TCL works at the warehouse level
Databricks → TCL works at the Delta Lake level (distributed storage).
Similarity → Both support BEGIN, COMMIT, ROLLBACK.
Limitation → Neither supports SAVEPOINT**
~~~~~~~~~~
BEGIN;
UPDATE users SET name = 'Thani' WHERE id = 1;
DELETE FROM orders WHERE product_id = 10;
ROLLBACK;  -- undo everything
COMMIT;    -- save everything
~~~~~~~~~~~~
## Snowflake JOIN Examples
~~~~~~~~~~
-- Inner Join
SELECT u.id, u.name, o.amount
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- Left Join
SELECT u.id, u.name, o.amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- Full Outer Join
SELECT u.id, u.name, o.amount
FROM users u
FULL OUTER JOIN orders o ON u.id = o.user_id;
~~~~~~~~~~~

## Databricks JOIN Examples
~~~~~~~~
-- Inner Join
SELECT u.id, u.name, o.amount
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- Left Join
SELECT u.id, u.name, o.amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- Semi Join (Databricks specific)
SELECT * FROM users u
WHERE EXISTS (SELECT 1 FROM orders o WHERE u.id = o.user_id);

-- Anti Join (Databricks specific)
SELECT * FROM users u
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE u.id = o.user_id);
~~~~~~~~~~~

| Join Type      | **Snowflake**               | **Databricks**              |
|----------------|-----------------------------|-----------------------------|
| **INNER JOIN** | Supported                   | Supported                   |
| **LEFT JOIN**  | Supported                   | Supported                   |
| **RIGHT JOIN** | Supported                   | Supported                   |
| **FULL OUTER** | Supported                   | Supported                   |
| **CROSS JOIN** | Supported                   | Supported                   |
| **SEMI JOIN**  | Not native (use EXISTS)     | Supported directly          |
| **ANTI JOIN**  | Not native (use NOT EXISTS) | Supported directly          |

## Set Operations

**Set operations combine results from multiple queries.
Main types:
UNION → combines results, removes duplicates.
UNION ALL → combines results, keeps duplicates.
INTERSECT → returns common rows.
EXCEP-- UNION
SELECT id, name FROM users
UNION
SELECT id, name FROM customers;

-- UNION ALL
SELECT id, name FROM users
UNION ALL
SELECT id, name FROM customers;

-- INTERSECT
SELECT id FROM users
INTERSECT
SELECT id FROM customers;

-- EXCEPT (Snowflake uses MINUS keyword)
SELECT id FROM users
MINUS
SELECT id FROM customers;
T / MINUS → returns rows from first query not in second**

## Snowflake and databricks Set Operations
~~~~~~~~~
-- UNION
SELECT id, name FROM users
UNION
SELECT id, name FROM customers;

-- UNION ALL
SELECT id, name FROM users
UNION ALL
SELECT id, name FROM customers;

-- INTERSECT
SELECT id FROM users
INTERSECT
SELECT id FROM customers;

-- EXCEPT (Snowflake uses MINUS keyword)
SELECT id FROM users
MINUS
SELECT id FROM customers;
~~~~~~~~~~
**Snowflake and Databricks both support pseudo/virtual columns, but they differ in naming and usage. Snowflake uses virtual columns defined at table creation, while Databricks relies on system-generated pseudo columns (like _metadata) in Delta tables. Neither stores extra data — values are computed at query time**

## Snowflake Pseudo/Virtual Columns 
**Definition: A virtual column is a computed column defined in the table schema.**
~~~~
CREATE OR REPLACE TABLE orders (
    order_id INT,
    quantity INT,
    unit_price DECIMAL(10,2),
    revenue DECIMAL(12,2) AS (quantity * unit_price) VIRTUAL
);

-- Querying the virtual column
SELECT order_id, revenue FROM orders;
~~~~~~~
##Databricks Pseudo Columns
**Definition: Databricks Delta tables expose pseudo/system columns like _metadata for file-level details.**
Common pseudo columns:

_metadata.file_path → path of the file where the row came from

_metadata.file_size → size of the source file

_metadata.row_index → row number within the file
~~~~~~~
-- Querying pseudo columns in Databricks
SELECT _metadata.file_path, _metadata.file_size, name
FROM users;
~~~~~~~
## Aggregation in Snowflake
Definition: Aggregate functions operate across multiple rows and return a single value.

Common functions: SUM, AVG, COUNT, MIN, MAX, LISTAGG, MEDIAN, STDDEV, VARIANCE.

Advanced functions: APPROX_COUNT_DISTINCT, PERCENTILE_CONT, REGR_SLOPE, KURTOSIS, SKEW.

## Aggregation in Databricks
Definition: Databricks SQL supports ANSI aggregates plus extensions for big data.

Common functions: SUM, AVG, COUNT, MIN, MAX.

Advanced grouping: GROUPING SETS, CUBE, ROLLUP → multiple aggregations in one query.

Special functions: agg() for metric views, approximate functions for large datasets.






