# Common Clauses

### Aggregation and Grouping
**GROUP BY**
Group rows that have the same values into summary rows
```sql
-- syntax
select sum(column_name)
from table_name
where condition
group by table_name;

-- example
select Address, Age, sum(Salary)
as total_salary from Customers
group by Address, Age,
-- this group customers by age and calculate the average salary for each age group
```

**ORDER BY**
Sort result-set in ascending or descending order
```sql
-- syntax
select * from table_name
order by column_name asc/desc;

-- example
select * from Products
order by Price desc;
```
String values will be order alphabetically.
Muti-column order
```sql
select * from table_name
order by column1 asc/desc, column2 asc/desc;

-- if column1 have same value, order by column2
```

**DISTINCT**
Return only different values
```sql
-- syntax
select distinct column1 from table_name;

-- example
select distinct Country from Customers;
```
### Conditional and Control-of Flow
**WHERE**
Filter records
```sql
-- syntax
select * from table_name where condition;

-- example
select * from Person where Age > 30;
```

**HAVING**
Filter record after aggregate function
```sql
-- syntax
select column1, count(*) from table_name group by column1 having count(*) > 1;

-- example
select DepartmentID, count(*) from Employees
group by DepartmentID having count(*) > 10;
```

**IF**
```sql
-- syntax
if (condition) then
	-- sql statements
end if;
```

### Others
**LIMIT**
Limit number of records returned
```sql
-- syntax
select * from table_name limit n;
```

**OFFSET**
Specifies offset of the first row to return
```sql
-- syntax
select * from table_name limit n offset m;
```

**CASE**
Conditional logic in query
```sql
-- syntax
select column1,
	case
		when condition1 then result1
		when condition2 then result2
		else result3
	end
from table_name;
```
