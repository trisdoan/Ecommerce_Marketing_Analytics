# :computer:Ecommerce Marketing Analytics

CREDIT: John Pauler.

You can view his course here: https://www.udemy.com/course/advanced-sql-mysql-for-analytics-business-intelligence/

## :page_facing_up:Overview
I work as an Data Analyst for Maven Fuzzy Factory, an online retailer.

I worked with the Head of Marketing and website manager to evaluate company's performance: marketing channels, website conversion rate and understanding impact of new product launches.


## :pencil:Objectives
1. Using trended perfomance data to show company's growth

2. Explain details company's perfomance

3. Quantify the revenue impact

4. Analyze current performance and assess upcoming opportunities

5. Explain growth by diving to channels and website optimizations

## :green_book:Table of contents

1. Part 1 - Analyzing first 8 month
2. Part 2 - Analyzing

# PART 1
  The company has been live for 8 month. CEO requested to make a dashboard in order to present company performance metrics to the board.
  
  The following dashboard is created by me in order to visualize what I found out after 8 month of the business.
  
  <img src="images/part1.png" width="800"/>
  
## 1. Gsearch seems to be biggest driver of business. Request is: pull monthly trends for gsearch sessions and orders to showcase growth

### Steps:
- Use **LEFT JOIN** to merge website_session table with orders table
- Use **COUNT DISTINCT** to count monthly sessions and order

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
    AND A.created_at < '2012-11-27'
GROUP BY YEAR(A.created_at),MONTH(created_at)
ORDER BY yy;
````


## 2. Display similar trend for gsearch monthly, but this time splitting out nonbrand and brand. Manager wonders if brand is picking up at all

### Steps:
- Use **LEFT JOIN** to merge website_session table with orders table
- Use **COUNT** and **CASE WHEN** to filter sessions running by brand and non_brand campaigns


````sql
Select
	YEAR(A.created_at) AS yy,
	MONTH(A.created_at) AS mm,
    COUNT(DISTINCT CASE WHEN A.utm_campaign = 'nonbrand' then A.website_session_id else null END) AS non_brand_ses_count,
    COUNT(DISTINCT CASE WHEN A.utm_campaign = 'nonbrand' then B.order_id else null END) AS non_brand_order_count,
    COUNT(DISTINCT CASE WHEN A.utm_campaign = 'brand' then A.website_session_id else null END) AS brand_ses_count,
    COUNT(DISTINCT CASE WHEN A.utm_campaign = 'brand' then B.order_id  else null END) AS brand_order_count
From website_sessions A 
LEFT JOIN orders B
	ON A.website_session_id = B.website_session_id
Where A.utm_source = 'gsearch'
   AND A.utm.created_at < '2012-11-27'
GROUP BY 
		YEAR(A.created_at),
		MONTH(A.created_at);

````




## 3. Still on gsearch, dive into nonbrand, pull monthly sessions and orders split by device type

### Steps:
- Use **LEFT JOIN** to merge website_session table with orders table
- Use **COUNT** and **CASE WHEN** to filter sessions viewed by mobile and desktop

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
Where A.utm_source = 'gsearch'
    AND A.utm_campaign = 'nonbrand'
    AND A.utm.created_at < '2012-11-27'
GROUP BY 
		YEAR(A.created_at),
		MONTH(A.created_at);
````



## 4. Pull monthly sessions trends for Gesearch, alongside monthly trends for each channels 


### Steps:
- Use **LEFT JOIN** to merge website_session table with orders table
- Use **COUNT** and **CASE WHEN** to filter sessions generated by channels

````sql
Select
	YEAR(created_at) AS yy,
	MONTH(created_at) AS mm,
    COUNT(DISTINCT CASE WHEN utm_source = 'gsearch' then website_session_id else null END) AS gsearch_paid_sessions,
    COUNT(DISTINCT CASE WHEN utm_source = 'bsearch' then website_session_id else null END) AS bsearch_paid_sessions,
    COUNT(DISTINCT CASE WHEN utm_source is null and http_referer is not null then website_session_id else null END) AS organic_sessions,
    COUNT(DISTINCT CASE WHEN utm_source is null and http_referer is null then website_session_id else null END) AS direct_type_in_sessions
