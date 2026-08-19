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
~~~~~
-- Grant SELECT privilege on a table to a user/group
GRANT SELECT ON TABLE users TO `analyst`;

-- Grant usage on a catalog
GRANT USAGE ON CATALOG sales TO `analyst`;

-- Revoke privilege
REVOKE SELECT ON TABLE users FROM `analyst`;
~~~~~~~
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
## SUM
Returns the total of a numeric column.
~~~~
SELECT SUM(quantity) AS total_qty FROM orders;
~~~~~
### AVG
Returns the average value of a numeric column.
~~~~
SELECT AVG(unit_price) AS avg_price FROM orders;
~~~~~
### COUNT
Returns the number of rows or non‑null values.
~~~~~~
SELECT COUNT(*) AS total_orders FROM orders;
~~~~~~
### MIN / MAX 
Return the smallest or largest value in a column.
~~~~~~
SELECT MIN(unit_price), MAX(unit_price) FROM orders;
~~~~~~
### LISTAGG
Concatenates values from multiple rows into a single string.
~~~~
SELECT LISTAGG(product_name, ', ') AS product_list FROM products;
~~~~
### MEDIAN 
Returns the middle value of a numeric column.
~~~~~
SELECT MEDIAN(unit_price) AS median_price FROM orders;
~~~~~~
### STDDEV / VARIANCE 
Measure spread of numeric values.
~~~~~
SELECT STDDEV(unit_price), VARIANCE(unit_price) FROM orders;
~~~~~
### APPROX_COUNT_DISTINCT
Returns approximate distinct count for large datasets.
~~~~~
SELECT APPROX_COUNT_DISTINCT(customer_id) AS approx_customers FROM orders;
~~~~~
### PERCENTILE_CONT
Returns a percentile value using continuous distribution.
~~~~~~
SELECT PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY unit_price) AS p90_price FROM orders;
~~~~~~~
### REGR_SLOPE 
Returns slope of regression line between two columns.
~~~~~
SELECT REGR_SLOPE(sales, quantity) AS slope FROM orders;
~~~~~
### KURTOSIS 
Measures peakedness of distribution.
~~~~~
SELECT KURTOSIS(unit_price) AS price_kurtosis FROM orders;
~~~~~
### SKEW 
Measures asymmetry of distribution.
~~~~~~
SELECT SKEW(unit_price) AS price_skew FROM orders;
~~~~~~

## Aggregation in Databricks
Definition: Databricks SQL supports ANSI aggregates plus extensions for big data.

Common functions: SUM, AVG, COUNT, MIN, MAX.

Advanced grouping: GROUPING SETS, CUBE, ROLLUP → multiple aggregations in one query.
### SUM 
Returns the total of a numeric column.
~~~~~~
SELECT SUM(quantity) AS total_qty FROM dealer;
~~~~~~
### AVG 
Returns the average value of a numeric column.
~~~~~~
SELECT AVG(quantity) AS avg_qty FROM dealer;
~~~~~~~
### COUNT
Returns the number of rows or non‑null values.
~~~~~
SELECT COUNT(*) AS total_rows FROM dealer;
~~~~~
### MIN / MAX
Return the smallest or largest value in a column.
~~~~~~
SELECT MIN(quantity), MAX(quantity) FROM dealer;
~~~~~~~
### GROUPING SETS
Generates multiple groupings in one query.
~~~~~~
SELECT city, car_model, SUM(quantity) 
FROM dealer 
GROUP BY GROUPING SETS ((city), (car_model));
~~~~~~
### CUBE
Generates all combinations of groupings for multidimensional analysis.
~~~~~~~
SELECT city, car_model, SUM(quantity) 
FROM dealer 
GROUP BY CUBE(city, car_model);
~~~~~~~
### ROLLUP → 
Generates hierarchical groupings (drill‑down totals).
~~~~~~
SELECT city, car_model, SUM(quantity) 
FROM dealer 
GROUP BY ROLLUP(city, car_model);
~~~~~~
### agg()
Used in Databricks metric views to compute aggregates programmatically.
~~~~~~
SELECT agg('SUM', quantity) AS total_qty FROM dealer;
~~~~~~
## Date Functions (Snowflake & Databricks)
Definition → Used to extract, format, and manipulate date/time values.
~~~~~~~
-- Current date
SELECT CURRENT_DATE;

