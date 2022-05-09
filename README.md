# Case Study: Ecommerce Marketing Analytics

CREDIT: John Pauler.

You can view his course here: https://www.udemy.com/course/advanced-sql-mysql-for-analytics-business-intelligence/

## Case Study Overview
Help the owner of the restauntant to generate some datasets so his team can easily inspect the data without needing to use SQL.

**1. Visiting patterns**

**2. Amount customers have spent**

**3. Top favorite products**

## Summarized Insights

1. 


## Techniques I used:
1. CTE
2. Windown function: ROW_NUMBER, RANK()
3. Aggregate functions: SUM, COUNT
4. CASE WHEN

## 1. Gsearch seems to be biggest driver of business =>  pull monthly trends for gsearch sessions and orders to show growth

### Steps:
- Use **LEFT JOIN** to find all sales for each customers
- Use **SUM** to calculate total sales per customers


````sql
Select
	YEAR(A.created_at) AS yy,
	MONTH(A.created_at) AS mm,
    COUNT( DISTINCT A.website_session_id) as monthly_sess_count,
    COUNT( DISTINCT B.order_id) as monthly_order_count
From website_sessions A
LEFT JOIN orders B
	ON A.website_session_id = B.website_session_id
WHERE A.utm_source = 'gsearch'
GROUP BY YEAR(A.created_at),MONTH(created_at)
ORDER BY yy;
````

| customer_id | total_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |



## 2. monthly similar trend for gsearch, split nonbrand and brand. Manager wonders if brand is picking up at all

### Steps:
- Use **COUNT AND GROUP BY** to find how many days each customers visited


````sql
Select
	YEAR(A.created_at) AS yy,
	MONTH(A.created_at) AS mm,
    COUNT(DISTINCT CASE WHEN A.utm_campaign = 'nonbrand' then A.website_session_id else null END) AS non_brand_ses_count,
    COUNT(DISTINCT CASE WHEN A.utm_campaign = 'nonbrand' then B.order_id else null END) AS non_brand_order_count,
    COUNT(DISTINCT CASE WHEN A.utm_campaign = 'brand' then A.website_session_id else null END) AS brand_ses_count,
    COUNT(DISTINCT CASE WHEN A.utm_campaign = 'nonbrand' then B.order_id  else null END) AS brand_order_count
From website_sessions A 
LEFT JOIN orders B
	ON A.website_session_id = B.website_session_id
Where utm_source = 'gsearch'
GROUP BY 
		YEAR(A.created_at),
		MONTH(A.created_at);

````
| customer_id | days        |
| ----------- | ----------- |
| B           | 6           |
| A           | 4           |
| C           | 2           |



## 3. still gsearch, dive into nonbrand, pull monthly sessions and orders split by device type ?

### Steps:
- Use **CTE** and **RANK()** to rank order by each customers
- Use **SELECT DISTINCT** to find the first item purchased

````sql
Select
	YEAR(A.created_at) AS yy,
	MONTH(A.created_at) AS mm,
    COUNT(DISTINCT CASE WHEN A.device_type = 'mobile' then A.website_session_id else null END) AS mobile_ses_count,
    COUNT(DISTINCT CASE WHEN A.device_type = 'mobile' then B.order_id else null END) AS mobile_order_count,
    COUNT(DISTINCT CASE WHEN A.device_type = 'desktop' then A.website_session_id else null END) AS deskt_ses_count,
    COUNT(DISTINCT CASE WHEN A.device_type = 'desktop' then B.order_id  else null END) AS deskt_order_count
From website_sessions A 
LEFT JOIN orders B
	ON A.website_session_id = B.website_session_id
Where utm_source = 'gsearch'
	AND
		utm_campaign = 'nonbrand'
GROUP BY 
		YEAR(A.created_at),
		MONTH(A.created_at);
````
| customer_id | product_name|
| ----------- | ----------- |
| A           | curry       |
| A           | sushi       |
| B           | curry       |
| C           | ramen       |




## 4. pull monthly sessions trends for Gesearch, alongside monthly trends for each channels 


