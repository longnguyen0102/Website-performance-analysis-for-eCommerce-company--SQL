# Website analysis - eCommerce - SQL

Please click on the link below to see the code:
https://console.cloud.google.com/bigquery?sq=690384817981:f2ba7cdb31be4548849a4ad57d91170f

---
![KPMG Transaction Analysis](https://github.com/Dorothy-Ho-Vy/Sample_SQL_Python_template/blob/4dee6ff56077b90b1aea82e8517136f7185a77a3/Blue%20White%20Modern%20Payment%20Gateway%20Service%20Twitter%20Post.png.crdownload)

👉🏻Change Icon emoji 🔥🔍📘🚩 to your likings by clicking "Start" + "."

# 📊 Website analysis - eCommerce - SQL
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
| Field Name  |Data Type|Description|
| :—––––––––: | :———– | :————– |
|fullVisitorId|STRING|The unique visitor ID.|
|date|STRING|The date of the session in YYYYMMDD format.|
|totals|RECORD|This section contains aggregate values across the session.|
|totals.bounces|INTEGER|Total bounces (for convenience). For a bounced session, the value is 1, otherwise it is null.|
|totals.hits|INTEGER|Total number of hits within the session.|
|totals.pageviews|INTEGER|Total number of pageviews within the session.|
|totals.visits|INTEGER|The number of sessions (for convenience). This value is 1 for sessions with interaction events. The value is null if there are no interaction events in the session.|
|totals.transactions|INTEGER|Total number of ecommerce transactions within the session.|
|trafficSource.source|STRING|The source of the traffic source. Could be the name of the search engine, the referring hostname, or a value of the utm_source URL parameter.|
|hits|RECORD|This row and nested fields are populated for any and all types of hits.|
|hits.eCommerceAction|RECORD|This section contains all of the ecommerce hits that occurred during the session. This is a repeated field and has an entry for each hit that was collected.|
|hits.eCommerceAction.action_type|STRING|"The action type. Click through of product lists = 1, Product detail views = 2, Add product(s) to cart = 3, Remove product(s) from cart = 4, Check out = 5, Completed purchase = 6, Refund of purchase = 7, Checkout options = 8, Unknown = 0.  
Usually this action type applies to all the products in a hit, with the following exception: when hits.product.isImpression = TRUE, the corresponding product is a product impression that is seen while the product action is taking place (i.e., a ""product in list view"")."|
|hits.product|RECORD|This row and nested fields will be populated for each hit that contains Enhanced Ecommerce PRODUCT data.|
|hits.product.productQuantity|INTEGER|The quantity of the product purchased.|
|hits.product.productRevenue|INTEGER|The revenue of the product, expressed as the value passed to Analytics multiplied by 10^6 (e.g., 2.40 would be given as 2400000).|
|hits.product.productSKU|STRING|Product SKU.|
|hits.product.v2ProductName|STRING|Product Name.|


| STT | Cột 1 | Cột 2 |
| :—– | :———- | :————– |
| 1 | Dòng 11 | Dòng 21 |
| 2 | Dòng 12 | Dòng 22 |
| 3 | Dòng 13 | Dòng 23 |
| 4 | Dòng 14 | Dòng 24 |

#### 1️⃣ Tables Used:  
Mention how many tables are in the dataset.  

#### 2️⃣ Table Schema & Data Snapshot  

Table 1: Products Table  

👉🏻 Insert a screenshot of table schema 

📌If the table is too big, only capture a part of it that contains key metrics you used in the projects or put the table in toggle

 _Example:_

| Column Name | Data Type | Description |  
|-------------|----------|-------------|  
| Product_ID  | INT      | Unique identifier for each product |  
| Name        | TEXT     | Product name |  
| Category    | TEXT     | Product category |  
| Price       | FLOAT    | Price per unit |  


Table 2: Sales Transactions  

👉🏻 Insert a screenshot of table schema.


---

## ⚒️ Main Process

1️⃣ Data Cleaning & Preprocessing  
2️⃣ Exploratory Data Analysis (EDA)  
3️⃣ SQL/ Python Analysis 

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
