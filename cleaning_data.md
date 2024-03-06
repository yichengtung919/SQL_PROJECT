What issues will you address by cleaning the data?
1. NULL values in the secondary key and date Duplicated data in all_sessions table- filter out any null value in the secondary key column 
2. Potential duplicated data in all_sessions table based on fullvisitorid as secondary key
3. Null currency code in the all_sessions table- assume that all transactions are done in USD and filled the null values with USD
4. NULL values and duplicated data in the primary key column sku in products table. -Remove null value in the sku column and identify duplicated data if any 
5. The unit price and revenue in the analytics table needs to be divided by 1,000,000.- added a new column for the correct unit price and revenue
   
**After consulting with a mentor, I have decided to clean data on a per table basis and implement data cleaning technique discussed.

The queries ran in the question page is still using the prior version table but should be switch to the below table**

6. Data cleaning on all_session table


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

6.
```sql
--All_sessions table
-- STEP 1- Remove NULL values
--Check if there are NULL values in any of the potential primary/foreign keys
SELECT		*
FROM		all_sessions
WHERE		fullvisitorid IS NULL 
			OR visitid IS NULL

--Check if product_refund_amount column is empty
SELECT		*
FROM		all_sessions
WHERE		product_refund_amount IS NOT NULL
--Check if item_quantity column is empty
SELECT		*
FROM		all_sessions
WHERE		item_quantity IS NOT NULL

--Check if item_revenue column is empty
SELECT		*
FROM		all_sessions
WHERE		item_revenue IS NOT NULL

--Check if search_keyword column is empty
SELECT		*
FROM		all_sessions
WHERE		search_keyword IS NOT NULL

--The above columns returned with all NULL values ,
--which means they are not usefull information and can be dropped in the dataset.

--STEP 2- replace country value with correct information base on city
--There are missing values in city and country columns. 
--Assuming if city data is available, the country data can be filled in.
--Create a list of city and corresponding country
CREATE TEMP TABLE city_country AS
(SELECT		DISTINCT(city), country
FROM		all_sessions
WHERE		city NOT IN ('(not set)','not available in demo dataset')
ORDER BY	city)

--Create a list for city that are in multiple countries in the data set
WITH city_in_multiple_countries_cte AS
(SELECT		city 
FROM		city_country
GROUP BY	city
HAVING		COUNT(*)>1)

--investigate what is the correct country for the city
SELECT		*
FROM		city_country
WHERE		city IN (SELECT city FROM city_in_multiple_countries_cte)

--34 rows need to be reviewed and replaced with the correct country data

--STEP 3-identifying duplicates
SELECT		fullvisitorid, time, country, city, visitid, product_sku 
FROM		all_sessions
GROUP BY	fullvisitorid, time, country, city, visitid, product_sku 
HAVING		COUNT(*)>1

--The above query returned with no result which means there are no identical rows in the data set.

--STEP 4- check if all revenue data are in numeric format
SELECT		*
FROM		all_sessions
WHERE		total_transaction_rev IS NOT NULL 
			AND CAST(total_transaction_rev AS TEXT) LIKE '%[^0-9]%'
--the query returns with empty rows which means all revenue data is in double precision(numeric) format

--To combine the above steps and come up with a clean dataset
CREATE OR REPLACE VIEW all_sessions_clean_V2 AS
(SELECT		fullvisitorid, 
			channel_grouping, 	
			time, 	
			city,
			CASE 
        	WHEN city = 'Amsterdam' THEN 'Netherlands'
       		WHEN city = 'Bangkok' THEN 'Thailand'
       		WHEN city = 'Dublin' THEN 'Ireland'
        	WHEN city = 'Hong Kong' THEN 'Hong Kong'
        	WHEN city = 'Istanbul' THEN 'Turkey'
        	WHEN city = 'London' THEN 'United Kingdom'
        	WHEN city = 'Los Angeles' THEN 'United States'
        	WHEN city = 'Mexico City' THEN 'Mexico'
        	WHEN city = 'New York' THEN 'United States'
        	WHEN city = 'Paris' THEN 'France'
        	WHEN city = 'San Francisco' THEN 'United States'
        	WHEN city = 'Singapore' THEN 'Singapore'
        	WHEN city = 'Toronto' THEN 'Canada'
        	WHEN city = 'Vancouver' THEN 'Canada'
        	WHEN city = 'Yokohama' THEN 'Japan'
        	ELSE country
			END AS country,
			total_transaction_rev,
			transactions,
			time_on_site,
			page_views,
			session_quality_dim,
			date,
			visitid,
			type,
			product_quantity,
			product_price,
			product_revenue,
			product_sku,
			v2_product_name,
			v2_product_category,
			product_variant,
			'USD' AS currency_code,
			transaction_revenue,
			transactionid,
			page_title,
			pagepathlevel1,
			ecommerceaction_type,
			ecommerceaction_step,
			ecommerceaction_option
FROM		all_sessions)
```