### Steps:
- Use **LEFT JOIN** to find all number of products
- Use **GROUP BY** and **ORDER** to find the most item purchased

````sql
Select
	YEAR(created_at) AS yy,
	MONTH(created_at) AS mm,
    COUNT(DISTINCT CASE WHEN utm_source = 'gsearch' then website_session_id else null END) AS gsearch_paid_sessions,
    COUNT(DISTINCT CASE WHEN utm_source = 'bsearch' then website_session_id else null END) AS bsearch_paid_sessions,
    COUNT(DISTINCT CASE WHEN utm_source is null and http_referer is not null then website_session_id else null END) AS organic_sessions,
	COUNT(DISTINCT CASE WHEN utm_source is null and http_referer is null then website_session_id else null END) AS direct_type_in_sessions
From website_sessions 
GROUP BY 
		YEAR(created_at),
		MONTH(created_at);
````
| product_name | count_sales |
| ------------ | ----------- |
| ramen        | 8           |




## 5. website performance improvement, pull session to order conversion rate, by month


### Steps:
- Use **CTE** and **RANK()** to find and rank all items purchased by each customers
- Get only the first rank of each customers

````sql
Select
	YEAR(A.created_at) AS yy,
	MONTH(A.created_at) AS mm,
    COUNT(DISTINCT A.website_session_id) as sessions,
    COUNT(DISTINCT B.order_id) as orders,
    ROUND(COUNT(DISTINCT B.order_id)/COUNT(DISTINCT A.website_session_id),4)*100 as conv_rate
From website_sessions A  
LEFT JOIN orders B
	On A.website_session_id = B.website_session_id
GROUP BY 
		YEAR(A.created_at),
		MONTH(A.created_at);
````
| customer_id | product_name | count_sales |
| ----------- | ------------ | ----------- |
| A           | ramen        |3            |
| B           | sushi        |2            |
| B           | ramen        |2            |
| B           | curry        |2            |
| C           | ramen        |3            |





## 6. For the gsearch lander test, estimate revenue that test earned us


### Steps:
- Use **CTE** and **RANK()*** to rank each time customers purchased
- Use **LEFT JOIN** to join back to table menu to get product_name

````sql
Select 
	Min(website_pageview_id) as first_test_pv
From website_pageviews
Where pageview_url = '/lander-1';
# first website_pageview_id = 23504 when first test started


DROP TABLE IF EXISTS conv_rate; 
CREATE temporary TABLE conv_rate AS 	
	WITH first_test_pv AS (
		Select
			A.website_session_id,
		    Min(A.website_pageview_id) as min_pv_id
		From website_pageviews A
		INNER JOIN website_sessions B
			On A.website_session_id = B.website_session_id
	        AND B.created_at < '2012-07-28'
	        AND A.website_pageview_id > 23504
	        AND utm_source = 'gsearch'
	        AND utm_campaign = 'nonbrand'
		Group by 
			A.website_session_id
), -- Bring in landing page to each session, restricting to home or lander 1 only
	nonbrand_test_session_w_landing_pages AS (
		Select
			A.website_session_id,
			B.pageview_url as landing_page
		From first_test_pv A
		Left join website_pageviews B
			ON A.min_pv_id = B.website_pageview_id 
		Where B.pageview_url IN ('/home', '/lander-1' )
),
	-- bring in orders
	nonbrand_test_session_w_orders AS (
		Select
			A.website_session_id,
			A.landing_page,
			B.order_id as order_id
		From nonbrand_test_session_w_landing_pages A
		Left join orders B
			ON A.website_session_id= B.website_session_id;
)
	--Find differences conv rate between 2 landing pages
	Select
		landing_page,
		Count(Distinct website_session_id) as sessions,
		Count(Distinct order_id) as orders,
		Count(Distinct order_id)/Count(Distinct website_session_id) as conv_rate
	From nonbrand_test_session_w_orders
	Group by 1;
	-- 0.0319 for /home vs 0.0406 for lander-1
	-- 0.0087 additional orders per session


