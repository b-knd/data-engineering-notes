# Data Definition Language

**CREATE**
`create` new SQL database
```sql
-- syntax
create database database_name
```
`create` new table
```sql
-- syntax
create table table_name (
	column1 datatype,
	column2 datatype,
	...
);
-- from existing table
create table new_table_name AS
	select column1, column2, ...
	from existing_table_name
	where....;

-- example
create table Persons (
	PersonID int,
	LastName varchar(255),
	FirstName varchar(255),
	Address varchar(255),
	City varchar(255)
);
```

**ALTER**
`alter` to add, delete, or modify columns in existing table

Add column
```sql
-- syntax
alter table table_name
add column_name datatype;

-- example
alter table Customers
add Email varchar(255);
```

Drop column
```sql
-- syntax
alter table table_name
drop column column_name;

-- example
alter table Customers
drop column Email;
```

Rename column
```sql
--syntax
alter table table_name
rename column old_name to new_name;
```

Modify datatype
```sql
-- syntax
alter table table_name
alter column column_name datatype;

-- example
alter table Persons
add DateOfBirth date;

alter table Persons
alter DateOfBirth year;
```

**DROP**
`drop` an existing table in database
```sql
-- syntax
drop table table_name;
```
`truncate` to delete data but not table
```sql
truncate table table_name;
```