From website_sessions 
WHERE website_sessions.created_at < '2012-11-27'
GROUP BY 
		YEAR(created_at),
		MONTH(created_at);
````





## 5. To analyze website performance improvement, pull session to order conversion rate, by month for the first 8 month


### Steps:
- Use **LEFT JOIN** to merge website_session table with orders table
- Use **COUNT DISTINCT** to find how many sessions and orders were generated
- Then divide amount of orders by sessions to calculate conversion rate

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
WHERE  website_sessions.created_at < '2012-11-27'
GROUP BY 
	YEAR(A.created_at),
	MONTH(A.created_at);
````





## 6. For the gsearch lander test, estimate incremental revenue that test has earned from 19 Jun to Jul 28


First, I used **Min** to find the first test pageview when the gsearch lander test started.

````sql
Select 
	Min(website_pageview_id) as first_test_pv
From website_pageviews
Where pageview_url = '/lander-1';
````
So when first test started, the vert first website_pageview_id is  23504 


Then I did following steps to calculate conversion rate.

Firstly, I used **TEMPORARY TABLE** where I merged website_pageviews and website_sessions to generate all session with their first pageview
````sql
DROP TABLE IF EXISTS first_test_pv; 
CREATE temporary TABLE first_test_pv AS 	
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
	Group by A.website_session_id;
````

Secondly, I found landing page for each session by **LEFT JOIN** with website_pageviews again, restricting only url '/home' and '/lander-1'
````sql
DROP TABLE IF EXISTS nonbrand_test_session_w_landing_pages; 
CREATE temporary TABLE nonbrand_test_session_w_landing_pages AS 	
	Select
		A.website_session_id,
		B.pageview_url as landing_page
	From first_test_pv A
	Left join website_pageviews B
		ON A.min_pv_id = B.website_pageview_id 
	Where B.pageview_url IN ('/home', '/lander-1' );
````

Thirdly, I joined with order to find order for each sessions which converted in to transactions. I used **LEFT JOIN** because there are some sessions not converting into transactions.
````sql
DROP TABLE IF EXISTS nonbrand_test_session_w_orders; 
CREATE temporary TABLE nonbrand_test_session_w_orders AS 	
	Select
		A.website_session_id,
		A.landing_page,
		B.order_id as order_id
	From nonbrand_test_session_w_landing_pages A
	Left join orders B
		ON A.website_session_id= B.website_session_id;
````

Lastly, I calculated conversion rate for each landing page.

````sql
	Select
		landing_page,
		Count(Distinct website_session_id) as sessions,
		Count(Distinct order_id) as orders,
		Count(Distinct order_id)/Count(Distinct website_session_id) as conv_rate
	From nonbrand_test_session_w_orders
	Group by 1;
````
So
	-- 0.0319 for /home vs 0.0406 for lander-1
	-- 0.0087 additional orders per session


In order to find incremental revenue, I found the latest session, meaning the first session of testlanding page.
````sql
	Select
		Max(A.website_session_id) as most_recent_gsearch_nonbrand_home_pv
	From website_sessions A
	Left join website_pageviews B
		On A.website_session_id = B.website_session_id
	Where utm_source = 'gsearch'
		AND utm_campaign = 'nonbrand'
    	AND pageview_url = '/home'
		AND A.created_at < '2012-11-27';
````
So the first session starting test landing page is 17145.

Finally, I counted all session starting with session_id = 17145
````sql
	Select 
		Count(website_session_id) as sessions_since_test
	From website_sessions
	Where created_at < '2012-11-27'
		AND website_session_id > 17145
    	AND utm_source = 'gsearch'
    	AND utm_campaign = 'nonbrand';
````
So
	-- sessions_since_test: 22972
	-- 22972 * 0.00887(incremental cvrate) = 202 incremental orders since that test;
	


## 7. For landing page test analyzed previous, show full conversion funnel from each of the two pages to orders from 19 Jun to Jul 28

I initially created a temporary table which includes full funnels, meaning the road leading customers from homepage to thank-you for purchasing.
* Firstly, I merged website_sessions and website_pageviews by **LEFT JOIN**
* Then I used **CASE WHEN** to check if each segment went through each section of the funnel
* Thirdly, I counted how many sessions went through each segment of the funnel
* Lastly, I counted how many sessions for each  two landing pages: homepage and customer_lander
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
	GROUP BY 1);