-- Extract year
SELECT YEAR(order_date) FROM orders;

-- Add days
SELECT DATEADD(day, 7, order_date) FROM orders;
~~~~~~~
👉 “Date functions help in extracting, formatting, and manipulating date/time values like YEAR, MONTH, DATEADD, and CURRENT_DATE.”

## Analytics Functions (Snowflake & Databricks)
Definition → Used for advanced analysis like ranking, windowing, and statistical calculations.
~~~~~~
-- Row number per partition
SELECT ROW_NUMBER() OVER (PARTITION BY city ORDER BY sales) AS row_num
FROM orders;

-- Running total
SELECT SUM(sales) OVER (ORDER BY order_date) AS running_total
FROM orders;

-- Rank
SELECT RANK() OVER (ORDER BY sales DESC) AS sales_rank
FROM orders;
~~~~~
👉 “Analytics functions provide ranking, windowing, and statistical analysis using ROW_NUMBER, RANK, SUM OVER, and similar functions.”

## Character Manipulation Functions (Snowflake & Databricks)
Definition → Used to modify, extract, and format string values.
~~~~~
-- Convert to upper case
SELECT UPPER(name) FROM users;

-- Substring
SELECT SUBSTRING(name, 1, 3) FROM users;

-- Concatenate
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM users;

-- Trim spaces
SELECT TRIM('   Thanigai   ') AS clean_name;
~~~~~~
👉 “Character manipulation functions handle string operations like UPPER, LOWER, SUBSTRING, CONCAT, and TRIM.”
## PL/SQL vs SQL
### SQL
Declarative language used to query and manipulate data.  
~~~~~
SELECT name, salary FROM employees WHERE department = 'HR';
~~~~~~~
### PL/SQL
Procedural extension of SQL (Oracle) that supports loops, conditions, variables, exceptions, cursors, and stored procedures.  ~~
DECLARE v_total NUMBER;
BEGIN
  SELECT SUM(salary) INTO v_total FROM employees;
  IF v_total > 100000 THEN
    DBMS_OUTPUT.PUT_LINE('High Budget');
  ELSE
    DBMS_OUTPUT.PUT_LINE('Normal Budget');
  END IF;
END;
~~
### Snowflake Equivalent pl\sql

**Snowflake does not support PL/SQL directly, but it provides:
Snowflake Scripting
JavaScript Stored Procedures**

Snowflake Scripting → PL/SQL‑like procedural blocks (DECLARE, BEGIN…END, IF, FOR, WHILE).
JavaScript Stored Procedures → procedural logic written in JavaScript.
~~~~
DECLARE v_total NUMBER;
BEGIN
  SELECT SUM(quantity) INTO v_total FROM orders;
  IF v_total > 1000 THEN
    RETURN 'High Volume';
  ELSE
    RETURN 'Normal Volume';
  END IF;
END;
~~~~~
**Snowflake replaces PL/SQL with Snowflake Scripting and JavaScript stored procedures, supporting procedural logic inside the warehouse**
**Exceptions**

In Snowflake Scripting, you can handle exceptions using EXCEPTION blocks, similar to PL/SQL.
~~~~~~
BEGIN
  LET v_total INT;
  SELECT SUM(quantity) INTO v_total FROM orders;
EXCEPTION
  WHEN OTHER THEN
    RETURN 'Error occurred';
END;
~~~~~~
👉 Interview point: “Snowflake Scripting supports exception handling blocks, making it close to PL/SQL.”
**Cursors**
Snowflake Scripting supports explicit cursors to iterate over query results.
~~~~~
DECLARE cur CURSOR FOR SELECT order_id, quantity FROM orders;
FOR rec IN cur DO
  LET v_msg STRING := 'Order ' || rec.order_id || ' Qty ' || rec.quantity;
END FOR;
~~~~~
👉 Interview point: “Snowflake allows cursors inside scripting blocks to loop through query results.”

