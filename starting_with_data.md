Question 1: find the total number of unique visitors

SQL Queries:
```sql
SELECT		COUNT(*)
FROM		(SELECT		DISTINCT(fullvisitorid)
			FROM		all_sessions_clean 

			UNION

			SELECT		DISTINCT(fullvisitorid)
			FROM		analytics)
```
Answer: 130345



Question 2: find the total number of unique visitors by referral

SQL Queries:
```sql
CREATE OR REPLACE VIEW unique_fullvisitorid AS
(SELECT		DISTINCT(fullvisitorid), channel_grouping
			FROM		all_sessions_clean 

			UNION

			SELECT		DISTINCT(fullvisitorid), channel_grouping
			FROM		analytics_with_correct_unit_price)

WITH all_sessions_analytics_referral_cte AS
(SELECT		uf.fullvisitorid, ac.channel_grouping AS channel1,al.channel_grouping AS channel2 
FROM		all_sessions_clean AS ac
JOIN		unique_fullvisitorid AS uf
ON			uf.fullvisitorid = ac.fullvisitorid
JOIN		analytics_with_correct_unit_price AS al
ON			uf.fullvisitorid = al.fullvisitorid
WHERE		ac.channel_grouping = 'Referral' 
			AND al.channel_grouping = 'Referral')

SELECT		COUNT(DISTINCT (fullvisitorid))
FROM		all_sessions_analytics_referral_cte
```

Answer: 672




Question 3: compute the percentage of visitors to the site that actually makes a purchase

SQL Queries:
```sql
WITH visits_with_revenue_cte AS
(SELECT		COUNT(DISTINCT(visitid)) AS visits_with_revenue
FROM		analytics_with_correct_unit_price
WHERE		revenue IS NOT NULL),

total_visits_cte AS
(SELECT		COUNT(DISTINCT(visitid)) AS total_visits
FROM		analytics_with_correct_unit_price)

SELECT		(SELECT	CAST(visits_with_revenue AS DOUBLE PRECISION) FROM visits_with_revenue_cte)
			/(SELECT total_visits FROM total_visits_cte)*100 AS traffic_conversion_rate
```
Answer: 4.3%
