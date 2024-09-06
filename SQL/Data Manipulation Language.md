# Data Manipulation Language

**SELECT**
`select` data from database
```sql
-- syntax
select column1, column2, ...
from table_name;

-- example
select CustomerName, City 
from Customers;
```
Use `*` to select all columns in the table

**INSERT**
`insert` new record `into` table
```sql
--- syntax
-- new row
insert into table_name
values (value1, value2, value3,...);

-- specifying columns
insert into table_name (column1, column2, ...)
values (value1, value2, ...);

-- multiple rows
insert into table_name (column1, column2, ...)
values
(value1a, value2a, ...),
(value1b, value2b, ...),
...
```

**UPDATE**
`update` to modify existing records
```sql
-- syntax
update table_name
set column1 = value1, column2 = value2,...
where condition;

-- example
update Customers
set ContactName = 'Alfred Schmidt', City = 'Frankfurt'
where CustomerID = 1;
```

**DELETE**
`delete` existing records in a table
```sql
-- syntax
delete from table_name where condition;
````