### Databricks Equivalent
Databricks does not support PL/SQL or stored procedures.
Instead, it uses:
+PySpark / Scala → procedural programming for workflows.
+SQL → declarative queries.
+UDFs (User Defined Functions) → custom logic in Python/Scala.
+Notebooks → workflow orchestration.
~~~~~
from pyspark.sql.functions import col, when

df = orders.withColumn(
    "order_type",
    when(col("quantity") > 10, "Bulk Order").otherwise("Regular Order")
)
~~~~~
**“Databricks doesn’t support PL/SQL; instead, you write procedural logic in PySpark/Scala notebooks or define UDFs for custom functions.”**

**Exceptions**
Databricks SQL does not support exception handling like PL/SQL.
Error handling is done in PySpark/Scala using try…except (Python) or try…catch (Scala).
Example (PySpark):
~~~~~
python
try:
    df = spark.sql("SELECT * FROM orders")
except Exception as e:
    print("Error occurred:", e)
~~~~~
👉 Interview point: “Databricks handles exceptions at the programming language level (Python/Scala), not inside SQL.”
**Cursors**
Databricks SQL does not support cursors.
Iteration is done using DataFrame APIs in PySpark/Scala.
Example (PySpark):

python
~~
for row in df.collect():
    print(f"Order {row.order_id}, Qty {row.quantity}")
~~
👉 Interview point: “Databricks replaces cursors with DataFrame iteration in PySpark/Scala.”
## Snowflake vs Databricks (PL/SQL Equivalent)

| Feature              | **Snowflake**                                   | **Databricks**                                   |
|-----------------------|------------------------------------------------|--------------------------------------------------|
| **Language Type**    | Declarative + Procedural (Snowflake Scripting) | Declarative SQL + Procedural PySpark/Scala       |
| **Procedural Blocks**| Supported via Snowflake Scripting (`IF`, `LOOP`, `DECLARE`) | Not supported in SQL; use notebooks/UDFs         |
| **Stored Procedures**| JavaScript & Snowflake Scripting                | Not supported; use notebook workflows            |
| **UDFs**             | Supported (JavaScript, SQL UDFs)                | Supported (Python/Scala UDFs)                    |
## PL/SQL vs SQL in Snowflake & Databricks
+SQL → Declarative language used to query and manipulate data.
+PL/SQL → Procedural extension of SQL that adds loops, conditions, variables, exceptions, cursors, and stored procedures.
### Snowflake
SQL → Fully supported (ANSI SQL).
PL/SQL Equivalent → Not supported directly, but Snowflake provides:
Snowflake Scripting → PL/SQL‑like procedural blocks (DECLARE, BEGIN…END, IF, FOR, WHILE).
JavaScript Stored Procedures → procedural logic written in JavaScript.
### Databricks
SQL → Fully supported (ANSI SQL for queries).
PL/SQL Equivalent → Not supported. Instead, Databricks uses:
PySpark / Scala → procedural programming for workflows.
UDFs (User Defined Functions) → custom logic in Python/Scala.
Notebooks & Jobs → workflow orchestration instead of stored procedures.
## Functions in Snowflake vs Databricks
### Snowflake
+Create Function → Yes, you can create UDFs (User Defined Functions) in SQL or JavaScript.
~~~~
CREATE OR REPLACE FUNCTION add_tax(price FLOAT)
RETURNS FLOAT
LANGUAGE SQL
AS (price * 1.18);
~~~~
+Alter Function 
Supported with ALTER FUNCTION (to rename, change properties).
~
ALTER FUNCTION add_tax RENAME TO apply_tax;
~
+Recursion 
Not supported in SQL UDFs, but possible in JavaScript stored procedures.
+Drop Function → Supported with DROP FUNCTION.
~~
DROP FUNCTION add_tax(FLOAT);
~~
**👉 “Snowflake supports SQL and JavaScript UDFs. You can create, alter, and drop them. Recursion is only possible in JavaScript stored procedures, not SQL UDFs.”**

### Databricks
+Create Function → Yes, you can create UDFs in Python/Scala or SQL.
~~~~
python
from pyspark.sql.functions import udf
from pyspark.sql.types import FloatType

def add_tax(price):
    return price * 1.18

