Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**


SQL Queries:

```sql

--filled in the currency_code and make '(not set)'and 'not available in demo dataset' in the city column NULL
--filtered out any fullvisitorid and date column that may potential be NULL
CREATE OR REPLACE VIEW	all_sessions_clean AS
(SELECT		* , 
			CASE WHEN currency_code IS NULL THEN 'USD'
			ELSE currency_code 
			END AS currency_code_filled,
 			CASE WHEN city = '(not set)' THEN NULL
 			WHEN city = 'not available in demo dataset' THEN NULL
 			ELSE city
 			END AS city_name,
 			product_price/1000000 AS actual_product_price
FROM		all_sessions
WHERE		fullvisitorid IS NOT null 
			AND date is NOT null);

--isolate transaction data and create column fullvisitorid to be the primary key
CREATE OR REPLACE VIEW transaction_rev AS
(SELECT		fullvisitorid, AVG(total_transaction_rev) AS adj_total_transaction_rev
FROM		all_sessions_clean
GROUP BY	fullvisitorid
ORDER BY	fullvisitorid)

			
--transaction rev by city
SELECT		ac.city_name, SUM(adj_total_transaction_rev) AS transaction_rev_by_city
FROM		all_sessions_clean AS ac
JOIN		transaction_rev AS tr
ON		ac.fullvisitorid = tr.fullvisitorid
WHERE		ac.city_name IS NOT NULL
GROUP BY	ac.city_name
HAVING		SUM(adj_total_transaction_rev) IS NOT NULL
ORDER BY	SUM(adj_total_transaction_rev) DESC

			
--transaction rev by country
SELECT		ac.country, SUM(adj_total_transaction_rev) AS transaction_rev_by_country
FROM		all_sessions_clean AS ac
JOIN		transaction_rev AS tr
ON			ac.fullvisitorid = tr.fullvisitorid
WHERE		ac.country IS NOT NULL
GROUP BY	ac.country
HAVING		SUM(adj_total_transaction_rev) IS NOT NULL
ORDER BY	SUM(adj_total_transaction_rev) DESC


```


Answer: 
Top 5 Cities- San Francisco, Atlanta, Sunnyvale, Palo Alto, Tel Aviv-Yafo
Top 5 Countries- United States, Israel, Australia, Canada, Switzerland 




**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:
```sql
--average number of products ordered from each city
SELECT		city_name, AVG (p.ordered_quantity) AS avg_num_product
FROM		all_sessions_clean AS ac
JOIN		products AS p
ON		ac.product_sku = p.sku		
WHERE		city_name IS NOT NULL 
GROUP BY	city_name
ORDER BY	avg_num_product DESC

--avg product ordered from each country
SELECT		country, AVG (p.ordered_quantity) AS avg_num_product
FROM		all_sessions_clean AS ac
JOIN		products AS p
ON			ac.product_sku = p.sku		
WHERE		country IS NOT NULL 
GROUP BY	country
ORDER BY	avg_num_product DESC

```
Answer:

--city
"city_name"	"avg_num_product"
"Council Bluffs"7589.0000000000000000
"Cork"	        3786.0000000000000000
"Bellflower"	3786.0000000000000000
"Santiago"	3607.0000000000000000
"Bellingham"	2836.0000000000000000
"Detroit"	2748.0000000000000000
"Westville"	2299.0000000000000000
"Santa Fe"	1932.5000000000000000

Etc...

Total of 250 rows

--country
"country"	 "avg_num_product"
"Montenegro"	  3786.0000000000000000
"Mali"	          3786.0000000000000000
"Papua New Guinea"2558.0000000000000000
"Réunion"	  2538.0000000000000000
"Georgia"	  2506.4000000000000000
"Côte d’Ivoire"	  1928.5000000000000000
"Moldova"	  1893.0000000000000000
"Tanzania"	  1429.0000000000000000

Etc...

Total of 134 rows


