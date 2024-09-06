# SQL Cheatsheet

| Keyword    |                                           |
| ---------- | ----------------------------------------- |
| `select`   | retrieve data from one or more table      |
| `create`   | create new table, database, index or view |
| `alter`    | modifies existing database object         |
| `drop`     | delete existing database object           |
| `insert`   | add new records to table                  |
| `update`   | modifies existing records in table        |
| `delete`   | removes records from table                |
| `grant`    | give user access privileges to database   |
| `revoke`   | removes access privileges from user       |
| `begin`    | start a transaction                       |
| `commit`   | save transaction's changes permanently    |
| `rollback` | undoes transaction's change               |
| `where`    | filter records                            |
| `having`   | filter records after aggregate function   |
| `if`       | conditional execution                     |
| `and`/`or` | combine multiple conditions               |
| `not`      | negate condition                          |

**Joins**

| Keywords          |                                                                      |
| ----------------- | -------------------------------------------------------------------- |
| `inner join`      | select records that have matching values in both tables              |
| `left join`       | select records from left table, matched records from right table     |
| `right join`      | select records from right table, and matched records from left table |
| `full outer join` | select records when there is a match in left/right table             |

**Set Operations**

| Keywords    |                                                                                          |
| ----------- | ---------------------------------------------------------------------------------------- |
| `union`     | combines result set of two or more select statements (remove duplicates)                 |
| `union all` | combines result set (include duplicates)                                                 |
| `intersect` | return common records from two select statements                                         |
| `except`    | return records from first select statement that are not found in second select statement |

**Aggregation and Grouping**

| Keywords   |                                                    |
| ---------- | -------------------------------------------------- |
| `group by` | group rows that have same values into summary rows |
| `order by` | sort result set                                    |
| `distinct` | select unique records                              |

**Other keywords**

| Keywords |                                             |
| -------- | ------------------------------------------- |
| `limit`  | limit number of records returned            |
| `offset` | specifies offset of the first row to return |
| `case`   | provides conditional logic in query         |

**Functions**

| Keywords          |                                                                   |
| ----------------- | ----------------------------------------------------------------- |
| `count`           | returns number of rows                                            |
| `sum`             | return total sum of numeric column                                |
| `avg`             | return average value of numeric column                            |
| `min`             | return minimum value in a column                                  |
| `max`             | return maximum value in a column                                  |
| `upper`           | converts string to uppercase                                      |
| `lower`           | converts string to lowercase                                      |
| `length`          | returns length of string                                          |
| `round`           | round numeric value to specified number of decimal place          |
| `concat`          | concatenates two or more strings                                  |
| `substring`       | extracts substring from a string                                  |
| `replace`         | replaces occurrence of specified substring with another substring |
| `trim`            | removes leading and trailing spaces from string                   |
| `now`             | returns current date and time                                     |
| `curdate`         | returns current date                                              |
| `datediff`        | returns difference between two dates                              |
| `date_format`     | formats date value according to a specified format                |
| `abs`             | returns absolute value of a number                                |
| `ceil`            | returns smallest integer greater than or equal to a number        |
| `floor`           | returns largest integer less than or equal to a number            |
| `mod`             | returns remainder of division operation                           |
| `cast`/ `convert` | converts value to a specified data type                           |

 
