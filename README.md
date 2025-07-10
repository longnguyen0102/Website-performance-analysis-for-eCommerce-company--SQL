# Website performance analysis for eCommerce company using SQL
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

(sửa lại phần này)  

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

*Note: Click the white triangle to see codes*  

### 1️⃣ Calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month).  
<details>
 <summary>Code:</summary>
 
 ```sql
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

![result_query_1](https://github.com/longnguyen0102/photo/blob/main/eCommerce_project/sql_ecommerce_query01_result.png)  

➡️ February 2017 had the least number of visits and pageviews; however transactions number was higher than January 2017. March 2017 had the highest number of the first quarter of 2017. Maybe the reason behind was January and February are after Holiday season so custoners did not want to spend more money in purchasing.  

### 2️⃣ Bounce rate per traffic source in July 2017.  
(Bounce_rate = num_bounce/total_visit) (order by total_visit DESC).  
*Note: Bounce session is the session that user does not raise any click after landing on the website*  
<details>
 <summary>Code:</summary>
 
```sql
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

*Note: Because the result has many column, so the image shows about first 20 columns.*  

![result_query_2](https://github.com/longnguyen0102/photo/blob/main/eCommerce_project/sql_ecommerce_query02_result.png)  

➡️ The order of result is decending according to the total_visits. Google had the most visits and the bounce rate was about 51.5%. l.facebook.com had the most bounce_rate (88.235%) in the first 20 results, this number indicated that users tended to scroll for news feed rather that interact with the website.  

### 3️⃣ Revenue by traffic source by week, by month in June 2017.  
<details>
 <summary>Code:</summary>
 
```sql
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

*Note: Because the result has many column, so the image shows about first 20 columns.*  

![result_query_3](https://github.com/longnguyen0102/photo/blob/main/eCommerce_project/sql_ecommerce_query03_result.png)  

➡️ In June 2017, most revenue was from unknown source (these are from users' action or Google Analytics cannot track the origin of the visit).  

### 4️⃣ Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017.  
*Note: fullVisitorId field is user id.*  
<details>
 <summary>Code:</summary>
 
```sql
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

![result_query_4](https://github.com/longnguyen0102/photo/blob/main/eCommerce_project/sql_ecommerce_query04_result.png)  

➡️ Average number pageviews of non-purchasers in June and July was about three times bigger than purchasers. However, in each month, the number pageviews of purchasers increased (~32%) and in non-purchasers was ~5.4%. These numbers indicated that the UI/UX of website or any marketing campaign worked during June and July of 2017.

### 5️⃣ Average number of transactions per user that made a purchase in July 2017.  
<details>
 <summary>Code:</summary>
 
```sql
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

![result_query_5](https://github.com/longnguyen0102/photo/blob/main/eCommerce_project/sql_ecommerce_query05_result.png)  

➡️ As can see from the result, this number in July 2017 was high. Each customers made 4 transactions; they did not buy one time then leave, they came back to make more transactions in a month.  

### 6️⃣ Average amount of money spent per session. Only include purchaser data in July 2017.    
*Note: Condition of purchaser: transactions >=1 and productRevenue IS NOT NULL.*  
<details>
 <summary>Code:</summary>
 
```sql
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

![result_query_6](https://github.com/longnguyen0102/photo/blob/main/eCommerce_project/sql_ecommerce_query06_result.png)  

➡️ Revenue made from an user is 43.86 in July. If the cost to bring a new user to the platform is lower than 43.86, the company is making profit.  

### 7️⃣ Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017.  
*Output should show product name and the quantity was ordered.*  
<details>
 <summary>Code:</summary>
 
```sql
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

*Note: Because the result has many column, so the image shows about first 20 columns. 'sumup' is the total number of purchased product.*  

![result_query_7](https://github.com/longnguyen0102/photo/blob/main/eCommerce_project/sql_ecommerce_query07_result.png)  

➡️ Google Sunglasses was the most selling item to people who purchased "Youtube Men's Vintage Henley". When looking at the following items, we can see users tended to buy fashion products or accessories.

### 8️⃣ Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017.  
For example, 100% product view then 40% add_to_cart and 10% purchase.  
*Note: Add_to_cart_rate = number product  add to cart/number product view.  
Purchase_rate = number product purchase/number product view.  
The output should be calculated in product level.* 
<details>
 <summary>Code:</summary>
 
```sql
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

![result_query_8](https://github.com/longnguyen0102/photo/blob/main/eCommerce_project/sql_ecommerce_query08_result.png)  

➡️ The numbers of 'Add to cart', 'Purchases' increased from January to March. The reason behind maybe the improvement of website UI/UX, or marketing campaigns hit the "right spot" of users.
➡️ The add-to-cart rate of the first 3 months of 2017 increased (from 28.47% to 37.29%). From February to March, we can see the purchase rate increased dramatically from 9.59% to 12.64%.  

## 📌 Key Takeaways:  
✔️ Understanding the basics of SQL query: ```SUM```, ```GROUP BY```, ```ORDER BY```, ```COUNT``` syntax.  
✔️ Applying Window Functions in query writing helps make the code more concise and readable.   
✔️ Understanding how to use ```JOIN``` syntax correctly is essential to ensure accurate results and achieve the most effective data combinations.  
✔️ The importance of date transformation in data extraction lies in enabling accurate filtering, aggregation, and trend analysis over time. Proper date handling ensures that time-based insights are reliable and actionable.  
✔️ By using BigQuery, Understanding of ```UNNEST``` syntax in extracting the right amount of data.  
✔️ Understanding practical data requirements enables you to extract the right data, leading to focused and actionable insights.
