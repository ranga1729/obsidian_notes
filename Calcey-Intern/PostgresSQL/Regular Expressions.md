Regular expressions are used to for pattern matching within SQL queries. 

Syntax,
```sql
SELECT colName FROM tableName WHERE colName REGEXP 'regex_pattern';
```

**Metacharacters,** (Special characters with specific meanings for pattern matching)
1. `.` : Matches any single character
2. `*` : Matches zero or more of the preceding elements.
3. `+` : Matches one or more of the preceding characters.
4. `[]` : Matches any characters within the brackets.
	Ex: `[aeiou]` matches any vowel
5. `^` : Matches the beginning of the string.
6. `$` : Matches the end of the string.
7. `|` : Logical or operator
	Ex: `a|b` Matches either 'a' or 'b'

**Modifiers** : Alter the behavior of the regex
	Ex: 'pattern'i
	`i` for case-insensitive matching. Often place at the end of the pattern.

Ex: 
1. Find names staring with 'A'. (case-sensitive)
```sql
SELECT name FROM stuents WHERE REGEXP '^[A]';
```
2. Find email with '.com'. (case-sensitive)
```sql
SELECT email FROM users WHERE email REGEXP '.*\.com$'
```
3. Find product codes with 5 digits.
```sql
SELECT code FROM products WHERE REGEXP '^\d{5}$'
```
