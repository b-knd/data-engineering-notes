# Snowflake SQL Capabilities
- ["SELECT" * "EXCLUDE"](#select-with-exclude)
- ["SELECT" * "RENAME"](#select-with-rename)
- [REPLACE](#replace)
- [MIN_BY & MAX_BY](#min_by-&-max_by)
- [GROUP BY ALL](#group-by-all)
- [ILIKE](#ilike)
- [IF NOT EXISTS, IF EXISTS](#if-exists-if-not-exists)
- [ARRAY_SORT, ARRAY_MIN, ARRAY_MAX](#array_sort-array-min-array-max)
- [BANKER's ROUNDING](#bankers-rounding)
- ["SELECT *" alternative "TABLE"](#table)

## "SELECT" with "EXCLUDE"
To pick all columns except a subset
```sql
SELECT * EXCLUDE (SK, NAME) FROM demo_tables;
```

## "SELECT" with "RENAME"
To rename individual columns in select statement (useful when selecting all but need to rename few columns, will not affect original data in the table)
```
--Renaming one column
SELECT * RENAME ID AS CUST_ID FROM demo_table;

--Renaming more than one columns
SELECT * RENAME (ID AS CUST_ID, SK AS C_SK) FROM demo_table;

--With exclude
SELECT * EXCLUDE ID RENAME (SK AS C_SK) FROM demo_table;
```

## REPLACE
To change values of certain columns within statement (useful when selecting all but need to rename few columns, will not affect the original data in table)
```sql
--changing values of a column to upper cases
SELECT * REPLACE (UPPER(FIRST_NAME) AS FIRST_NAME) FROM demo_table;

--changing values for more than one columns
SELECT * REPLACE (UPPER(FIRST_NAME) AS FIRST_NAME, UPPER(LAST_NAME) AS LAST_NAME) FROM demo_table;
```

## MIN_BY & MAX_BY
To find rows containing minimum or maximum value of the column and returns value of another column in that row. This can remove SELF-JOINS or SUB-QUERY necessity
<p align=center>
  <img src="https://github.com/b-knd/data-engineering-notes/blob/main/Snowflake/media/minby-maxby-sampledata.png">
</p>

To get the latest status of order (max date time)
```sql
--Traditional query
SELECT org.ORDER_ID, ord.STATUS
  FROM demo_table org INNER JOIN
    (SELECT ORDER_ID, MAX(ORDER_DT) AS MAX_ORDER_DT
     FROM demo_table org GROUP BY ORDER_ID
    ) tbl
ON org.ORDER_ID = tbl.ORDER_ID AND org.ORDER_DT = tdl.MAX_ORDER_DT;

--Using max_by
SELECT ORDER_ID, MAX_BY(STATUS, ORDER_DT) FROM demo_table GROUP BY ORDER_ID;
```

## GROUP BY ALL
To avoid listing all columns when we need to do aggregation on multiple columns. 
```sql
SELECT
  O_ORDERSTATUS,
  O_ORDERPRIORITY,
  SUM(O_TOTALPRICE)
FROM SNOWFLAKE_SAMPLE_DATA.TPCH_SF10.ORDERS;
```
_The caveat is that we cannot combine other columns once we wrote GROUP BY ALL, the query will FAIL_

## ILIKE
To select with some patterns from table
- %: to match any sequence of zero or more characters
- -: to match any single character
- case insensitive
```sql
SELECT * ILIKE '%order'
FROM SNOWFLAKE_SAMPLE_DATA.TPCH_SF10.ORDERS
LIMIT 10;

SELECT * ILIKE '%_r'
FROM SNOWFLAKE_SAMPLE_DATA.TPCH_SF10.ORDERS
LIMIT 10;
```
<p align=center>
  <img src="https://github.com/b-knd/data-engineering-notes/blob/main/Snowflake/media/ilike_example.png">
</p>

## IF EXISTS, IF NOT EXISTS
To execute statement only if conditions are met.
```sql
--This will add new column department name if it is not already in demo_table
ALTER TABLE demo_table ADD COLUMN IF NOT EXISTS DEPARTMENT_NAME VARCHAR;

--This will drop column last_name if it exists within demo_table
ALTER TABLE demo_table DROP COLUMN IF EXISTS LAST_NAME;
```

## ARRAY_SORT, ARRAY_MIN, ARRAY_MAX
To process array data. These functions have the potential to consider NULL and give suitable outputs
```sql
--This query would give elements of an input array in a sorted manner.
SELECT ARRAY_SORT(ARRAY_GENERATE_RANGE(0,100,20), FALSE);
   o/p:: [80,60,40,20,0]
SELECT ARRAY_SORT(ARRAY_GENERATE_RANGE(0,100,20), TRUE);
   o/p:: [0,20,40,60,80]

--This query would maximum value in this element even if we have NULLs in it.
SELECT ARRAY_MAX([20, 0, NULL, 10, NULL]);
    o/p:: 20
SELECT ARRAY_MIN([20, 0, NULL, 10, NULL]);
    o/p:: 10
```

## BANKER'S ROUNDING
The multiple option when using ROUND()
```sql
--Normal rounding
SELECT ROUND(2.5, 0);
  o/p:: 3

--HALF_TO_EVEN: round to nearest even number
SELECT ROUND(2.5, 0, 'HALF_TO_EVEN')
  o/p:: 2 (which is the nearest even number as compared to 4)

--HALF_AWAY_FROM_ZERO: for value with fractional part exactly equals to .5, if positive then x+0.5, if negative then x-0.5
SELECT ROUND(2.5, 0, 'HALF_AWAY_FROM_ZERO')
  o/p:: 3
SELECT ROUND(-2.5, 0, 'HALF_AWAY_FROM_ZERO')
  o/p:: -3
```
_0 in the arguments means number of decimals for rounding, 0 means round to integer_

# TABLE
Keyword "TABLE" as an alternative for "SELECT *"
```sql
--conventional
SELECT * FROM demo_table;
TABLE demo_table;
--both give the same output
```