--Finding the most recent pageview for gesearch nonbrand where the traffic was sent to home 
	Select
		Max(A.website_session_id) as most_recent_gsearch_nonbrand_home_pv
	From website_sessions A
	Left join website_pageviews B
		On A.website_session_id = B.website_session_id
	Where utm_source = 'gsearch'
		AND utm_campaign = 'nonbrand'
    	AND pageview_url = '/home'
		AND A.created_at < '2012-11-27';
	-- Max website_session_id = 17145

	Select 
		Count(website_session_id) as sessions_since_test
	From website_sessions
	Where created_at < '2012-11-27'
		AND website_session_id > 17145
    	AND utm_source = 'gsearch'
    	AND utm_campaign = 'nonbrand';
	-- sessions_since_test: 22972
	-- 22972 * 0.00887(incremental cvrate) = 202 incremental orders since that test;
````
| customer_id | order_date  |product_name |
| ----------- | ----------- |-----------  |
| A           | 2021-01-07  |curry        |
| B           | 2021-01-11  |sushi        |





## 7. For landing page test analyzed previous, show full conversion funnel from each of the two pages to orders

### Steps:
- Use **CTE** and **RANK()*** to rank each time customers purchased
- Use **LEFT JOIN** to join back to table menu to get product_name

````sql
DROP TABLE IF EXISTS full_funnel;
CREATE TEMP TABLE full_funnel AS (
	With pageview_level AS (
		Select
			A.website_session_id,
			B.pageview_url,
			CASE WHEN pageview_url = '/home' THEN 1 ELSE 0 END AS homepage,
			CASE WHEN pageview_url = '/lander-1' THEN 1 ELSE 0 END AS customer_lander,
			CASE WHEN pageview_url = '/products' THEN 1 ELSE 0 END AS products_page,
			CASE WHEN pageview_url = '/the-original-mr-fuzzy' THEN 1 ELSE 0 END AS mrfuzzy_page,
			CASE WHEN pageview_url = '/cart' THEN 1 ELSE 0 END AS cart_page,
			CASE WHEN pageview_url = '/shipping' THEN 1 ELSE 0 END AS shipping_page,
			CASE WHEN pageview_url = '/billing' THEN 1 ELSE 0 END AS billing_page,
			CASE WHEN pageview_url = '/thank-you-for-your-order' THEN 1 ELSE 0 END AS thankyou_page
		From website_sessions A 
		LEFT JOIN website_pageviews B 
			ON A.website_session_id = B.website_session_id
		WHERE A.utm_source = 'gsearch'
			AND A.utm_campaign = 'nonbrand'
			AND A.created_at < '2012-07-28'
			AND a.created_at > '2012-06-19'
),
	session_level AS (
	Select
		website_session_id,
		MAX(homepage) as saw_homepage,
		MAX(customer_lander) as saw_customer_lander,
		MAX(products_page) as saw_product_page,
		MAX(mrfuzzy_page) as saw_mrfuzzy_page,
		MAX(cart_page) as saw_cart_page,
		MAX(shipping_page) as saw_shipping_page,
		MAX(billing_page) as saw_billing_page,
		MAX(thankyou_page) as saw_thankyou
	From pageview_level
	GROUP BY website_session_id
)
	Select
		CASE
			WHEN saw_homepage = 1 THEN "saw_homepage"
			WHEN saw_customer_lander = 1 THEN "saw_customer_lander"
		END AS segment,
		COUNT(DISTINCT website_session_id) AS sessions,
		COUNT(DISTINCT CASE WHEN saw_product_page = 1 THEN website_session_id ELSE NULL END) AS to_product,
		COUNT(DISTINCT CASE WHEN saw_mrfuzzy_page = 1 THEN website_session_id ELSE NULL END) AS to_mrfuzzy,
		COUNT(DISTINCT CASE WHEN saw_cart_page = 1 THEN website_session_id ELSE NULL END) AS to_cart,
		COUNT(DISTINCT CASE WHEN saw_shipping_page = 1 THEN website_session_id ELSE NULL END) AS to_shipping,
		COUNT(DISTINCT CASE WHEN saw_billing_page = 1 THEN website_session_id ELSE NULL END) AS to_billing,
		COUNT(DISTINCT CASE WHEN saw_thankyou = 1 THEN website_session_id ELSE NULL END) AS to_thankyou
	FROM session_level
	GROUP BY 1
);
	
