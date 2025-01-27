CASE expression is Similar to if-then-else statement allowing you to conditionally evaluates different values based on specified conditions. 

*Syntax*
```postgresql
CASE
	WHEN condition1 THEN result1
	WHEN condition2 THEN result2
	...
	ELSE result_else
END
```

Ex:
```postgresql
SELECT Name, Age, 
	CASE 
		WHEN Age >= 18 THEN 'Adult' 
		WHEN Age >= 13 THEN 'Teenager' 
		ELSE 'Child' 
	END AS AgeGroup 
FROM Students;
```