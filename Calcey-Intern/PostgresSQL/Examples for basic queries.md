Sample data `airbnb_listings` denoting rental apartments on Airbnb.
![[Pasted image 20240527142447.png]]

#### Querying the table
1. Get all the columns from a table
```sql
SELECT * FROM airbnb_listings;
```

2. Return the `city` column from the table
```sql
SELECT city FROM airbnb_listings;
```

3. Get the `city` and `year_listed` columns from the table
```sql
SELECT city, year_listed FROM airbnb_listings;
```

4. Get the listing `id`, `city` ordered by the `number_of_rooms` in ascending order.
```sql
SELECT id, city FROM airbnb_listings ORDER BY number_of_trooms ASC;
```
- `ORDER BY` is used to order data on a specific column.
- `ASC` = Ascending order

5. Get the listing `id`, `city`, ordered by the `number_of_rooms` in descending order
```sql
SELECT id, city FROM airbnb_listings ORDER BY number_of_rooms DESC;
```
- `DESC` = Descending order

6. Get the first 5 rows from `airbnb_listings`
```sql
SELECT * FROM airbnb_listings LIMIT 5;
```
- `LIMIT` is used to limit the output rows.

#### Filtering on numeric columns
1. Get a unique list of cities where there are listings.
```sql
SELECT DISTINCT city FROM airbnb_listings;
```
- `DISTINCT` is used to get unique values.

2. Get all the listings where `number_of_rooms` is more or equal to 3.
```sql
SELECT * FROM airbnb_listings WHERE number_of_rooms > = 3;
```

3. Get all the listings where `number_of_rooms` is more than 3.
```sql
SELECT * FROM airbnb_listings WHERE number_of_rooms > 3;
```

4. Get all the listings where `number_of_rooms` is exactly 3.
```sql
SELECT * FROM airbnb_listings WHERE number_of_rooms = 3;
```

5. Get all the listings where `number_of_rooms` is lower or equal to 3.
```sql
SELECT * FROM airbnb_listings WHERE number_of_rooms < = 3;
```

6. Get all the listings where `number_of_rooms` is lower than 3.
```sql
SELECT * FROM airbnb_listings WHERE number_of_rooms < 3;
```

7. Get all the listings with 3 to 6 rooms.
```sql
SELECT * FROM airbnb_listings WHERE number_of_rooms BETWEEN 3 AND 6;
```
- `BETWEEN` & `AND` is used to give two values to filter on.

#### Filtering on text columns
1. Get all the listings that are based in `'Paris'`
```sql
SELECT * FROM airbnb_listings WHERE city='Paris';
```

2. Get the listings based in the `'USA'` and in `‘France’`
```sql
SELECT * FROM airbnb_listings WHERE country IN ('USA', 'France');
```
- `IN` is used to give a list of values.

3. Get all listings where `city` starts with `"j"` and where it does not end with `"t"
```sql
SELECT * FROM airbnb_listings WHERE city LIKE 'j%' and NOT LIKE '%t';
```
- `LIKE` is used to give partial strings to filter. 

#### Filtering on multiple columns
1. Get all the listings in `"Paris"` where `number_of_rooms` is bigger than 3.
```sql
SELECT * FROM airbnb_listings WHERE city='Paris' AND number_Of_rooms>'3';
```

2. Get all the listings in `"Paris"` OR the ones that were listed after 2012.
```sql
SELECT * FROM airbnb_listings WHERE city='Paris' OR year_listed > 2012;
```

#### Filtering on missing data
1. Get all the listings where `number_of_rooms` is missing
```sql
SELECT * FROM airbnb_listings WHERE number_of_room IS NULL;
```
- not `=NULL`, but `IS NULL`

2. Get all the listings where `number_of_rooms` is not missing.
```sql
SELECT * FROM airbnb_listings WHERE number_of_listings IS NOT NULL;
```
#### Simple Aggregations
1. Get the total number of rooms available across all listings
```sql
SELECT SUM(number_of_rooms) FROM airbnb_listings; 
```

2. Get the average number of rooms per listing across all listings.
```sql
SELECT AVG(number_of_rooms) FROM airbnb_listings; 
```

3. Get the listing with the highest number of rooms across all listings.
```sql
SELECT MAX(number_of_rooms) FROM airbnb_listings;
```

4. Get the listing with the lowest number of rooms across all listings.
```sql
SELECT MIN(number_of_rooms) FROM airbnb_listings;
```

#### Grouping, filtering and sorting
1. Get the total number of rooms for each country.
```sql
SELECT country, SUM(number_of_rooms) FROM airbnb_listings GROUP BY country;
```

2. Get the average number of rooms for each country
```sql
SELECT country, AVG(number_of_rooms) FROM airbnb_listings GROUP BY country;
```

3. Get the listing with the maximum number of rooms for each country.
```sql
SELECT country, MAX(number_of_rooms) FROM airbnb_listings GROUP BY country;
```

4. Get the listing with the lowest amount of rooms for each country.
```sql
SELECT country, MIN(number_of_rooms) FROM airbnb_listings GROUP BY country;
```

5. For each country, get the average number of rooms per listing, sorted by ascending order.
```sql
SELECT country, AVG(number_of_rooms) AS avg_rooms FROM airbnb_listings GROUP BY country ORDER BY avg_rooms ASC;
```

6. For Japan and the USA, get the average number of rooms per listing in each country.
```sql
SELECT country, AVG(number_of_rooms) FROM airbnb_listings WHERE country IN ('USA', 'JaPan') GROUP BY country;
```

7. Get the number of listings per country.
```sql
SELECT country, count(id) FROM airbnb_listing GROUP BY country;
```

8. Get all the years where there were more than 100 listings per year.
```sql
SELECT year_listed FROM airbnb_listings GROUP BY year_listed HAVING COUNT(id) > 100;
```
