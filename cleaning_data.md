What issues will you address by cleaning the data?
1. NULL values in the primary key and date Duplicated data in all_sessions table- filter out any null value in the secondary key column 
2. Potential duplicated data in all_sessions table based on fullvisitorid as secondary key
3. Null currency code in the all_sessions table- assume that all transactions are done in USD and filled the null values with USD 
4. NULL values and duplicated data in the primary key column sku in products table. -Remove null value in the sku column and identify duplicated data if any 
5. The unit price and revenue in the analytics table needs to be divided by 1,000,000.- added a new column for the correct unit price and revenue



Queries:
Below, provide the SQL queries you used to clean your data.

1. 
```sql
SELECT		* , 
			CASE WHEN currency_code IS NULL THEN USD
			ELSE currency_code 
			END AS currency_code_filled
FROM		all_sessions
WHERE		fullvisitorid IS NOT null 
			AND date is NOT null
```
2.
```sql
-- Identified that there are duplicated data based on fullvisitorid and the potential impact it has on total_transaction_rev and noticed that there is one duplicated transaction with total_transaction_rev. 
SELECT 		*
FROM		all_sessions
WHERE		fullvisitorid IN
(SELECT		fullvisitorid
FROM		all_sessions
GROUP BY	fullvisitorid
HAVING		COUNT(*)>1)
		AND total_transaction_rev IS NOT NULL
ORDER BY	fullvisitorid
```
-- Next step is to take the average of these duplicated transactions so we have unique data in the fullvisitorid column


3.
```sql
SELECT		* , 
			CASE WHEN currency_code IS NULL THEN USD
			ELSE currency_code 
			END AS currency_code_filled
FROM		all_sessions
WHERE		fullvisitorid IS NOT null 
			AND date is NOT null
```
4.
```sql
SELECT		*
FROM		products
WHERE		sku IS NOT NULL 
		AND sku NOT IN 
			(SELECT	sku 
			 FROM	products
			 GROUP BY sku
			 HAVING COUNT(*)>1)
```

5.
```sql
CREATE OR REPLACE VIEW analytics_with_correct_unit_price AS
(SELECT		*, 
 			unit_price/1000000 AS actual_unit_price,
 			revenue/1000000 AS actual_revenue
FROM		analytics)
```
