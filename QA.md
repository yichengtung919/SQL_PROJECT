What are your risk areas? Identify and describe them.

1. There seems to be duplicated data in the all_sessions_clean table with a lot of missing country and/or city data. 
There may be potential completion and accuracy risk to answer the first question on country and city transaction revenue.
2. To check the answer in Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?


QA Process:
Describe your QA process and include the SQL queries used to execute it.

1.
```sql
--double check if there are still any duplicates
WITH 		duplicated_cte AS
(SELECT		DISTINCT(fullvisitorid) 
FROM		all_sessions_clean
GROUP BY	fullvisitorid
HAVING		COUNT(*)>1)

--compare the transactions with duplicated fullvisitorid on their country and city data 
SELECT		fullvisitorid, country, city_name, total_transaction_rev
FROM		all_sessions_clean
WHERE		fullvisitorid IN (SELECT fullvisitorid FROM duplicated_cte)
ORDER BY	fullvisitorid
```
2.

```sql
--double check a few samples if the end result agrees with the first product in each city_name
SELECT		ac.city_name, 
		p.sku, p.name, 
		SUM(p.ordered_quantity) AS top_selling_product_quantity, 
		RANK() OVER (PARTITION BY ac.city_name ORDER BY SUM(p.ordered_quantity) DESC) AS product_rank
FROM		all_sessions_clean AS ac
JOIN		products AS p
ON		ac.product_sku = p.sku
WHERE		city_name IS NOT NULL
GROUP BY	ac.city_name,p.sku, p.name
ORDER BY	ac.city_name, top_selling_product_quantity DESC
```

