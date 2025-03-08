# TABLE OF CONTENTS
**1.ACKNOWLEDGEMENT**

**2.PROJECT OVERVIEW**

**3.DATA SOURCES**

**4.DATA ANALYSIS**

**5.CONCLUSION**

# ACKNOWLEDGEMENT

This project was prepared with the assistance of the course [Advanced SQL: MySQL for Ecommerce Data Analysis](https://www.udemy.com/course/advanced-sql-mysql-for-analytics-business-intelligence/) on Udemy.

# PROJECT OVERVIEW 

Case Situation :- As Maven Fuzzy Factory approaches its ninth month of operation, their CEO is set to present crucial performance indicators to the board. My task involves gathering key metrics that showcase the company's promising growth path, which will be essential for the upcoming presentation.

Objective :- As an analyst for Maven Fuzzy Factory, my task involves using SQL to extract and analyze website traffic and performance data from the company's database. This analysis aims to quantify the company's growth and provide insights into how that growth was achieved. The first step is to extract relevant data using SQL queries, focusing on metrics such as traffic sources, landing pages, and conversion funnel performance. Once the data is analyzed, the next step is to effectively communicate the findings to stakeholders, telling a compelling story of the company's growth trajectory and highlighting key factors that contributed to this success. This narrative should be supported by data-driven insights that help stakeholders understand the company's performance and future opportunities.


# DATA SOURCES
This project analyzes eCommerce data using the Maven Fuzzy Factory database, which consists of six interrelated tables. The dataset includes comprehensive information on website activity, products, and orders and refunds. By leveraging MySQL, the project explores how customers access and interact with the website, evaluates landing page performance and conversion rates, and investigates product-level sales trends. The insights gained aim to provide a deeper understanding of user behavior, product performance, and refund patterns, contributing to data-driven decision-making in an eCommerce context. Please find below ER diagram of database :-
![Alt text](https://github.com/aa-abhinavacharya/MySQL_For_Advanced_Ecommerce_Data_Analysis/blob/main/ecommerce_data_source.JPG?raw=true)

# DATA ANALYSIS

QUESTION 1 :- Gsearch seems to be the biggest driver of our business. Could you pull **monthly trends for gsearch sessions and orders** so that we can showcase the growth there?

```sql
SELECT
	YEAR(website_sessions.created_at) AS yr, 
    MONTH(website_sessions.created_at) AS mo, 
    COUNT(DISTINCT website_sessions.website_session_id) AS sessions, 
    COUNT(DISTINCT orders.order_id) AS orders, 
    COUNT(DISTINCT orders.order_id)/COUNT(DISTINCT website_sessions.website_session_id) AS conv_rate
FROM website_sessions
	LEFT JOIN orders 
		ON orders.website_session_id = website_sessions.website_session_id
WHERE website_sessions.created_at < '2012-11-27'
	AND website_sessions.utm_source = 'gsearch'
GROUP BY 1,2;
```

Solution :- 

![Alt text](https://github.com/aa-abhinavacharya/MySQL_For_Advanced_Ecommerce_Data_Analysis/blob/main/solution1.JPG)

Conclusion from QUESTION 1 result :- 

The data shows a significant increase in both Gsearch sessions and orders from March to November 2012, with a slight improvement in conversion rates. Gsearch appears to be a key driver of business growth, particularly in the later months, suggesting seasonality effects. Continued focus on optimizing Gsearch could further enhance traffic and conversions, offering opportunities for even greater growth.


QUESTION 2 :- Next, it would be great to see a similar monthly trend for Gsearch, but this time **splitting out nonbrand and brand campaigns separately**. I am wondering if brand is picking up at all. If so, this is a good story to tell. 

```sql

SELECT
	YEAR(website_sessions.created_at) AS yr, 
    MONTH(website_sessions.created_at) AS mo, 
    COUNT(DISTINCT CASE WHEN utm_campaign = 'nonbrand' THEN website_sessions.website_session_id ELSE NULL END) AS nonbrand_sessions, 
    COUNT(DISTINCT CASE WHEN utm_campaign = 'nonbrand' THEN orders.order_id ELSE NULL END) AS nonbrand_orders,
    COUNT(DISTINCT CASE WHEN utm_campaign = 'brand' THEN website_sessions.website_session_id ELSE NULL END) AS brand_sessions, 
    COUNT(DISTINCT CASE WHEN utm_campaign = 'brand' THEN orders.order_id ELSE NULL END) AS brand_orders
FROM website_sessions
	LEFT JOIN orders 
		ON orders.website_session_id = website_sessions.website_session_id
WHERE website_sessions.created_at < '2012-11-27'
	AND website_sessions.utm_source = 'gsearch'
GROUP BY 1,2;
```

Result :- 

![Alt text](https://github.com/aa-abhinavacharya/MySQL_For_Advanced_Ecommerce_Data_Analysis/blob/main/solution2.JPG))

Conclusion from QUESTION 2 result :-

The data shows a consistent increase in both nonbrand and brand Gsearch sessions and orders over the months. While nonbrand sessions and orders significantly outpace brand campaigns, brand sessions and orders do show gradual growth, particularly in the later months. This indicates that brand campaigns are gaining traction, and highlighting this upward trend in brand performance could be an impactful story to share.

QUESTION 3 :- While we’re on Gsearch, could you dive into nonbrand, and **pull monthly sessions and orders split by device type?** I want to flex our analytical muscles a little and show the board we really know our traffic sources.

```sql

SELECT
	YEAR(website_sessions.created_at) AS yr, 
    MONTH(website_sessions.created_at) AS mo, 
    COUNT(DISTINCT CASE WHEN device_type = 'desktop' THEN website_sessions.website_session_id ELSE NULL END) AS desktop_sessions, 
    COUNT(DISTINCT CASE WHEN device_type = 'desktop' THEN orders.order_id ELSE NULL END) AS desktop_orders,
    COUNT(DISTINCT CASE WHEN device_type = 'mobile' THEN website_sessions.website_session_id ELSE NULL END) AS mobile_sessions, 
    COUNT(DISTINCT CASE WHEN device_type = 'mobile' THEN orders.order_id ELSE NULL END) AS mobile_orders
FROM website_sessions
	LEFT JOIN orders 
		ON orders.website_session_id = website_sessions.website_session_id
WHERE website_sessions.created_at < '2012-11-27'
	AND website_sessions.utm_source = 'gsearch'
    AND website_sessions.utm_campaign = 'nonbrand'
GROUP BY 1,2;
```

Result:-

![Alt text](https://github.com/aa-abhinavacharya/MySQL_For_Advanced_Ecommerce_Data_Analysis/blob/main/solution3.JPG)

Conclusion from QUESTION 3 result:-

The data reveals that desktop sessions and orders consistently outpace mobile sessions and orders, indicating that desktop remains the primary driver of conversions. While mobile sessions have been growing, mobile orders are significantly lower, suggesting that mobile traffic is not converting as effectively as desktop. The trend highlights a potential opportunity to enhance the mobile user experience to capture more of the growing mobile traffic. Focusing on optimizing mobile performance could help balance the growth between both devices, maximizing overall conversions.

QUESTION 4 :- I’m worried that one of our more pessimistic board members may be concerned about the large % of traffic from Gsearch. **Can you pull monthly trends for Gsearch, alongside monthly trends for each of our other channels?**

Solution step :-  We need to find the various utm sources and referers to see the traffic we're getting
```sql

SELECT DISTINCT 
	utm_source,
    utm_campaign, 
    http_referer
FROM website_sessions
WHERE website_sessions.created_at < '2012-11-27';


SELECT
	YEAR(website_sessions.created_at) AS yr, 
    MONTH(website_sessions.created_at) AS mo, 
    COUNT(DISTINCT CASE WHEN utm_source = 'gsearch' THEN website_sessions.website_session_id ELSE NULL END) AS gsearch_paid_sessions,
    COUNT(DISTINCT CASE WHEN utm_source = 'bsearch' THEN website_sessions.website_session_id ELSE NULL END) AS bsearch_paid_sessions,
    COUNT(DISTINCT CASE WHEN utm_source IS NULL AND http_referer IS NOT NULL THEN website_sessions.website_session_id ELSE NULL END) AS organic_search_sessions,
    COUNT(DISTINCT CASE WHEN utm_source IS NULL AND http_referer IS NULL THEN website_sessions.website_session_id ELSE NULL END) AS direct_type_in_sessions
FROM website_sessions
	LEFT JOIN orders 
		ON orders.website_session_id = website_sessions.website_session_id
WHERE website_sessions.created_at < '2012-11-27'
GROUP BY 1,2;
```
Result:- 

![Alt text](https://github.com/aa-abhinavacharya/MySQL_For_Advanced_Ecommerce_Data_Analysis/blob/main/solution4.JPG)

Conclusion from QUESTION 4 result:-

The data indicates that Gsearch paid sessions have been the dominant traffic source, showing steady growth from March to November 2012. However, other channels such as Bsearch paid, organic search, and direct type-in are also contributing, though at a much smaller scale. The relatively high share of Gsearch paid sessions might raise concerns about over-reliance on one channel. However, the growth in Bsearch paid and organic search traffic over time suggests that the business is diversifying its sources, reducing dependency on a single channel. It could be valuable to emphasize the ongoing diversification efforts to reassure the board.

QUESTION 5 :-  I’d like to tell the story of our website performance improvements over the course of the first 8 months. **Could you pull session to order conversion rates, by month?**

```sql
SELECT
	YEAR(website_sessions.created_at) AS yr, 
    MONTH(website_sessions.created_at) AS mo, 
    COUNT(DISTINCT website_sessions.website_session_id) AS sessions, 
    COUNT(DISTINCT orders.order_id) AS orders, 
    COUNT(DISTINCT orders.order_id)/COUNT(DISTINCT website_sessions.website_session_id) AS conversion_rate    
FROM website_sessions
	LEFT JOIN orders 
		ON orders.website_session_id = website_sessions.website_session_id
WHERE website_sessions.created_at < '2012-11-27'
GROUP BY 1,2;
```

Result:-

![Alt text](https://github.com/aa-abhinavacharya/MySQL_For_Advanced_Ecommerce_Data_Analysis/blob/main/solution5.JPG)

Conclusion from QUESTION 5 result :-

The data highlights a clear improvement in conversion rates over the first 8 months of 2012. While the conversion rate started at 0.0319 in March, it showed gradual improvement, reaching a peak of 0.0453 in October. This increase suggests that the website's performance improved over time, likely due to optimization efforts or changes in user behavior. Although the conversion rate slightly dipped in November, it still remained higher than in the earlier months, indicating a general upward trend in conversion efficiency. This improvement in conversion rates reflects positively on website optimization and could be a key point to emphasize when discussing business growth and website performance improvements.

QUESTION 6 :- For the gsearch lander test, **please estimate the revenue that test earned us** (Hint: Look at the increase in CVR from the test (Jun 19 – Jul 28), and use nonbrand sessions and revenue since then to calculate incremental value)

Step 1 :-
```sql

USE mavenfuzzyfactory;

SELECT
	MIN(website_pageview_id) AS first_test_pv
FROM website_pageviews
WHERE pageview_url = '/lander-1';
```

Step 2:- we'll find the first pageview id 
```sql
CREATE TEMPORARY TABLE first_test_pageviews
SELECT
	website_pageviews.website_session_id, 
    MIN(website_pageviews.website_pageview_id) AS min_pageview_id
FROM website_pageviews 
	INNER JOIN website_sessions 
		ON website_sessions.website_session_id = website_pageviews.website_session_id
		AND website_sessions.created_at < '2012-07-28' -- prescribed by the assignment
		AND website_pageviews.website_pageview_id >= 23504 -- first page_view
        AND utm_source = 'gsearch'
        AND utm_campaign = 'nonbrand'
GROUP BY 
	website_pageviews.website_session_id;
```

Step 3:- we'll bring in the landing page to each session, like last time, but restricting to home or lander-1 this time.

```sql
CREATE TEMPORARY TABLE nonbrand_test_sessions_w_landing_pages
SELECT 
	first_test_pageviews.website_session_id, 
    website_pageviews.pageview_url AS landing_page
FROM first_test_pageviews
	LEFT JOIN website_pageviews 
		ON website_pageviews.website_pageview_id = first_test_pageviews.min_pageview_id
WHERE website_pageviews.pageview_url IN ('/home','/lander-1');
-- SELECT * FROM nonbrand_test_sessions_w_landing_pages;
```

Step 4:- After this, we make a table to bring in orders

```sql
CREATE TEMPORARY TABLE nonbrand_test_sessions_w_orders
SELECT
	nonbrand_test_sessions_w_landing_pages.website_session_id, 
    nonbrand_test_sessions_w_landing_pages.landing_page, 
    orders.order_id AS order_id

FROM nonbrand_test_sessions_w_landing_pages
LEFT JOIN orders 
	ON orders.website_session_id = nonbrand_test_sessions_w_landing_pages.website_session_id
;

---SELECT * FROM nonbrand_test_sessions_w_orders;
```
Step 5:- We find the difference between conversion rates

```sql
SELECT
	landing_page, 
    COUNT(DISTINCT website_session_id) AS sessions, 
    COUNT(DISTINCT order_id) AS orders,
    COUNT(DISTINCT order_id)/COUNT(DISTINCT website_session_id) AS conv_rate
FROM nonbrand_test_sessions_w_orders
GROUP BY 1;
-- .0319 for /home, vs .0406 for /lander-1 
-- .0087 additional orders per session
```
Result:-

![Alt text](https://github.com/aa-abhinavacharya/MySQL_For_Advanced_Ecommerce_Data_Analysis/blob/main/solution6(a).JPG)

The data suggests that the "lander-1" page has a higher conversion rate (0.0406) compared to the "home" page (0.0318). Despite having a slightly lower number of sessions, the "lander-1" page appears to be more effective at converting visitors into orders. 

Step 6:-finding the most recent pageview for gsearch nonbrand where the traffic was sent to /home

```sql
SELECT 
	MAX(website_sessions.website_session_id) AS most_recent_gsearch_nonbrand_home_pageview 
FROM website_sessions 
	LEFT JOIN website_pageviews 
		ON website_pageviews.website_session_id = website_sessions.website_session_id
WHERE utm_source = 'gsearch'
	AND utm_campaign = 'nonbrand'
    AND pageview_url = '/home'
    AND website_sessions.created_at < '2012-11-27'
;
-- max website_session_id = 17145
```
Result:-

![Alt text](https://github.com/aa-abhinavacharya/MySQL_For_Advanced_Ecommerce_Data_Analysis/blob/main/solution6(b).JPG)

Step 7:- Counting website sessions since test after website_session_id = 17145

```sql
SELECT 
	COUNT(website_session_id) AS sessions_since_test
FROM website_sessions
WHERE created_at < '2012-11-27'
	AND website_session_id > 17145 -- last /home session
	AND utm_source = 'gsearch'
	AND utm_campaign = 'nonbrand'
;
-- 22,972 website sessions since the test

-- X .0087 incremental conversion = 202 incremental orders since 7/29
	-- roughly 4 months, so roughly 50 extra orders per month.
```

Result:-

![Alt text](https://github.com/aa-abhinavacharya/MySQL_For_Advanced_Ecommerce_Data_Analysis/blob/main/solution6(c).JPG)

Conclusion from QUESTION 6 result:-
The Gsearch lander test has resulted in an estimated 202 incremental orders since July 29, which translates to roughly 50 extra orders per month. This indicates a positive impact on conversions due to the test, leading to a consistent increase in monthly revenue. By leveraging this incremental improvement, the business can continue to benefit from enhanced performance moving forward.

QUESTION 7 :- For the landing page test you analyzed previously, it would be great to show a **full conversion funnel from each  of the two pages to orders**. You can use the same time period you analyzed last time (Jun 19 – Jul 28).

```sql
SELECT
	website_sessions.website_session_id, 
    website_pageviews.pageview_url, 
    -- website_pageviews.created_at AS pageview_created_at, 
    CASE WHEN pageview_url = '/home' THEN 1 ELSE 0 END AS homepage,
    CASE WHEN pageview_url = '/lander-1' THEN 1 ELSE 0 END AS custom_lander,
    CASE WHEN pageview_url = '/products' THEN 1 ELSE 0 END AS products_page,
    CASE WHEN pageview_url = '/the-original-mr-fuzzy' THEN 1 ELSE 0 END AS mrfuzzy_page, 
    CASE WHEN pageview_url = '/cart' THEN 1 ELSE 0 END AS cart_page,
    CASE WHEN pageview_url = '/shipping' THEN 1 ELSE 0 END AS shipping_page,
    CASE WHEN pageview_url = '/billing' THEN 1 ELSE 0 END AS billing_page,
    CASE WHEN pageview_url = '/thank-you-for-your-order' THEN 1 ELSE 0 END AS thankyou_page
FROM website_sessions 
	LEFT JOIN website_pageviews 
		ON website_sessions.website_session_id = website_pageviews.website_session_id
WHERE website_sessions.utm_source = 'gsearch' 
	AND website_sessions.utm_campaign = 'nonbrand' 
    AND website_sessions.created_at < '2012-07-28'
		AND website_sessions.created_at > '2012-06-19'
ORDER BY 
	website_sessions.website_session_id,
    website_pageviews.created_at;

CREATE TEMPORARY TABLE session_level_made_it_flagged
SELECT
	website_session_id, 
    MAX(homepage) AS saw_homepage, 
    MAX(custom_lander) AS saw_custom_lander,
    MAX(products_page) AS product_made_it, 
    MAX(mrfuzzy_page) AS mrfuzzy_made_it, 
    MAX(cart_page) AS cart_made_it,
    MAX(shipping_page) AS shipping_made_it,
    MAX(billing_page) AS billing_made_it,
    MAX(thankyou_page) AS thankyou_made_it
FROM(
SELECT
	website_sessions.website_session_id, 
    website_pageviews.pageview_url, 
    -- website_pageviews.created_at AS pageview_created_at, 
    CASE WHEN pageview_url = '/home' THEN 1 ELSE 0 END AS homepage,
    CASE WHEN pageview_url = '/lander-1' THEN 1 ELSE 0 END AS custom_lander,
    CASE WHEN pageview_url = '/products' THEN 1 ELSE 0 END AS products_page,
    CASE WHEN pageview_url = '/the-original-mr-fuzzy' THEN 1 ELSE 0 END AS mrfuzzy_page, 
    CASE WHEN pageview_url = '/cart' THEN 1 ELSE 0 END AS cart_page,
    CASE WHEN pageview_url = '/shipping' THEN 1 ELSE 0 END AS shipping_page,
    CASE WHEN pageview_url = '/billing' THEN 1 ELSE 0 END AS billing_page,
    CASE WHEN pageview_url = '/thank-you-for-your-order' THEN 1 ELSE 0 END AS thankyou_page
FROM website_sessions 
	LEFT JOIN website_pageviews 
		ON website_sessions.website_session_id = website_pageviews.website_session_id
WHERE website_sessions.utm_source = 'gsearch' 
	AND website_sessions.utm_campaign = 'nonbrand' 
    AND website_sessions.created_at < '2012-07-28'
		AND website_sessions.created_at > '2012-06-19'
ORDER BY 
	website_sessions.website_session_id,
    website_pageviews.created_at
) AS pageview_level

GROUP BY 
	website_session_id
;
-- then this would produce the final output, part 1
SELECT
	CASE 
		WHEN saw_homepage = 1 THEN 'saw_homepage'
        WHEN saw_custom_lander = 1 THEN 'saw_custom_lander'
        ELSE 'uh oh... check logic' 
	END AS segment, 
    COUNT(DISTINCT website_session_id) AS sessions,
    COUNT(DISTINCT CASE WHEN product_made_it = 1 THEN website_session_id ELSE NULL END) AS to_products,
    COUNT(DISTINCT CASE WHEN mrfuzzy_made_it = 1 THEN website_session_id ELSE NULL END) AS to_mrfuzzy,
    COUNT(DISTINCT CASE WHEN cart_made_it = 1 THEN website_session_id ELSE NULL END) AS to_cart,
    COUNT(DISTINCT CASE WHEN shipping_made_it = 1 THEN website_session_id ELSE NULL END) AS to_shipping,
    COUNT(DISTINCT CASE WHEN billing_made_it = 1 THEN website_session_id ELSE NULL END) AS to_billing,
    COUNT(DISTINCT CASE WHEN thankyou_made_it = 1 THEN website_session_id ELSE NULL END) AS to_thankyou
FROM session_level_made_it_flagged 
GROUP BY 1
;



-- then this as final output part 2 - click rates

SELECT
	CASE 
		WHEN saw_homepage = 1 THEN 'saw_homepage'
        WHEN saw_custom_lander = 1 THEN 'saw_custom_lander'
        ELSE 'uh oh... check logic' 
	END AS segment, 
	COUNT(DISTINCT CASE WHEN product_made_it = 1 THEN website_session_id ELSE NULL END)/COUNT(DISTINCT website_session_id) AS lander_click_rt,
    COUNT(DISTINCT CASE WHEN mrfuzzy_made_it = 1 THEN website_session_id ELSE NULL END)/COUNT(DISTINCT CASE WHEN product_made_it = 1 THEN website_session_id ELSE NULL END) AS products_click_rt,
    COUNT(DISTINCT CASE WHEN cart_made_it = 1 THEN website_session_id ELSE NULL END)/COUNT(DISTINCT CASE WHEN mrfuzzy_made_it = 1 THEN website_session_id ELSE NULL END) AS mrfuzzy_click_rt,
    COUNT(DISTINCT CASE WHEN shipping_made_it = 1 THEN website_session_id ELSE NULL END)/COUNT(DISTINCT CASE WHEN cart_made_it = 1 THEN website_session_id ELSE NULL END) AS cart_click_rt,
    COUNT(DISTINCT CASE WHEN billing_made_it = 1 THEN website_session_id ELSE NULL END)/COUNT(DISTINCT CASE WHEN shipping_made_it = 1 THEN website_session_id ELSE NULL END) AS shipping_click_rt,
    COUNT(DISTINCT CASE WHEN thankyou_made_it = 1 THEN website_session_id ELSE NULL END)/COUNT(DISTINCT CASE WHEN billing_made_it = 1 THEN website_session_id ELSE NULL END) AS billing_click_rt
FROM session_level_made_it_flagged
GROUP BY 1
;
```

Result :-

![Alt text](https://github.com/aa-abhinavacharya/MySQL_For_Advanced_Ecommerce_Data_Analysis/blob/main/solution7.JPG)

Conclusion from QUESTION 7 result:-

The Custom Lander outperforms the Homepage across the entire conversion funnel, with higher conversion rates at each stage, particularly from landing page to product view and from MR Fuzzy to cart. This suggests that users coming from the custom landing page are more likely to continue through the purchase journey. The Homepage performs weaker, especially in the transition from shipping to billing.


QUESTION 8 :-  I’d love for you to **quantify the impact of our billing test**, as well. Please analyze the lift generated from the test (Sep 10 – Nov 10), in terms of **revenue per billing page session**, and then pull the number of billing page sessions for the past month to understand monthly impact.

```sql
SELECT
	billing_version_seen, 
    COUNT(DISTINCT website_session_id) AS sessions, 
    SUM(price_usd)/COUNT(DISTINCT website_session_id) AS revenue_per_billing_page_seen
 FROM( 
SELECT 
	website_pageviews.website_session_id, 
    website_pageviews.pageview_url AS billing_version_seen, 
    orders.order_id, 
    orders.price_usd
FROM website_pageviews 
	LEFT JOIN orders
		ON orders.website_session_id = website_pageviews.website_session_id
WHERE website_pageviews.created_at > '2012-09-10' -- prescribed in assignment
	AND website_pageviews.created_at < '2012-11-10' -- prescribed in assignment
    AND website_pageviews.pageview_url IN ('/billing','/billing-2')
) AS billing_pageviews_and_order_data
GROUP BY 1
;
-- $22.83 revenue per billing page seen for the old version
-- $31.34 for the new version
-- LIFT: $8.51 per billing page view
```
Result :-

![Alt text](https://github.com/aa-abhinavacharya/MySQL_For_Advanced_Ecommerce_Data_Analysis/blob/main/solution8(a).JPG)

```sql
SELECT 
	COUNT(website_session_id) AS billing_sessions_past_month
FROM website_pageviews 
WHERE website_pageviews.pageview_url IN ('/billing','/billing-2') 
	AND created_at BETWEEN '2012-10-27' AND '2012-11-27' -- past month

-- 1,194 billing sessions past month
-- LIFT: $8.51 per billing session
-- VALUE OF BILLING TEST: $10,160 over the past month
```
Result:-

![Alt text](https://github.com/aa-abhinavacharya/MySQL_For_Advanced_Ecommerce_Data_Analysis/blob/main/solution8(b).JPG)


Conclusion from QUESTION 8 result:-

The billing test generated a lift of $8.51 per billing session, leading to a total impact of $10,160 over the past month based on 1,194 billing sessions. This quantifies the positive revenue impact from the test and highlights the effectiveness of the new billing page version (/billing-2).

# CONCLUSION

There was a  gain in gathering valuable insights into traffic sources, conversion funnels, and the impact of optimization tests through the analysis of  key business data . It helped to learn how to identify growth opportunities by focusing on channels like Gsearch and understanding device-specific performance, particularly the need to optimize mobile experiences. Iy helped in brand vs. nonbrand campaigns comparison and their gradual growth highlighted the importance of balancing both efforts. Additionally, quantifying the lift from tests, such as the billing page update, reinforced the value of continuous testing for revenue growth. Overall, this experience enhances the ability to analyze performance metrics, optimize conversions, and make data-driven recommendations that drive business results.
