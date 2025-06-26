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

### 👤 Who is this project for?  

- Data analysts & business analysts.  
- Decision-makers.

---

## 📂 Dataset Description & Data Structure  

### 📌 Data Source  
- Source: Google analytic database.
- Size: The dataset has 338 columns. In this project, using only 17 columns.

### 📊 Data Structure & Relationships  
<details>
 <summary>Table using in this project:</summary>

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

</details>

---

## ⚒️ Main Process

1️⃣ Calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month).  
<details>
 <summary>Code:</summary>
 
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
</details>  
Result:  
![result_query_1](https://github.com/longnguyen0102/photo/blob/main/eCommerce_project/sql_ecommerce_query01_result.png)  

2️⃣ Bounce rate per traffic source in July 2017.  
(Bounce_rate = num_bounce/total_visit) (order by total_visit DESC).  
*Note: Bounce session is the session that user does not raise any click after landing on the website*  
<details>
 <summary>Code:</summary>
 
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
;
```
</details>  
Result:  
*Note: Because the result has many column, so the image shows about first 20 columns.*  
![result_query_2](https://github.com/longnguyen0102/photo/blob/main/eCommerce_project/sql_ecommerce_query02_result.png)  

3️⃣ Revenue by traffic source by week, by month in June 2017.  
<details>
 <summary>Code:</summary>
 
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
;
```
</details>  
Result:  
*Note: Because the result has many column, so the image shows about first 20 columns.*  
![result_query_3](https://github.com/longnguyen0102/photo/blob/main/eCommerce_project/sql_ecommerce_query03_result.png)  

4️⃣ Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017.  
*Note: fullVisitorId field is user id.*  
<details>
 <summary>Code:</summary>
 
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
;
```
</details>  
Result:  
![result_query_4](https://github.com/longnguyen0102/photo/blob/main/eCommerce_project/sql_ecommerce_query04_result.png)  

5️⃣ Average number of transactions per user that made a purchase in July 2017.  
<details>
 <summary>Code:</summary>
 
```
WITH get_data AS(
  SELECT
    FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d',date)) month
    ,fullVisitorId
    ,totals.transactions
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
  UNNEST (hits) hits,
  UNNEST (hits.product) product
  WHERE totals.transactions >= 1 AND productRevenue IS NOT NULL
)
,total_transactions_purchaser AS(
  SELECT
    month
    ,COUNT(DISTINCT fullVisitorId) count_visitor
    ,SUM(transactions) sum_transactions
  FROM get_data
  GROUP BY month
)
SELECT
  month
  ,(sum_transactions / count_visitor) avg_total_transactions_per_user
FROM total_transactions_purchaser
;
```
</details>  
Result:  
![result_query_5](https://github.com/longnguyen0102/photo/blob/main/eCommerce_project/sql_ecommerce_query05_result.png)  

6️⃣ Average amount of money spent per session. Only include purchaser data in July 2017.    
*Note: Condition of purchaser: transactions >=1 and productRevenue IS NOT NULL.*  
<details>
 <summary>Code:</summary>
 
```
WITH get_data AS(
    SELECT
        FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d',date)) month
        ,totals.transactions
        ,totals.visits
        ,productRevenue
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
    UNNEST (hits) hits,
    UNNEST (hits.product) product
    WHERE totals.transactions IS NOT NULL AND productRevenue IS NOT NULL
)
,sum_revenue_and_visit AS(
    SELECT
        month
        ,SUM(productRevenue) / 1000000 sum_revenue
        ,SUM(visits) sum_visit
    FROM get_data
    GROUP BY month
)
SELECT
    month
    ,ROUND((sum_revenue / sum_visit),2) avg_revenue_by_user_per_visit
FROM sum_revenue_and_visit
;
```
</details>  
Result:  
![result_query_6](https://github.com/longnguyen0102/photo/blob/main/eCommerce_project/sql_ecommerce_query06_result.png)  

7️⃣ Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017.  
*Output should show product name and the quantity was ordered.*  
<details>
 <summary>Code:</summary>
 
```
/*filter purchaser*/
WITH vintage_purchasers AS(
  SELECT
    fullVisitorId
    ,product.v2ProductName
    ,product.productQuantity
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
  UNNEST (hits) hits,
  UNNEST (hits.product) product
  WHERE productRevenue IS NOT NULL AND productQuantity IS NOT NULL AND product.v2ProductName = "YouTube Men's Vintage Henley"
)
  SELECT
    product.v2ProductName other_purchased_products
    ,SUM(product.productQuantity) sumup
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
  UNNEST (hits) hits,
  UNNEST (hits.product) product
  WHERE productRevenue IS NOT NULL AND productQuantity IS NOT NULL AND fullVisitorId IN (SELECT fullVisitorId FROM vintage_purchasers)
    AND product.v2ProductName <> "YouTube Men's Vintage Henley"
  GROUP BY other_purchased_products
  ORDER BY sumup DESC
  ;
```
</details>  
Result:  
*Note: Because the result has many column, so the image shows about first 20 columns.*  
![result_query_7](https://github.com/longnguyen0102/photo/blob/main/eCommerce_project/sql_ecommerce_query07_result.png)  

8️⃣ Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017.  
For example, 100% product view then 40% add_to_cart and 10% purchase.  
*Note: Add_to_cart_rate = number product  add to cart/number product view.  
Purchase_rate = number product purchase/number product view.  
The output should be calculated in product level.* 
<details>
 <summary>Code:</summary>
 
```
WITH filtered_data AS(
  SELECT
    FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d',date)) month
    ,CAST(hits.eCommerceAction.action_type AS INT64) AS action_type
    ,product.productRevenue
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
  UNNEST (hits) hits,
  UNNEST (hits.product) product
  WHERE _table_suffix BETWEEN '0101' AND '0331'
  ORDER BY action_type
)
,product_view AS(
  SELECT
    month
    ,COUNT(action_type) num_product_view
  FROM filtered_data
  WHERE action_type = 2
  GROUP BY month
)
,add_to_cart AS(
  SELECT
    month
    ,COUNT(action_type) num_addtocart
  FROM filtered_data
  WHERE action_type = 3
  GROUP BY month
)
,purchase AS(
  SELECT
    month
    ,COUNT(action_type) num_purchase
  FROM filtered_data
  WHERE action_type = 6 AND productRevenue IS NOT NULL
  GROUP BY month
)
,all_data_needed AS(
  SELECT * FROM product_view
  JOIN add_to_cart
  USING(month)
  JOIN purchase
  USING(month)
  ORDER BY month
)
SELECT 
  *
  ,ROUND((num_addtocart / num_product_view)*100,2) add_to_cart_rate
  ,ROUND((num_purchase / num_product_view)*100,2) purchase_rate
FROM all_data_needed
;
```
</details>  
Result:  
![result_query_8](https://github.com/longnguyen0102/photo/blob/main/eCommerce_project/sql_ecommerce_query08_result.png)  

## 📌 Key Takeaways:  
✔️ Understanding the basics of SQL query.  
✔️ Know how to apply Window Functions when writing queries.  
✔️ Understanding real-world requirements when using SQl to retrieve neceesary data
