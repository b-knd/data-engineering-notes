# 5.2 Explain how to work with semi-structured data

## Flatten the nested structured

**FLATTEN**

- Explode semi-structured data into multiple rows
- accept VARIANT, OBJECT, ARRAY
- Return set of fixed columns, most notably VALUE, INDEX, KEY\
- Key parameters
    - INPUT - array or objects to exploed (required)
    - OUTER - if TRUE, return row even if input is empty or null (similar to left join)
    - RECURSIVE - if TRUE, flattens input and all nested children within it
    - MODE - ‘ARRAY’, ‘OBJECT’ or ‘BOTH’ to restrict what types of data are flattened

**LATERAL**

- join keyword that allow function to reference columns from preceding table in the same query

LATERAL FLATTEN

- For every element exploded, result row contains the other columns from the original parent row

```sql
# extract items from JSON column named raw_data in a table called orders
SELECT
	id,
	f.value::string AS item_name
FROM orders,
LATERAL FLATTEN(input => raw_data:items) f;
```

## VARIANT column

- Universal data type designed to store semi-structured data like JSON, Avro, ORC, Parquet, or XML
- Highly flexible because it can hold value of any other Snowflake data type, including arrays and objects, without requiring pre-defined schema
- Limit to 16MB compressed data

**Key features**

- Schema flexibility - load raw data directly into VARIANT column without defining internal structure
- Automatic optimisation - sub-columnarisation to extract common elements from VARIANT data into columnar form for faster querying, making performance for regular data similar to standard relational columns
- Size limit - 128MB compressed
- Native querying - query nested data using SQL-like syntax (dot and bracket notation) or specialised functions

**Best practices**

- Storage - if future usage of semi-structured data is unknown, recommended to store in VARIANT column
- Performance pruning - Dates/Timestamps slower if put in VARIANT column (stored as string) → consider flattening into separate relational columns
- Handling NULLs - Snowflake distinguishes between SQL NULL and JSON NULL (literal “null”)