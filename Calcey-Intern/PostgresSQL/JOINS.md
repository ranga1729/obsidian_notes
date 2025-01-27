Types of joins, 
1. Inner join
2. Left join
3. Right join
4. Full outer join
5. Self join
6. Cross join
7. Natural join

#### 1. Inner Join
When we perform a inner join between tables A and B, it compares each row in Table A to each row in Table B using a join condition. 

Ex: Get the employees name and their department name.
Tables: employees, departments
![[Pasted image 20240528114507.png]]
```postgresql
SELECT employee_name, department_name FROM employees
INNER JOIN departments 
ON employees.deparetment_id = departments.deparetment_id
```
**When tables have columns with common names we need to specify the tables name along with the column name.**
Or we can use **using** clause.
###### using,
This is a alternative clause to **ON** clause.
Ex:
```postgresql
SELECT employee_name, department_name FROM employees
INNER JOIN departments using(deparetment_id);
```
#### Left Join/ Right Join
**Left Join**: It ensures that all rows from the left (first mentioned) table are included in the result set, regardless of whether there's a matching record in the right table.
It also includes rows from the right (second mentioned) table that have matching values in the join condition column(s).
If there's no match found for a row in the left table, the corresponding columns from the right table will be filled with NULL values in the result set.

**Use case**: Left Join is ideal when you want to include all data from the left table, even if there's no corresponding data in the right table. This helps maintain a complete picture of the left table's data. 

Same applies to the right join but prioritizing the right (secondly mentioned) table. 

Ex:
![[Pasted image 20240528154216.png]]
*Left Join*
```postgresql
SELECT employee_name, department_name FORM employee
LEFT JOIN departments ON employees.department_id = departments.department_id;
```
*Right join*
```postgresql
SELECT employee_name, department_name FROM employees
RIGTH JOIN departments ON employees.department_id = departments.department_id;
```
#### Full outer join
Full outer join is a combination of left join and the right join. 
If a row from one table doesn't have a match in the other table based on the join condition, the corresponding columns from the unmatched table will be filled with NULL values in the result set.
![[Pasted image 20240528155633.png]]
Ex:
```postgresql
SELECT employee_name, department_name FROM employees 
FULL OUTER JOIN departments ON employees.department_id = departments.department_id
```

### Self Join
A join between the same table. 
Used in scenarios where we need to compare data in a table with itself. 

**To write the SQL command for a self-join we need a table and column alias.**
An alias is a temporal name given to a table or column for the duration of the query. This alias is used to differentiate between two tables or columns with the same name.  

Ex:
```postgresql
SELECT e1.employee_name, e2.employee_name FROM employees e1
LEFT JOIN employees e2 ON e1.department_id = e2.department_id
AND e1.employee_id ! = e2.employee_id;
```
![[Pasted image 20240528164312.png]]
`AND e1.employee_id ! = e2.employee_id;` is used to prevent from each row get compared by itself.
### Cross Join (Cartesian join)
It's a join between two tables which returns every possible combination of rows between two tables. It results in a much larger dataset compared to other join types like inner join or left join.
It pairs every single row from the left table with every single row from the right table, regardless of any matching criteria.
Unlike other joins that require a specific condition for rows to be matched, Cross Join doesn't have a WHERE clause or any explicit join condition.

Use cases: To generate all possible combinations of data from two tables. For instance, creating a list of all product-category combinations for a product recommendation system.

Ex:
```postgresql
SELECT employee_name, department_name FROM employees CROSS JOIN departments; 
```
OR
```postgresql
SELECT employee_name, department_name FROM employees INNER JOIN departments ON true;
```
![[Pasted image 20240528165136.png]]
### Natural Join
Natural join operations doesn't required explicitly specified join condition. Instead it automatically identifies join tables based on columns with matching names. 
We don't have to mention the join condition here.
Ex:
```postgresql
SELECT employee_name, department_name FROM employees Natural JOIN departments;
```
![[Pasted image 20240528165954.png]]
*Natural join is a inner join by default.*
To use a left/right join with natural join you need to mention the keywords. 
```postgresql
SELECT employee_name, department_name FROM employees Natural LEFT JOIN departments;
```
```postgresql
SELECT employee_name, department_name FROM employees Natural RIGHT JOIN departments;
```
Ex:
![[Pasted image 20240528170616.png]]