**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:
```sql
--product category ordered from each city
SELECT		ac.v2_product_category,ac.city_name, SUM(p.ordered_quantity)AS quantity_ordered 
FROM		all_sessions_clean AS ac
JOIN		products AS p
ON		ac.product_sku = p.sku		
WHERE		city_name IS NOT NULL
		AND ac.v2_product_category NOT IN (
		'${escCatTitle}','(not set)' )
GROUP BY	ac.city_name, ac.v2_product_category
ORDER BY	ac.v2_product_category, ac.city_name


--product category ordered from each country
SELECT		ac.v2_product_category,ac.country, SUM(p.ordered_quantity)AS quantity_ordered 
FROM		all_sessions_clean AS ac
JOIN		products AS p
ON			ac.product_sku = p.sku		
WHERE		country IS NOT NULL
			AND ac.v2_product_category NOT IN (
			'${escCatTitle}','(not set)' )
GROUP BY	ac.country, ac.v2_product_category
ORDER BY	ac.v2_product_category, ac.country

--Pattern-CITY
SELECT 		v2_product_category, COUNT(v2_product_category)
FROM		(SELECT		ac.v2_product_category,ac.city_name, SUM(p.ordered_quantity)AS quantity_ordered 
			FROM		all_sessions_clean AS ac
			JOIN		products AS p
			ON			ac.product_sku = p.sku		
			WHERE		city_name IS NOT NULL
			AND ac.v2_product_category NOT IN (
			'${escCatTitle}','(not set)' )
			GROUP BY	ac.city_name, ac.v2_product_category
			ORDER BY	ac.v2_product_category, ac.city_name)
GROUP BY	v2_product_category
ORDER BY	COUNT(v2_product_category) DESC

--Pattern-COUNTRY
SELECT 		v2_product_category, COUNT(v2_product_category)
FROM		(SELECT		ac.v2_product_category,ac.country, SUM(p.ordered_quantity)AS quantity_ordered 
			FROM		all_sessions_clean AS ac
			JOIN		products AS p
			ON			ac.product_sku = p.sku		
			WHERE		country IS NOT NULL
			AND ac.v2_product_category NOT IN (
			'${escCatTitle}','(not set)' )
			GROUP BY	ac.country, ac.v2_product_category
			ORDER BY	ac.v2_product_category, ac.country)
GROUP BY	v2_product_category
ORDER BY	COUNT(v2_product_category) DESC

```


Answer:
The top 2 categories in both city and country ranks are "Home/Apparel/Men's/Men's-T-Shirts/" and "Home/Shop by Brand/YouTube/".
Out of the top 5 categories in both city and country ranks, the keyword "apparel" shows up 50% of the time. This indicates that a big portion of the customers are looking for that specific product type.

Top 5 categories in cities
"v2_product_category"			"count"
"Home/Apparel/Men's/Men's-T-Shirts/"	113
"Home/Shop by Brand/YouTube/"		98
"Home/Apparel/"				93
"Home/Electronics/"			69
"Home/Apparel/Men's/Men's-Outerwear/"	68


Top 5 categories in countries
"v2_product_category"			"count"
"Home/Shop by Brand/YouTube/"		104
"Home/Apparel/Men's/Men's-T-Shirts/"	83
"Home/Shop by Brand/Google/"		56
"Home/Electronics/"			56
"Home/Apparel/Men's/Men's-Outerwear/"	55



**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:

