# Functions

### Aggregate functions
Create set of values and return single value, often used with `group by`

`count` - return number of rows
```sql
select count(*) from table_name;
```

`sum` - return total sum of numeric column
```sql
select sum(column_name) from table_name;
```

`avg` - return average value of numeric column
```sql
select avg(column_name) from table_name;
```

`min` - return minimum value in column
```sql
select min(column_name) from table_name;
```

`max` - return maximum value in column
```Sql
select max(column_name) from table_name;
```

### Scalar Functions
Return single value based on input value

`length` - return length of string
```sql
select length(column_name) from table_name;
```

`round` - round numeric value to specified number of decimal place
```sql
select round(column_name, 2) from table_name;
```

### String Functions
`upper` - converts string to uppercase
```sql
select upper(column_name) from table_name;
```

`lower` - converts string to lowercase
```sql
select lower(column_name) from table_name;
```

`concat` - concatenate two or more strings
```sql
select concat(column1, column2) from table_name;
```

`substring` - extract substring from result string
```sql
select substring(column_name, start_position, length) from table_name;
```

`replace` - replaces occurrence of specified substring with another substring
```Sql
select replace(column_name, 'oldsubstring', 'newsubstring') from table_name;
```

`trim` - remove leading and trailing spaces from string
```sql
select trim(column_name) from table_name;
```

### Date and Time Functions
`now` - return current date and time
```Sql
select now();
```

`curdate` - return current date
```sql
select curdate();
```

`datediff` - return difference between two dates
```sql
select datediff(date1, date2) from table_name;
```

`date_format` - format date value according to specified format
```sql
select date_format(date_column, '%Y-%m-%d') from table_name;
```

### Numeric Functions
`abs` - return absolute value of number
```sql
select abs(column_name) from table_name;
```

`ceil` - return smallest integer greater than or equal to a number
```sql
select ceil(column_name) from table_name;
```

`floor` - return largest integer less than or equal to a number
```sql
select floor(column_name) from table_name;
```

`mod` - return remainder of division operation
```sql
select mod(column_name, divisor) from table_name
```

### Conversion Functions
`cast` 
```sql
select cast(column_name as data_type) from table_name;
```

`convert`
```sql
select convert(column_name, data_type) from table_name;
```
