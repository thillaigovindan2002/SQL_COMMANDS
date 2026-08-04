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