#See how conversion rate between each page
	Select
		segment,
		to_product/sessions as lander_clickthr,
		to_mrfuzzy/to_product as product_clickthr,
		to_cart/to_mrfuzzy as mrfuzzy_clickthr,
		to_shipping/to_cart as cart_clickthr,
		to_billing/to_shipping as shipping_clickthr,
		to_thankyou/to_billing as billing_clickthr
	FROM full_funnel;
````
| customer_id | order_date  |product_name |
| ----------- | ----------- |-----------  |
| A           | 2021-01-01  |sushi        |
| A           | 2021-01-01  |curry        |
| B           | 2021-01-04  |sushi        |


## 8. Quantify impact of billing test. Analyze the lift  generated from test(Sep 10 - Nov 10), in term of revenue per billing page session And pull number of billing page sessions for the past month to understand monthly impact

### Steps:
- Use **LEFT JOIN** to get information of each items when they were purchased
- Use **WHERE** to get items only order_date < join_date

````sql
With billing_pageview_w_order AS (
		Select
			A.website_session_id,
			A.pageview_url AS biling_version_seen,
			B.price_usd
		From website_pageviews A 
		LEFT JOIN orders B 
			ON A.website_session_id = B.website_session_id
		WHERE A.created_at > '2012-09-10'
			AND A.created_at < '2012-11-10'
			AND A.pageview_url IN ('/billing', '/billing-2')
)
	Select
		biling_version_seen,
		COUNT(DISTINCT website_session_id) AS sessions,
		SUM(price_usd)/COUNT(DISTINCT website_session_id) AS rev_per_page
	From billing_pageview_w_order
	GROUP BY biling_version_seen

	-- Old version: 22.99 USD 
	-- New version: 31.38 USD 
	-- => new version brought around 8 USD per page view

	#Calculate how much money this test bring
	Select
		COUNT(website_session_id) AS billing_session
	From website_pageviews
	Where pageview_url IN ('/billing', '/billing-2')
		AND created_at BETWEEN '2012-10-27' AND '2012-11-27'
	-- 1194 sessions => value of test: 10,160k USD ;
````
| customer_id | total_unique_product|total_price  |
| ----------- | ------------------- |-----------  |
| A           | 2                   |25           |
| A           | 2                   |40           | 



## 9.  Show volume growth: Pull overall session and order volume, trended by quarter

### Steps:
- Use **CASE WHEN** to double the point for product "sushi"

````sql
Select
	YEAR(A.created_at) as yr,
    quarter(A.created_at) as qr,
	COUNT(DISTINCT A.website_session_id) as total_session,
    COUNT(DISTINCT B.order_id) as orders,
	COUNT(DISTINCT B.order_id)/COUNT(DISTINCT A.website_session_id) as conv_rate
From website_sessions A 
LEFT JOIN orders B
	ON A.website_session_id = B.website_session_id
Group By YEAR(A.created_at),
		quarter(A.created_at);
````
| customer_id | total_point |
| ----------- | ----------- |
| B           | 940         |
| C           | 360         |
| A           | 860         |



## 10. Showcase all of efficiency improvements. Show quarterly figures for session-to-order, conv rate, rev per order, rev per session

### Steps:
- Use **CASE WHEN** to double the point for product "sushi" and first period when they became membership

````sql
Select
	YEAR(A.created_at) AS yr,
    quarter(A.created_at) AS qr,
	COUNT(DISTINCT B.order_id)/COUNT(DISTINCT A.website_session_id) AS session_to_order,
    SUM(price_usd)/COUNT(DISTINCT B.order_id) AS rev_per_order,
    SUM(price_usd)/COUNT(DISTINCT A.website_session_id) AS rev_per_session