add_tax_udf = udf(add_tax, FloatType())
spark.udf.register("add_tax", add_tax_udf)
~~~~~
+Alter Function 
Not directly supported; you usually re‑register the UDF with new logic.

+Recursion 
Not supported in SQL UDFs; recursion can be done in Python/Scala functions.

+Drop Function 
No direct DROP FUNCTION; you unregister or overwrite UDFs.
**👉 Databricks supports UDFs in Python/Scala and SQL. Functions can be created and overwritten, but there’s no native ALTER or DROP syntax like Snowflake. Recursion is handled at the programming language level**

## difference between Stored Procedures and Functions in Snowflake and Databricks 
### Snowflake
+Stored Procedures → Used for complex workflows with multiple SQL statements, loops, cursors, and exception handling (JavaScript or Snowflake Scripting).

+Functions (UDFs) → Reusable logic that takes inputs and returns a single value, mainly for calculations or transformations.
### Databricks
+Stored Procedures → Not supported; workflows are implemented using notebooks, Jobs, or PySpark/Scala scripts.

+Functions (UDFs) → Custom logic written in Python/Scala and registered for SQL use, returning a single value.
## snowflake in trigger instead the stream and task
### Snowflake Streams
+Definition → A stream is a change data capture (CDC) object that records inserts, updates, and deletes on a table.

+Purpose → Lets you query only the delta changes since the last consumption, instead of scanning the whole table.
~~~
CREATE OR REPLACE TABLE members (
  id NUMBER, name STRING, fee NUMBER
);

-- Create a stream to track changes
CREATE OR REPLACE STREAM member_stream ON TABLE members;

-- Insert data
INSERT INTO members VALUES (1,'Joe',0), (2,'Jane',0);

-- Stream shows only new changes
SELECT * FROM member_stream;
~~~
👉 Interview point: “Streams capture row‑level changes for downstream processing, replacing the need for triggers.” 

### Snowflake Tasks
+Definition → A task is a scheduled unit of work that runs SQL statements or stored procedures.
+Purpose → Automates transformations, merges, or data movement based on time or stream conditions.
~~~~~
CREATE OR REPLACE TASK process_new_members
WAREHOUSE = mywh
SCHEDULE = '1 minute'
WHEN SYSTEM$STREAM_HAS_DATA('member_stream')
AS
  MERGE INTO members_target t
  USING member_stream s
  ON t.id = s.id
  WHEN MATCHED THEN UPDATE SET t.fee = s.fee
  WHEN NOT MATCHED THEN INSERT (id, name, fee) VALUES (s.id, s.name, s.fee);
~~~~~
👉 Interview point: “Tasks automate SQL execution on a schedule or when streams detect changes.” 
## Alternatives to Triggers in Databricks
### Delta Live Tables

Declarative pipelines that automatically process new data as it arrives.

Acts like a trigger by continuously applying transformations when source data changes.
👉 Interview point: “Delta Live Tables provide continuous ETL pipelines, replacing trigger‑style logic.”

### Structured Streaming

Real‑time streaming engine in Spark.

Processes events (like inserts/updates) as they happen.
👉 Interview point: “Structured Streaming handles event‑driven logic in real time, similar to triggers but at scale.”

### Notebook Workflows

Orchestrate jobs and tasks across notebooks.

Used for scheduled or conditional execution.
👉 Interview point: “Notebook workflows replace stored procedures and triggers with scheduled jobs.”
## Packages in Snowflake vs Databricks
### Snowflake
“Snowflake does not support PL/SQL‑style packages. Instead, procedural logic is grouped into stored procedures and orchestrated with tasks.”
### Databricks 
“Databricks does not support database packages. Instead, you modularize logic using Python/Scala packages, UDFs, and orchestrate workflows with notebooks or Jobs.”

## SQL Loader Equivalent
### Snowflake
SQLLoader* → ❌ Not available.

Instead Options →

Snowpipe → Continuous data ingestion service that auto‑loads files from cloud storage (S3, Azure Blob, GCS).

COPY INTO → Bulk load command to ingest data from staged files into tables.

External Tables → Query data directly from cloud storage without loading.
👉 Interview point: “Snowflake replaces SQL*Loader with COPY INTO for bulk loads and Snowpipe for continuous ingestion.”

