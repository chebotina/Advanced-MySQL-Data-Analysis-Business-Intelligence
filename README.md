# Advanced-MySQL-Data-Analysis-Business-Intelligence
Advanced SQL [Course](https://www.udemy.com/course/advanced-sql-mysql-for-analytics-business-intelligence/) from Maven Analytics

## Table of Contents
- [Overview of the MAVEN FUZZY FACTORY database](#overview-of-the-maven-fuzzy-factory-database)
- [Analyzing Traffic Sources](#analyzing-traffic-sources)
- [Analyzing Website Performance](#case-study-2)
- [Mid-Course Project](#case-study-3)
- [Analysis for Channel Portfolio Management](#case-study-4)
- [Analyzing Business Patterns and Seasonality](#case-study-5)
- [Product Analysis](#case-study-6)
- [User Analysis](#case-study-7)
- [Final Project](#case-study-8)

## Overview of the MAVEN FUZZY FACTORY database 
![1](https://user-images.githubusercontent.com/78378801/137518189-e91059a1-e3d2-4865-9aa0-154f6804cae5.jpg)

# Analyzing Traffic Sources 
<img src="https://user-images.githubusercontent.com/78378801/137519315-3f13c0d0-fc2b-4f92-9a75-8e8a9d1f8f5b.jpg" alt="Image" width="700">

### 1. Find top traffic sources with a breakdown by UTM sourse, campaign, and reffering domain. 
Note: All sessions were created before 2012-04-12.

````sql
SELECT 
  utm_source,
  utm_campaign,
  http_referer,
  COUNT(DISTINCT website_session_id) AS sessions
FROM website_sessions
WHERE created_at < '2012-04-12'
GROUP BY 
  utm_source, 
  utm_campaign, 
  http_referer
ORDER BY COUNT(DISTINCT website_session_id) DESC; 
````

#### Answer:
![solution1](https://user-images.githubusercontent.com/78378801/143680461-209d8cb8-2f8d-4472-87ae-bdeb8f02ebe4.jpg)

📌 As we see here, *gsearch/nonbrand* generates most sessions and this is our top one sourse of traffic. 
***

### 2. Calculate the conversion rate (CVR) from gsearch/nonbrand sessions to orders.
Note: All sessions were created before 2012-04-14. Based on what the company's paying for clicks, we'll need a CVR of 4% or more.

````sql
SELECT 
  COUNT(DISTINCT website_sessions.website_session_id) AS sessions,
  COUNT(DISTINCT orders.order_id) AS orders,
  COUNT(DISTINCT orders.order_id)/COUNT(DISTINCT website_sessions.website_session_id) AS session_to_order_conv_rate
FROM website_sessions
LEFT JOIN orders
ON website_sessions.website_session_id = orders.website_session_id
WHERE 
  website_sessions.created_at <  '2012-04-14' AND 
  website_sessions.utm_source = "gsearch" AND
  website_sessions.utm_campaign = "nonbrand";
````
In this case, I use *LEFT JOIN* because I need all website sessions and only those orders that match the website_session_id from the website_sessions table. 

#### Answer:
![solution2](https://user-images.githubusercontent.com/78378801/143682416-13b3fc03-ee8e-4bf2-bdb2-2dd5553af6cc.jpg)

📌 Conversion rate from gsearch/nonbrand sessions to orders is *2.88%* which is less than the company expected (4%). 
***

### 3. Calculate gsearch/nonbrand session volume, by week to see if there are any changes after bidding down. 
Note: All sessions were created before 2012-05-10. The company bid down gsearch nonbrand campaign on 2012-04-15.

````sql
SELECT
  MIN(DATE(created_at)) AS week_start_date,
  COUNT(DISTINCT website_session_id) AS sessions
FROM website_sessions
WHERE 
  created_at < '2012-05-10' AND 
  utm_source = "gsearch" AND
  utm_campaign = "nonbrand"
GROUP BY 
  YEAR(created_at),
  WEEK(created_at);
````

#### Answer:
![solution3](https://user-images.githubusercontent.com/78378801/143768110-1b6f5f07-cbab-4eb7-9442-86d93a700937.jpg)

📌 After bidding down on 2012-04-15, the volume of gsearch/nonbrand traffic started to decrease.
***

### 4. Calculate conversion rates from gsearch/nonbrand sessions to orders by device type.
Note: All sessions were created before 2012-05-11. 

````sql
SELECT 
website_sessions.device_type,
  COUNT(DISTINCT website_sessions.website_session_id) AS sessions,
  COUNT(DISTINCT orders.order_id) AS orders,
  COUNT(DISTINCT orders.order_id)/COUNT(DISTINCT website_sessions.website_session_id) AS conv_rt
FROM website_sessions
LEFT JOIN orders
ON website_sessions.website_session_id = orders.website_session_id
WHERE 
  website_sessions.created_at <  '2012-05-11' AND 
  website_sessions.utm_source = "gsearch" AND
  website_sessions.utm_campaign = "nonbrand"
GROUP BY ebsite_sessions.device_type;
````

#### Answer:
![solution4](https://user-images.githubusercontent.com/78378801/143779267-14f7deb1-e913-4e68-970b-139b88f2a225.jpg)

📌 Conversion rate for mobile much lower than for desktop.
***

### 5. Calculate weekly sessions for both desktio and mobile to see if there is an impact on traffic volume after bidding gsearch/nonbrabd desktop campaigns up on 2012-05-19.
Note: All sessions were created before 2012-06-09 and after 2012-04-15

````sql
SELECT 
  MIN(DATE(created_at)) AS week_start_date,
  COUNT(DISTINCT CASE WHEN device_type = 'desktop' THEN website_session_id ELSE NULL END) as dtop_sessions,
  COUNT(DISTINCT CASE WHEN device_type = 'mobile' THEN website_session_id ELSE NULL END) as mob_sessions
FROM website_sessions
WHERE 
  created_at < '2012-06-09' AND
  created_at > '2012-04-15' AND
  utm_source = "gsearch" AND
  utm_campaign = "nonbrand"
GROUP BY
  YEAR(created_at),
  WEEK(created_at);
````

#### Answer:
![solution5](https://user-images.githubusercontent.com/78378801/143780528-a92e7f10-e4f2-447b-b69c-a622f03814c4.jpg)

📌 Traffic volume increased after bidding gsearch/nonbrabd desktop campaigns up on 2012-05-19.

# Analyzing Website Performance
<img src="https://user-images.githubusercontent.com/78378801/144890243-2dc69f03-ac0f-470f-8fe9-1a6b85d7d2c6.jpg" alt="Image" width="700">

### 1. Find most-viewed website pages ranked by session volume. 
Note: All sessions were created before 2012-06-09.

````sql
SELECT
 pageview_url,
 COUNT(DISTINCT website_pageview_id) as pageviews
FROM website_pageviews
WHERE created_at < '2012-06-09'
GROUP BY pageview_url
ORDER BY pageviews DESC;
````

#### Answer:
![sol1](https://user-images.githubusercontent.com/78378801/144892155-e635f514-9a2c-4d0c-b635-b82e49235c74.jpg)

### 2. Find all entry pages and rank them on entry volume.
Note: All sessions were created before 2012-06-12.

First step - creating temp table:
````sql
CREATE TEMPORARY TABLE min_views
SELECT 
website_session_id,
MIN(website_pageview_id) AS min_pageview_id
FROM website_pageviews
WHERE created_at < '2012-06-12'
GROUP BY website_session_id;
````

Second step - joining tables and calculating session volume:
````sql
SELECT 
website_pageviews.pageview_url as landing_page,
COUNT(min_views.website_session_id) as sessions_hitting_this_landing_page
FROM min_views
LEFT JOIN website_pageviews
ON min_views.min_pageview_id = website_pageviews.website_pageview_id
GROUP BY website_pageviews.pageview_url;
````
#### Answer:
![sol2](https://user-images.githubusercontent.com/78378801/145713433-ee96ef4a-3177-4cf9-8d2c-bdfe19e54935.jpg)

📌 All traffic comes in through the homepage. 

### 3. Calculate bounce rates for traffic landing on the homepage with following columns: Sessions, Bounced Sessions, and % of Sessions which Bounced (aka “Bounce Rate”).
Note: All sessions were created before 2012-06-14.

First step - find min pageview id:
````sql
CREATE TEMPORARY TABLE first_pgv_id
SELECT
	a.website_session_id,
  MIN(website_pageview_id) AS first_pgv
FROM website_pageviews AS a
INNER JOIN website_sessions AS b
ON a.website_session_id = b.website_session_id AND
  b.created_at < '2012-06-14'
GROUP BY a.website_session_id;
````
Second step - find the pageview_url of the first pageview id:
````sql
CREATE TEMPORARY TABLE sessions_with_landing_page
SELECT
	first_pgv_id.website_session_id,
    website_pageviews.pageview_url AS landing_page
FROM first_pgv_id
LEFT JOIN website_pageviews
ON first_pgv_id.first_pgv = website_pageviews.website_pageview_id;
````
Third step - find number of bounced session:
````sql
CREATE TEMPORARY TABLE bounced_sessions
SELECT
	sessions_with_landing_page.website_session_id, 
    sessions_with_landing_page.landing_page,
    COUNT(website_pageviews.website_pageview_id) AS pages_viewed
FROM sessions_with_landing_page
LEFT JOIN website_pageviews
ON sessions_with_landing_page.website_session_id = website_pageviews.website_session_id
GROUP BY 
	sessions_with_landing_page.website_session_id, 
	sessions_with_landing_page.landing_page
HAVING COUNT(website_pageviews.website_pageview_id) = 1;
````
Fourth step - count all and bounced sessions, calculate bounce rate:
````sql
SELECT 
	sessions_with_landing_page.landing_page,
	COUNT(DISTINCT sessions_with_landing_page.website_session_id) AS sessions,
  COUNT(DISTINCT bounced_sessions.website_session_id) AS bounced_sessions,
  COUNT(DISTINCT bounced_sessions.website_session_id)/COUNT(DISTINCT sessions_with_landing_page.website_session_id) as bounce_rate
FROM sessions_with_landing_page
LEFT JOIN bounced_sessions
ON sessions_with_landing_page.website_session_id = bounced_sessions.website_session_id
GROUP BY sessions_with_landing_page.landing_page;
````
#### Answer:
![sol3](https://user-images.githubusercontent.com/78378801/145859139-c9ca5f64-d9fc-4f9f-a816-2dccbd57c338.jpg)

📌 Bounce rate is almost 60% which is pretty high.
