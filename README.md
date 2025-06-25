# Website analysis - eCommerce - SQL
Author: Nguyễn Hải Long 
Date: 2025-02  
Tools Used: SQL 

---

## 📑 Table of Contents  
1. [📌 Background & Overview](#-background--overview)  
2. [📂 Dataset Description & Data Structure](#-dataset-description--data-structure)  
3. [🔎 Final Conclusion & Recommendations](#-final-conclusion--recommendations)

---

## 📌 Background & Overview  

### Objective:
### 📖 This project is about using SQL to analyze transaction data from Google analytic dataset.

- Show up data as demand: number of pageview, visits; calculate the average of money spending by customers in a period of time...
- 

### 👤 Who is this project for?  

- Data analysts & business analysts.
- Decision-makers.

---

## 📂 Dataset Description & Data Structure  

### 📌 Data Source  
- Source: Google analytic database.
- Size: The dataset has 338 columns. In this project, using only 17 columns

### 📊 Data Structure & Relationships  
Table using in this project:  

| Field Name | Data Type | Description |
|------------|-----------|-------------|
| fullVisitorId | STRING | The unique visitor ID. |
| date | STRING | The date of the session in YYYYMMDD format. |
| totals | RECORD | This section contains aggregate values across the session. |
| totals.bounces | INTEGER | Total bounces (for convenience). For a bounced session, the value is 1, otherwise it is null. |
| totals.hits | INTEGER | Total number of hits within the session. |
| totals.pageviews | INTEGER | Total number of pageviews within the session. |
| totals.visits | INTEGER | The number of sessions (for convenience). This value is 1 for sessions with interaction events. The value is null if there are no interaction events in the session. |
| totals.transactions | INTEGER | Total number of ecommerce transactions within the session. |
| trafficSource.source | STRING | The source of the traffic source. Could be the name of the search engine, the referring hostname, or a value of the utm_source URL parameter. |
| hits | RECORD | This row and nested fields are populated for any and all types of hits. |
| hits.eCommerceAction | RECORD| This section contains all of the ecommerce hits that occurred during the session. This is a repeated field and has an entry for each hit that was collected.|
| hits.eCommerceAction.action_type | STRING | "The action type. Click through of product lists = 1, Product detail views = 2, Add product(s) to cart = 3, Remove product(s) from cart = 4, Check out = 5, Completed purchase = 6, Refund of purchase = 7, Checkout options = 8, Unknown = 0. Usually this action type applies to all the products in a hit, with the following exception: when hits.product.isImpression = TRUE, the corresponding product is a product impression that is seen while the product action is taking place (i.e., a ""product in list view"")." |
| hits.product | RECORD | This row and nested fields will be populated for each hit that contains Enhanced Ecommerce PRODUCT data. |
| hits.product.productQuantity | INTEGER | The quantity of the product purchased. |
| hits.product.productRevenue | INTEGER | The revenue of the product, expressed as the value passed to Analytics multiplied by 10^6 (e.g., 2.40 would be given as 2400000). |
| hits.product.productSKU | STRING | Product SKU. |
| hits.product.v2ProductName | STRING | Product Name. |


---

## ⚒️ Main Process

1️⃣ Calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)  
 ```
SELECT
    FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d',date)) month
    ,SUM(totals.visits) visits
    ,SUM(totals.pageviews) pageviews
    ,SUM(totals.transactions) transactions
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
WHERE _table_suffix BETWEEN '0101' AND '0331'
GROUP BY date
;
```
### Result:  
![result_query_1](https://github.com/longnguyen0102/photo/blob/main/eCommerce_project/sql_ecommerce_query01_result.png)

2️⃣ Bounce rate per traffic source in July 2017 (Bounce_rate = num_bounce/total_visit) (order by total_visit DESC)
Bounce session is the session that user does not raise any click after landing on the website
```
WITH sum_visits_and_bounces AS(
  SELECT
    trafficSource.source
    ,SUM(totals.visits) total_visits
    ,SUM(totals.bounces)total_no_of_bounces
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
  GROUP BY trafficSource.source
)
SELECT
  *
  ,ROUND((total_no_of_bounces / total_visits)*100.0, 3) bounce_rate
FROM sum_visits_and_bounces
ORDER BY total_visits DESC
```
### Result:
![result_query_2](https://github.com/longnguyen0102/photo/blob/main/eCommerce_project/sql_ecommerce_query02_result.png)

3️⃣ Revenue by traffic source by week, by month in June 2017
```
WITH data_with_date AS(
  SELECT 
  PARSE_DATE('%Y%m%d',date) date_format
  ,trafficSource.source
  ,productRevenue
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
  UNNEST (hits) hits,
  UNNEST (hits.product) product
  WHERE productRevenue IS NOT NULL
),
revenue_month AS(
  SELECT
    'Month' time_type
    ,FORMAT_DATE('%Y%m',date_format) time
    ,source
    ,ROUND((SUM(productRevenue) / 1000000),4) revenue
  FROM data_with_date
  GROUP BY time, source
),
revenue_week AS(
  SELECT
    'Week' time_type
    ,FORMAT_DATE('%Y%W', date_format) time
    ,source
    ,ROUND((SUM(productRevenue) / 1000000),4) revenue
  FROM data_with_date
  GROUP BY time, source
)
SELECT * FROM revenue_month
UNION DISTINCT
SELECT * FROM revenue_week
ORDER BY source, revenue DESC
```
### Result:
![result_query_3](https://github.com/longnguyen0102/photo/blob/main/eCommerce_project/sql_ecommerce_query03_result.png)

4️⃣ Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017
```
WITH get_data AS(
  SELECT
    fullVisitorId
    ,FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d',date)) month
    ,totals.transactions
    ,productRevenue
    ,totals.pageviews
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
  UNNEST (hits) hits,
  UNNEST (hits.product) product
  WHERE _table_suffix BETWEEN '0601' AND '0731'
)
,avg_purchasers AS(
SELECT
  month
  ,SUM (pageviews) / COUNT(DISTINCT fullVisitorId) avg_pageviews_purchase
FROM get_data
WHERE transactions >= 1 AND productRevenue IS NOT NULL
GROUP BY month
)
,avg_non_purchasers AS(
SELECT
  month
  ,SUM (pageviews) / COUNT(DISTINCT fullVisitorId) avg_pageviews_non_purchase
FROM get_data
WHERE transactions IS NULL AND productRevenue IS NULL
GROUP BY month
)
SELECT * FROM avg_purchasers
LEFT JOIN avg_non_purchasers
USING(month)
ORDER BY month
```
### Result:
![result_query_4](https://github.com/longnguyen0102/photo/blob/main/eCommerce_project/sql_ecommerce_query04_result.png)

- First, explain codes' purpose - what they do

- Then how your query/ code & Insert screenshots of your result

- Finally, explain your observations/ findings from the results  ts findings
  
 _Describe trends, key metrics, and patterns._  

---

## 🔎 Final Conclusion & Recommendations  

👉🏻 Based on the insights and findings above, we would recommend the [stakeholder team] to consider the following:  

📌 Key Takeaways:  
✔️ Recommendation 1  
✔️ Recommendation 2  
✔️ Recommendation 3
