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
**##NULL**

###NULL in Snowflake

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
But to make migration easier for teams moving from Oracle to Databricks, they also support NVL.