From website_sessions A 
LEFT JOIN orders B
	ON A.website_session_id = B.website_session_id
Group By YEAR(A.created_at),
		quarter(A.created_at);
````
| customer_id | total_point |
| ----------- | ----------- |
| A           | 1370        |
| B           | 940         |


 ## 11. Showcase grown specific channels. pull quarterly view of orders from Gsearch nonbrand, Bsearch nonbrand, brand search overall, organic search and direct type-in
 
### Steps:
- Use **CREATE TEMP TABLE** to create a dataset which contains item information, including boolean column "member"
- Use **CASE WHEN** and **DENSE_RANK()** to rank products based on order_date 

````sql
Select
	YEAR(A.created_at) AS yy,
	quarter(A.created_at) AS mm,
    COUNT(DISTINCT CASE WHEN utm_source = 'gsearch' and utm_campaign = 'nonbrand' THEN  order_id else null END) AS gsearch_nonbrand_order,
    COUNT(DISTINCT CASE WHEN utm_source = 'bsearch' and utm_campaign = 'nonbrand' then order_id else null END) AS bsearch_nonbrand_order,
	COUNT(DISTINCT CASE WHEN utm_campaign = 'brand' then order_id else null END) AS brand_order,
    COUNT(DISTINCT CASE WHEN utm_source is null and http_referer is not null then order_id else null END) AS organic_order,
	COUNT(DISTINCT CASE WHEN utm_source is null and http_referer is null then order_id else null END) AS direct_type_in_order
From website_sessions A 
LEFT JOIN orders B 
	ON A.website_session_id = B.website_session_id
GROUP BY 
		YEAR(A.created_at),
		quarter(A.created_at);
````
| customer_id | order_date  | product_name | price | member | ranking |
| ----------- | ----------- | -----------  | ----- | ------ |-------- |
| A           | 2021-01-01  |sushi         |10     |NO      |null     |
| A           | 2021-01-01  |curry         |15     |NO      |null     |


 ## 12. Showcase overall session-to-order conv rate trends for those same channels, by quarter Please also make note of any periods where we made major improvement, optimization
 
### Steps:
- Use **CREATE TEMP TABLE** to create a dataset which contains item information, including boolean column "member"
- Use **CASE WHEN** and **DENSE_RANK()** to rank products based on order_date 

````sql
Select
	YEAR(A.created_at) AS yr,
    quarter(A.created_at) AS qr,
	COUNT(DISTINCT B.order_id)/COUNT(DISTINCT A.website_session_id) AS session_to_order
From website_sessions A 
LEFT JOIN orders B
	ON A.website_session_id = B.website_session_id
Group By YEAR(A.created_at),
		quarter(A.created_at);
````


 ## 13. pull monthly trending for revenue and margin by product, along with total sales and revenu. Note anything about seasonality  
 
### Steps:
- Use **CREATE TEMP TABLE** to create a dataset which contains item information, including boolean column "member"
- Use **CASE WHEN** and **DENSE_RANK()** to rank products based on order_date 

````sql
Select
	YEAR(created_at) AS yy,
	MONTH(created_at) AS mm,
	ROUND(SUM(price_usd)) as total_rev,
    ROUND(SUM(CASE WHEN primary_product_id = 1 then price_usd ELSE NULL END),2) as MrFuzzy_rev,
    ROUND(SUM(CASE WHEN primary_product_id = 1 then price_usd - cogs_usd ELSE NULL END),2) as MrFuzzy_margin,
    ROUND(SUM(CASE WHEN primary_product_id = 2 then price_usd ELSE NULL END),2) as Love_Bear_rev,
    ROUND(SUM(CASE WHEN primary_product_id = 2 then price_usd - cogs_usd ELSE NULL END),2) as Love_Bear_margin,
	ROUND(SUM(CASE WHEN primary_product_id = 3 then price_usd ELSE NULL END),2) as Sugar_Panda_rev,
    ROUND(SUM(CASE WHEN primary_product_id = 3 then price_usd - cogs_usd ELSE NULL END),2) as Sugar_Panda_margin,
	ROUND(SUM(CASE WHEN primary_product_id = 4 then price_usd ELSE NULL END),2) as Mini_bear_rev,
    ROUND(SUM(CASE WHEN primary_product_id = 4 then price_usd - cogs_usd ELSE NULL END),2) as Mini_bear_margin