### Databricks
SQLLoader* → ❌ Not available.

Instead Options →

Auto Loader → Incrementally and automatically loads new files from cloud storage into Delta tables.

COPY INTO → SQL command to load data from files into Delta tables.

Structured Streaming → Real‑time ingestion for event‑driven pipelines.
👉 Interview point: “Databricks replaces SQL*Loader with Auto Loader for incremental ingestion and COPY INTO for bulk loads.”
## What is PRAGMA?
In databases like Oracle or PostgreSQL, PRAGMA is a directive or hint to the compiler/runtime (e.g., PRAGMA AUTONOMOUS_TRANSACTION, PRAGMA EXCEPTION_INIT).

It’s used to control behavior at a procedural or system level.

### Snowflake
PRAGMA Support → ❌ Not supported.

Instead Options →
Use Snowflake Scripting for procedural control (DECLARE, BEGIN…END, EXCEPTION).
Use Tasks and Streams for orchestration.

👉 Interview point: “Snowflake does not support PRAGMA directives; procedural control is handled through Snowflake Scripting and stored procedures.”

### Databricks
PRAGMA Support → ❌ Not supported in SQL.

Instead Options →
Use SET/CONFIG commands to control runtime behavior.
Use Spark SQL hints (like /*+ BROADCAST */) for query optimization.
Use Python/Scala code for exception handling and advanced logic.

👉 Interview point: “Databricks does not support PRAGMA; instead, runtime behavior is controlled through Spark configs, SQL hints, and programming logic.”
## What is DBMS_UTILITY?
In Oracle PL/SQL, DBMS_UTILITY is a built‑in package that provides utility functions (e.g., compile schema, analyze objects, get dependencies, format call stacks).
👉 Interview point: “It’s an Oracle‑specific package for database utilities, not available in Snowflake or Databricks.”

### Snowflake
DBMS_UTILITY Support → ❌ Not supported.

Instead Options →
Information Schema Views → query metadata (TABLES, COLUMNS, OBJECT_DEPENDENCIES).
Account Usage Views → monitor queries, objects, and performance.

Snowflake Scripting → procedural utilities (error handling, call stacks).
👉 Interview point: “Snowflake replaces DBMS_UTILITY with metadata views and account usage tables for schema and dependency management.”

### Databricks
DBMS_UTILITY Support → ❌ Not supported.

Instead Options →
Spark Catalog APIs → list tables, databases, functions (spark.catalog.listTables()).
System Tables/Views → query metadata about Delta tables.
Notebook Workflows → orchestration and utility tasks.

👉 Interview point: “Databricks replaces DBMS_UTILITY with Spark Catalog APIs and system tables for metadata and utility operations.”

## What is DBMS_PROFILER?
In Oracle PL/SQL, DBMS_PROFILER is a built‑in package used to profile PL/SQL code execution — measuring performance, identifying bottlenecks, and analyzing execution paths.
👉 Interview point: “It’s Oracle‑specific and not available in Snowflake or Databricks.”

🔹 Snowflake
DBMS_PROFILER Support → ❌ Not supported.

Instead Options →

Query Profile UI → Snowflake provides a visual query profiler in the web UI showing execution plans, stages, and performance metrics.

Query History / Account Usage Views → Monitor query runtime, resource usage, and bottlenecks.
👉 Interview point: “Snowflake replaces DBMS_PROFILER with its Query Profile and Account Usage views for performance analysis.”

🔹 Databricks
DBMS_PROFILER Support → ❌ Not supported.

Instead Options →

Spark UI → Provides detailed profiling of jobs, stages, tasks, and execution DAGs.

Databricks Jobs Dashboard → Monitors job execution times and performance.

Structured Streaming Metrics → For profiling streaming workloads.
👉 Interview point: “Databricks replaces DBMS_PROFILER with Spark UI and Databricks monitoring tools for performance profiling.”
##  Oracle indexes and their equivalents in Snowflake and Databricks
### Snowflake Equivalent
Micro‑partitions
Clustering keys
Search Optimization Service
Hybrid table indexes 
+Micro‑partitions
 data is automatically divided into micro‑partitions (50–500 MB) with metadata, enabling partition pruning to skip              irrelevant data and speed up queries.”
 ~~~~~
