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