```sql
--top selling product in cities
WITH city_product_rank AS
(SELECT		ac.city_name, 
		p.sku, 
		p.name, 
		SUM(p.ordered_quantity) AS top_selling_product_quantity, 
		RANK() OVER (PARTITION BY ac.city_name ORDER BY SUM(p.ordered_quantity) DESC) AS product_rank
FROM		all_sessions_clean AS ac
JOIN		products AS p
ON		ac.product_sku = p.sku
WHERE		city_name IS NOT NULL
GROUP BY	ac.city_name,p.sku, p.name
ORDER BY	ac.city_name, top_selling_product_quantity DESC)

SELECT		*
FROM		city_product_rank
WHERE		product_rank = 1


--top selling product in countries
WITH country_product_rank AS
(SELECT		ac.country, 
			p.sku, p.name, 
			SUM(p.ordered_quantity) AS top_selling_product_quantity, 
			RANK() OVER (PARTITION BY ac.country ORDER BY SUM(p.ordered_quantity) DESC) AS product_rank
FROM		all_sessions_clean AS ac
JOIN		products AS p
ON			ac.product_sku = p.sku
WHERE		country IS NOT NULL and country <> '(not set)'
GROUP BY	ac.country,p.sku, p.name
ORDER BY	ac.country, top_selling_product_quantity DESC)


SELECT		*
FROM		country_product_rank
WHERE		product_rank = 1




--compare the results of the top selling items in both lists

WITH city_topselling_product AS (
    WITH city_product_rank AS 
	(SELECT  ac.city_name,
            	 p.sku,
            	 p.name AS top_selling_product,
            	 SUM(p.ordered_quantity) AS top_selling_product_quantity,
            	 RANK() OVER (PARTITION BY ac.city_name ORDER BY SUM(p.ordered_quantity) DESC) AS product_rank
        FROM	 all_sessions_clean AS ac
        JOIN	 products AS p
        ON		 ac.product_sku = p.sku
        WHERE	 ac.city_name IS NOT NULL
        GROUP BY ac.city_name, p.sku, p.name)

      SELECT	 sku,
        		 top_selling_product,
        		 COUNT(*) AS count,
        		 'city' AS area
	  FROM		 city_product_rank
      WHERE		 product_rank = 1
      GROUP BY	 sku, top_selling_product),

country_topselling_product AS (
    WITH country_product_rank AS (
        SELECT
            ac.country,
            p.sku,
            p.name AS top_selling_product,
            SUM(p.ordered_quantity) AS top_selling_product_quantity,
            RANK() OVER (PARTITION BY ac.country ORDER BY SUM(p.ordered_quantity) DESC) AS product_rank
        FROM
            all_sessions_clean AS ac
        JOIN
            products AS p
        ON
            ac.product_sku = p.sku
        WHERE
            ac.country IS NOT NULL AND ac.country <> '(not set)'
        GROUP BY
            ac.country, p.sku, p.name
    )

    SELECT
        sku,
        top_selling_product,
        COUNT(*) AS count,
        'country' AS area
    FROM
        country_product_rank
    WHERE
        product_rank = 1
    GROUP BY
        sku, top_selling_product
)

SELECT
    sku,
    top_selling_product,
    count,
    area
FROM
    city_topselling_product

UNION ALL

SELECT
    sku,
    top_selling_product,
    count,
    area
FROM
    country_topselling_product
ORDER BY
    sku;
```


Answer:
"city_name"	"sku"		"name"						"top_selling_product_quantity"
"Adelaide"	"GGOEGAAX0568"	" Men's Watershed Full Zip Hoodie Grey"		54
"Ahmedabad"	"GGOEYOLR018699"" Leatherette Notebook Combo"			1148
"Akron"		"GGOEGAAX0106"	" Men's 100% Cotton Short Sleeve Hero Tee Navy"	15
"Amsterdam"	"GGOEYOCR077799"" Hard Cover Journal"				1330
"Ann Arbor"	"GGOEGFYQ016599""Foam Can and Bottle Cooler"			2442
"Antalya"	"9180813"	" Tube Power Bank"				0
"Antwerp"	"GGOEGOBG023599""Colored Pencil Set"				269
"Appleton"	"GGOEGESC014699""Aluminum Handy Emergency Flashlight"		66

Etc...

Total of 250 rows

--top selling product in countries
"country"	"sku"			"name"					"top_selling_product_quantity"
"Albania"	"GGOEYDHJ056099"	"22 oz  Bottle Infuser"			1465
"Algeria"	"GGOEYHPB072210"	" Twill Cap"				1429
"Argentina"	"GGOEGCBQ016499"	"SPF-15 Slim & Slender Lip Balm"	3682
"Armenia"	"GGOEAKDH019899"	"Windup Android"			1351
"Australia"	"GGOEYFKQ020699"	" Custom Decals"			18930
"Austria"	"GGOEYFKQ020699"	" Custom Decals"			3786
"Bahamas"	"GGOEGHPB071610"	" Twill Cap"				917
"Bahrain"	"GGOEGAAX0104"	" Men's 100% Cotton Short Sleeve Hero Tee White"528

Etc...

Total of 134 rows


--combined results for top-selling items in city and country 
"sku"			"top_selling_product"				"count"		"area"
"9180813"		" Tube Power Bank"				1		"city"
"9180905"		" Men's Long Sleeve Raglan Ocean Blue"		1		"city"
"9182743"		" Men's Microfiber 1/4 Zip Pullover Blue/Indigo"1		"city"
"9184734"		" Learning Thermostat 3rd Gen-USA - Copper"	1		"city"
"GGOEADHH055999"	"22 oz Android Bottle"				1		"city"
"GGOEADHH055999"	"22 oz Android Bottle"				1		"country"
"GGOEAFKQ020499"	"8 pc Android Sticker Sheet"			1		"city"
"GGOEAFKQ020599"	"Android Sticker Sheet Ultra Removable"		1		"city"

