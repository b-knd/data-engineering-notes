# Window Functions

Analytic functions, perform calculations across a set of table rows that are related to current row. They allow access to individual rows along with their aggregated result.

General syntax:
```sql
select column_name1,
	window_function(column_name2)
	over (partition by column_name1 order by column_name3)
as new_column
from table_name;
```

**ROW_NUMBER()**
Assigns unique number to each row within a partition of a result set
**RANK()**
Assigns rank to each row within a partition of a result set, which gaps in the ranking for ties
**DENSE_RANK()**
Similar to RANK(), but without gaps in the ranking
(Ex: if there is common rank, the next rank will not be skip but numbered consecutively)

```sql
select
	row_number() over (partition by department order by salary desc) as emp_row_no,
	name,
	department,
	salary,
	rank() over (partition by department order by salary desc) as emp_rank,
	dense_rank() over (partition by department order by salary desc) as emp_dense_rank
from employee;
 ```
 ![[Pasted image 20240628110628.png]]

**NTILE(n)**
Distributes the rows in an ordered partition into a specified number of approximately equal groups

**LAG()**
Provides access to a value from a previous row in the same result set without using self-join

**LEAD()**
Provides access to a value from a subsequent row in the same result set without using self-join

**FIRST_VALUE()**
Returns the first value in an ordered partition of a result set

**LAST_VALUE()**
Returns the last value in an ordered partition of a result set

**SUM()**
Calculates sum of a set of values

**AVG()**
Calculates the average of a set of values
```sql
select name, age, department, salary
	avg(salary) over (partition by department) as avg_salary
from employee;
```
![[Pasted image 20240628110305.png]]

```sql
select name, age, department, salary
	avg(salary) over (partition by department order by age) as avg_salary
from employee 
```
![[Pasted image 20240628110412.png]]
