1. Query the _Name_ of any student in **STUDENTS** who scored higher than  _Marks_. Order your output by the _last three characters_ of each name. If two or more students both have names ending in the same last three characters (i.e.: Bobby, Robby, etc.), secondary sort them by ascending _ID_.
![[Pasted image 20240528172639.png]]
```sql
SELECT NAME FROM STUDENTS WHERE MARKS > 75
ORDER BY RIGHT(NAME,3) ASC, ID ASC ;
```
- `ORDER BY` clause can work with more than one column. sperate columns with a comma.
- To order by the last three characters of the name, use `RIGHT(name,3)`.
