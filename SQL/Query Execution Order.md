1. FROM - identify tables to be queried and perform JOIN operations
2. WHERE - filter rows based on specified conditions
3. GROUP BY - group rows that have the same values in specified columns into aggregated date
4. HAVING - filter group based on specified condition
5. SELECT - select columns to be returned in the result set
6. DISTINCT - remove duplicate rows from the result set
7. ORDER BY - sort the result set by specified column
8. LIMIT/OFFSET - limit number of rows returned by the query and skip a specified number of rows

**Example**
```sql
select product_name as pn
from Orders
order by pn;
```

```sql
select product_name as pn
from Orders
order by product_name;
```

Both are correct but ordering based on alias in query 1 is preferred and more commonly used to maintain consistency and readability. The second query is correct but confusing.
