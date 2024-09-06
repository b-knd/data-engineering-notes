# Join and Set Operations

Combine rows from two or more tables, based on a related column between them

**INNER JOIN**
Returns only rows that have matching values in both tables
```sql
SELECT columns
FROM table1
INNER JOIN table2
ON table1.common_column = table2.common_column;
```

**LEFT (OUTER) JOIN**
Returns all rows from left table, and the matched rows from right table. Result is NULL from right side if there is no match.
```sql
select columns
from table1
left join table2
on table1.common_column = table2.common_column;
```

**RIGHT (OUTER) JOIN**
Returns all rows from the right table, and the matched rows from the left. The result is NULL from the left side if there is no match.
```sql
select columns
from table1
right join table2
on table1.common_column = table2.common_column;
```

**FULL (OUTER) JOIN**
Returns all rows when there is a match in one of the tables. If there is no match, the result is NULL from the side that does not have a match.
```sql
select columns
from table1
full outer join table2
on table1.common_column = table2.common_column;
```

**CROSS JOIN**
Returns the Cartesian product of the two tables, i.e., it returns all possible combination of rows.
```sql
select columns
from table1
cross join table2;
```

**SELF JOIN**
A regular join but table is joined with itself
```Sql
select a.columns, b.columns
from table1 a, table1 b
where condition;
```

<p align=center>
  <img width=800 src="https://github.com/b-knd/data-engineering-notes/blob/main/SQL/media/sql_joins.png">
</p>
<p align=center>
  <img width=800 src="https://github.com/b-knd/data-engineering-notes/blob/main/SQL/media/sql_joins_2.jpg">
</p>

Simple example of JOIN
Given table 1 - Orders

| OrderID | CustomerID | OrderDate  |
| ------- | ---------- | ---------- |
| 10308   | 2          | 1996-09-18 |
| 10309   | 37         | 1996-09-19 |
| 10310   | 77         | 1996-09-20 |

Table 2 - Customers

| CustomerID | CustomerName | ContactName | Country |
| ---------- | ------------ | ----------- | ------- |
| 1          | Alfreds      | Maria       | Germany |
| 2          | Ana          | Ana         | Mexico  |
| 3          | Antonio      | Antonio     | Mexico  |

```sql
select o.OrderID, c.CustomerName, o.OrderDate
from Orders o
inner join Customers c on o.CustomerID = c.CustomerID
```

Result table

| OrderID | CustomerName | OrderDate  |
| ------- | ------------ | ---------- |
| 10308   | Ana          | 9/18/1996  |
| 10365   | Antonio      | 11/27/1996 |
| 10383   | Around       | 12/16/1996 |

### Set Operations
**UNION** - retain duplicate values
```Sql
select column_name(s) from table1
union
select column_name(s) from table2;
```

**UNION ALL** - remove duplicate values (but keep original table duplicates)
```Sql
select column_name(s) from table1
union all
select column_name(s) from table2;
```