CREATE OR REPLACE TABLE SALES (
    SALE_ID INT,
    REGION STRING,
    SALE_DATE DATE,
    AMOUNT NUMBER
);
INSERT INTO SALES VALUES
(1, 'APAC', '2026-07-01', 100),
(2, 'EUROPE', '2026-07-02', 200),
(3, 'US', '2026-07-03', 300),
(4, 'APAC', '2026-08-01', 400);

SELECT REGION, SUM(AMOUNT)
FROM SALES
WHERE SALE_DATE = '2026-07-01'
GROUP BY REGION;
~~~~~~
Snowflake checks metadata:
Partition with min/max SALE_DATE covering 2026-07-01 → scanned.
Other partitions → skipped as irrelevant.

 _ Clustering keys 
 “Clustering keys in Snowflake define how micro‑partitions are organized, improving query performance by guiding metadata pruning.”
_ Search Optimization Service
 “Snowflake’s search optimization service builds hidden indexes to accelerate point‑lookup queries on semi‑structured data like JSON.”
_ Hybrid Tables
👉 “Hybrid tables in Snowflake use row‑based storage with enforced primary keys and indexes, designed for OLTP workloads alongside OLAP.”
### Databricks Index Equivalents
Snowflake has clustering keys + hybrid table indexes, but in Databricks the equivalents are:

+Data Skipping Index  
queries skip irrelevant files by storing min/max statistics for each column in Delta Lake.
same query in snowflake
+Bloom Filter Index  
👉 Probabilistic index for fast point lookups on high‑cardinality columns (like email, customer_id).
Equivalent to Snowflake’s search optimization service.
Definition: A probabilistic data structure that quickly checks if a value might exist in a dataset.

Purpose: Speeds up point lookups (=, IN) on high‑cardinality columns (like customer IDs, emails).

Benefit: Reduces the number of files scanned when searching for specific values.

+Z‑Order Clustering  
👉 Multi‑column ordering technique that co‑locates related values in the same set of files.
Equivalent to Snowflake’s clustering keys.

## OLTP vs OLAP clearly with usage, workflow, and examples

OLTP (Online Transaction Processing)
Purpose: Handles real‑time transactions (insert, update, delete).

Storage: Row‑oriented, normalized schema.

Usage: Banking systems, e‑commerce checkout, airline booking.

Key Traits:

Fast, small queries.

High concurrency.

ACID compliance.

Workflow (Bank Transfer Example)
User Action → Customer initiates transfer.

Transaction Request → SQL sent to OLTP DB.

Validation → Constraints checked (PK, FK).

Execution → Update balances.

Commit/Rollback → Ensure atomicity.

Response → Confirmation to user.
~~~~~
BEGIN TRANSACTION;
UPDATE ACCOUNTS SET BALANCE = BALANCE - 500 WHERE ACCOUNT_ID = 101;
UPDATE ACCOUNTS SET BALANCE = BALANCE + 500 WHERE ACCOUNT_ID = 202;
COMMIT;
~~~~~
OLAP (Online Analytical Processing)
Purpose: Handles complex analytical queries on large datasets.

Storage: Column‑oriented, denormalized schema (star/snowflake).

Usage: Business intelligence dashboards, trend analysis, forecasting.

Key Traits:

Heavy queries (aggregations, joins).

Few concurrent users.

Historical data analysis.

Workflow (Sales Trend Example)
Data Load → ETL/CDC pipeline moves OLTP data into OLAP warehouse.

Query Execution → Analyst runs aggregation query.

Processing → Columnar storage scans only relevant columns.

Result → Dashboard/report generated.
~~~~~
SELECT REGION, MONTH(SALE_DATE), SUM(AMOUNT) AS TOTAL_SALES
FROM SALES
GROUP BY REGION, MONTH(SALE_DATE);
~~~~~

**OLTP systems process fast, small, real‑time transactions with enforced ACID properties, while OLAP systems handle complex analytical queries over large historical datasets. Companies use OLTP for daily operations and OLAP for insights, connected by ETL pipelines**