As can be seen in the combined table there are actually a lot of top-selling items that exists in both the city and country list. 



**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:

```sql
--transaction rev by city
WITH total_transaction_revenue_city_cte AS
(SELECT		SUM(adj_total_transaction_rev)AS total_transaction_revenue
FROM		all_sessions_clean AS ac
JOIN		transaction_rev AS tr
ON			ac.fullvisitorid = tr.fullvisitorid
WHERE		ac.city_name IS NOT NULL)

SELECT		ac.city_name,ac.country, 
			SUM(adj_total_transaction_rev) AS transaction_rev_by_city,
			SUM(adj_total_transaction_rev)/(SELECT total_transaction_revenue FROM total_transaction_revenue_city_cte )*100 AS revenue_percentage
FROM		all_sessions_clean AS ac
JOIN		transaction_rev AS tr
ON			ac.fullvisitorid = tr.fullvisitorid
WHERE		ac.city_name IS NOT NULL
GROUP BY	ac.city_name,ac.country
HAVING		SUM(adj_total_transaction_rev) IS NOT NULL
ORDER BY	SUM(adj_total_transaction_rev) DESC

--Based on the transaction revenue data, the top 4 city that generates the highest revenue are all in the US. 
--The top 4 cities accounted for over 50% of the revenue 

WITH transaction_rev_by_city AS
(SELECT		ac.city_name,ac.country, SUM(adj_total_transaction_rev) AS transaction_rev_by_city
FROM		all_sessions_clean AS ac
JOIN		transaction_rev AS tr
ON			ac.fullvisitorid = tr.fullvisitorid
WHERE		ac.city_name IS NOT NULL
GROUP BY	ac.city_name,ac.country
HAVING		SUM(adj_total_transaction_rev) IS NOT NULL
ORDER BY	SUM(adj_total_transaction_rev) DESC)

SELECT		country, COUNT(*) 
FROM		transaction_rev_by_city
GROUP BY	country
--out of 20 countries that generates revenue, 15 are in the US. 

--transaction rev by country
WITH total_transaction_revenue_country_cte AS
(SELECT		SUM(adj_total_transaction_rev)AS total_transaction_revenue
FROM		all_sessions_clean AS ac
JOIN		transaction_rev AS tr
ON			ac.fullvisitorid = tr.fullvisitorid
WHERE		ac.country IS NOT NULL)


SELECT		ac.country, 
			SUM(adj_total_transaction_rev) AS transaction_rev_by_country,
			SUM(adj_total_transaction_rev)/(SELECT total_transaction_revenue FROM total_transaction_revenue_country_cte) *100 AS revenue_percentage
FROM		all_sessions_clean AS ac
JOIN		transaction_rev AS tr
ON			ac.fullvisitorid = tr.fullvisitorid
WHERE		ac.country IS NOT NULL
GROUP BY	ac.country
HAVING		SUM(adj_total_transaction_rev) IS NOT NULL
ORDER BY	SUM(adj_total_transaction_rev) DESC

--more than 93% of the revenue are genenerated from the US. 
--Israel and Australia are ranked the 2nd and 3rd with 3.62% and 2.15%, respectively. 
```

Answer:

--transaction rev by city
"city_name"	"country"	"transaction_rev_by_city"	"revenue_percentage"
"San Francisco"	"United States"	1877510000			20.030191819403846
"Atlanta"	"United States"	1596920000			17.03672093370602
"Sunnyvale"	"United States"	1112230000			11.865811765207928
"Palo Alto"	"United States"	608000000			6.486440352486825
"Tel Aviv-Yafo"	"Israel"	602000000			6.422429427955705
"New York"	"United States"	530360000			5.658138989054132
"Mountain View"	"United States"	492340000			5.252523097275268
"Los Angeles"	"United States"	479480000			5.1153263490302345

Total of 20 rows

--transaction rev by country

"country"	"transaction_rev_by_country"	"revenue_percentage"
"United States"	15524270000			93.2309636241015
"Israel"	602000000			3.615309454274443
"Australia"	358000000			2.149968080781147
"Canada"	150150000			0.9017254394672883
"Switzerland"	16990000			0.10203340137561925


The company relies heavily on the revenue generated from the US but has seen success in Tel Aviv-Yafo in Israel as a top-5 revenue generating city.