From orders 
GROUP BY 
		YEAR(created_at),
		MONTH(created_at);
````


 ## 14. Dive deep into impact of introducing new products. Pull monthly sessions to the product page, and show how % of those sessions clicking through another page has changed over time, along with a view of how conversion from products to placing an order has improved
 
### Steps:
- Use **CREATE TEMP TABLE** to create a dataset which contains item information, including boolean column "member"
- Use **CASE WHEN** and **DENSE_RANK()** to rank products based on order_date 

````sql
DROP TABLE IF EXISTS session_to_product;
CREATE TEMPORARY TABLE session_to_product AS
Select 
	A.website_session_id,
    B.website_pageview_id,
    A.created_at as date_seen
From website_sessions A
LEFT JOIN website_pageviews B
	ON A.website_session_id = B.website_session_id
WHERE  B.pageview_url = '/products';

-- Pull monthly sessions to the product page
Select
	YEAR(date_seen) AS yr,
    MONTH(date_seen) AS mn,
    COUNT(DISTINCT website_session_id) AS sessions
From session_to_product
GROUP BY 1,2;

-- show how % of those sessions clicking 
Select 
	YEAR(date_seen) AS yr,
    MONTH(date_seen) AS mn,
	ROUND(COUNT(DISTINCT B.website_session_id)/COUNT(DISTINCT A.website_session_id),2) AS clickth_conv_from_product_page,
    ROUND(COUNT(DISTINCT C.order_id)/COUNT(DISTINCT A.website_session_id),2) AS product_to_order
From session_to_product A 
LEFT JOIN website_pageviews B
	ON A.website_session_id = B.website_session_id
    AND A.website_pageview_id < B.website_pageview_id
LEFT JOIN orders C 
	ON A.website_session_id = C.website_session_id
GROUP BY 1,2;
````






 ## 15. we made 4th product available as a primary product on Dec 05, 2014 Pull sales data since then, and show how well each product cross-sell from one another
 
### Steps:
- Use **CREATE TEMP TABLE** to create a dataset which contains item information, including boolean column "member"
- Use **CASE WHEN** and **DENSE_RANK()** to rank products based on order_date 

````sql
DROP TABLE IF EXISTS sale_data ;
CREATE TEMPORARY TABLE sale_data AS
Select
	order_id,
    primary_product_id
From orders
WHERE created_at >'2014-12-05';

WITH cte AS (
	Select
		A.order_id,
        A.primary_product_id,
        B.product_id AS cross_sell_product
    From sale_data A
    LEFT JOIN order_items B
		ON A.order_id = B.order_id
        AND is_primary_item = 0
)
	Select
		primary_product_id,
        COUNT(DISTINCT order_id) AS total_orders,
        ROUND(COUNT(DISTINCT CASE WHEN cross_sell_product = 1 THEN order_id ELSE NULL END)/COUNT(DISTINCT order_id),2) AS cross_sell_with_product_1,
		ROUND(COUNT(DISTINCT CASE WHEN cross_sell_product = 2 THEN order_id ELSE NULL END)/COUNT(DISTINCT order_id),2) AS cross_sell_with_product_2,
		ROUND(COUNT(DISTINCT CASE WHEN cross_sell_product = 3 THEN order_id ELSE NULL END)/COUNT(DISTINCT order_id),2) AS cross_sell_with_product_3,
		ROUND(COUNT(DISTINCT CASE WHEN cross_sell_product = 4 THEN order_id ELSE NULL END)/COUNT(DISTINCT order_id),2) AS cross_sell_with_product_4
    From cte
    GROUP BY primary_product_id;
````

