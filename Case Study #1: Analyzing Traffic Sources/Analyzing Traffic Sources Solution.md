# Case Study #1: Analyzing Traffic Sources

## Solution

***

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