````

	
Finally, I calculated conversion rate for each page
````sql
	Select
		segment,
		to_product/sessions as lander_clickthr,
		to_mrfuzzy/sessions as product_clickthr,
		to_cart/sessions as mrfuzzy_clickthr,
		to_shipping/sessions as cart_clickthr,
		to_billing/sessions as shipping_clickthr,
		to_thankyou/sessions as billing_clickthr
	FROM full_funnel;
````


## 8. Quantify impact of billing test. Analyze the lift  generated from test(Sep 10 - Nov 10), in term of revenue per billing page session And pull number of billing page sessions for the past month to understand monthly impact

* Firstly, I created a **CTE** to get sessions which went to 2 billing pages, including revenue generated.
* Then I divided total revenue by number of sessions in order to get revenue generated per session.

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
	GROUP BY biling_version_seen;
````
So the old verson generated 22.99 USD per page, while the new version(/billing-2) generated 31.38 USD, more 8 USD than the old one did.
	
Finally, I count the total sessions for the past month (27/10 -27/11)
````sql
	Select
		COUNT(website_session_id) AS billing_session
	From website_pageviews
	Where pageview_url IN ('/billing', '/billing-2')
		AND created_at BETWEEN '2012-10-27' AND '2012-11-27';
````
So there were 1194 sessions in total when the new version was implemented, meaning the company generated 10,160k USD.




## 9.  Show volume growth: Pull overall session and order volume, trended by quarter

### Steps:
- Use **LEFT JOIN** to merge website_sessions with orders so as to pull sessions and orders, trended by quarter.
- Then divided number of orders by number of sessions to pull conversion rate per quarter.

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


## 10. Showcase all of efficiency improvements. Show quarterly figures for session-to-order, conversion rate, revenue per order, revenue per session

### Steps:
- Did the same as the above questions.
- Added revenue generated per order and session.

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

 ## 11. Showcase grown specific channels. pull quarterly view of orders from Gsearch nonbrand, Bsearch nonbrand, brand search overall, organic search and direct type-in
 
### Steps:
- Use **LEFT JOIN** to merge website_sessions with orders so as to pull orders, trended by quarter.
- Use **CASE WHEN** to separate orders by specific channels.

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

 ## 12. Showcase overall session-to-order conv rate trends for those same channels, by quarter. Please also make note of any periods where we made major improvement, optimization
 
### Steps:
- Use **LEFT JOIN** to merge website_sessions with orders so as to pull sessions and orders, trended by quarter.
- Then divided number of orders by number of sessions to pull conversion rate per quarter.

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


 ## 13. Pull monthly trending for revenue and margin by product, along with total sales and revenu. Note anything about seasonality  
 
### Steps:
- Use **CASE WHEN** and **SUM** to get revenue and margin by product.

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
GROUP BY YEAR(created_at),
	MONTH(created_at);
````


 ## 14. Dive deep into impact of introducing new products. Pull monthly sessions to the product page, and show how % of those sessions clicking through another page has changed over time, along with a view of how conversion from products to placing an order has improved
 

Firstly, I created a temporary table which contains number of sessons going to page "product"
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
````

Then I counted distinct sessions trended by month
````sql
Select
    YEAR(date_seen) AS yr,
    MONTH(date_seen) AS mn,
    COUNT(DISTINCT website_session_id) AS sessions
From session_to_product
GROUP BY 1,2;
````


````sql
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



 ## 15. 4th product was available as a primary product on Dec 05, 2014 Pull sales data since then, and show how well each product cross-sell from one another
 
 Firstly, I created a temporary table to pull order_id with primary_product_id, filted by dates greater than 05/12/2014
````sql
DROP TABLE IF EXISTS sale_data ;
CREATE TEMPORARY TABLE sale_data AS
Select
    order_id,
    primary_product_id
From orders
WHERE created_at >'2014-12-05';
````

Then I used **CTE** to merge sale_data(temporary table) with order_items.

Finally, I used **CASE WHEN** and **COUNT DISTINCT** to calculate conversion rate for each cross sell product
````sql
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
    GROUP BY primary_product_id
````

