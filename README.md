
#  🏷️ Promotion Effectiveness Analysis using SQL

This repository contains SQL scripts to analyze **sales promotions** conducted by **AtliQ Mart**, a leading FMCG retail chain in Southern India. The project evaluates the effectiveness of promotional campaigns during **Diwali 2023** and **Sankranti 2024**, providing actionable insights for the Sales Director to optimize future campaigns.



#  Introduction

Promotions play a critical role in driving sales, especially during festive seasons in India. AtliQ Mart, with **50 supermarkets** across Southern India, launched multiple promotional campaigns on its **AtliQ branded products**.

The **Sales Director (Bruce Haryali)** wants to understand which promotions worked well and which did not. Since this analysis will directly impact future promotional strategies, it was assigned to **Peter Pandey (Data Analyst)** under the guidance of the analytics manager **Tony**.

The objective is to use SQL queries to:

* Identify high-value products under promotions.
* Measure revenue impact before and after campaigns.
* Rank categories and products by incremental performance.
* Highlight cities and stores driving retail presence.


#  📂 Data Sources

The analysis uses AtliQ Mart’s **Retail Sales Database**:

* **fact\_sales** → Sales transactions (before & after promotion).
* **dim\_products** → Product details (name, category, segment).
* **dim\_stores** → Store details (store\_id, city).
* **dim\_campaigns** → Campaign details (Diwali, Sankranti, etc.).



#  📌 Business Requests

### 1️⃣ High-Value Products under BOGOF

**Objective:** List products with a **base price > 500** featured in **BOGOF (Buy One Get One Free)** promotions.
This helps identify high-value products that are heavily discounted.

```sql
SELECT P.product_name, S.base_price 
FROM [retail].[fact_sales] AS S 
LEFT JOIN [retail].[dim_products] AS P
    ON S.product_code = P.product_code 
WHERE S.base_price > 500 AND S.promo_type = 'BOGOF'
GROUP BY P.product_name, S.base_price;
```

---

### 2️⃣ Store Presence by City

**Objective:** Count the number of stores in each city (sorted in descending order).
This helps in **retail network optimization**.

```sql
SELECT city, COUNT(store_id) AS Store_count 
FROM [retail].[dim_stores]
GROUP BY city
ORDER BY Store_count DESC;
```

---

### 3️⃣ Campaign Revenue Impact

**Objective:** Compare **revenue before vs after promotions** for each campaign (in millions).
This shows which campaigns generated the maximum lift.

```sql
SELECT C.campaign_name, 
       CONCAT(ROUND(SUM(base_price * quantity_sold_before_promo)/1000000,2),' M') AS total_revenue_before_promo,
       CONCAT(ROUND(SUM(base_price * quantity_sold_after_promo)/1000000,2),' M') AS total_revenue_after_promo
FROM [retail].[fact_sales] AS S 
LEFT JOIN [retail].[dim_campaigns] AS C
    ON S.campaign_id = C.campaign_id
GROUP BY C.campaign_name;
```

---

### 4️⃣ Category-Wise Incremental Sold Quantity (ISU%) – Diwali

**Objective:** Calculate **ISU%** for each category during **Diwali campaign (CAMP\_DIW\_01)** and rank them.

```sql
WITH mycte AS (
    SELECT P.category, 
           SUM(S.quantity_sold_before_promo) AS Total_unit_sold_before, 
           SUM(S.quantity_sold_after_promo) AS Total_unit_sold_after
    FROM [retail].[fact_sales] AS S 
    LEFT JOIN [retail].[dim_products] AS P
        ON S.product_code = P.product_code
    WHERE S.campaign_id = 'CAMP_DIW_01'
    GROUP BY P.category
)
SELECT *, 
       ((Total_unit_sold_after - Total_unit_sold_before)/Total_unit_sold_before)*100 AS ISU%,
       DENSE_RANK() OVER (ORDER BY ((Total_unit_sold_after - Total_unit_sold_before)/Total_unit_sold_before)*100 DESC) AS ranking
FROM mycte;
```

---

### 5️⃣ Top 5 Products by Incremental Revenue (IR%)

**Objective:** Rank products by **Incremental Revenue % (IR%)** across all campaigns.

```sql
WITH mycte AS (
    SELECT P.product_name, P.category, 
           SUM(base_price * quantity_sold_before_promo) AS Revenue_before_promo, 
           SUM(base_price * quantity_sold_after_promo) AS Revenue_after_promo
    FROM [retail].[fact_sales] AS S 
    LEFT JOIN [retail].[dim_products] AS P
        ON S.product_code = P.product_code
    GROUP BY P.product_name, P.category
)
SELECT product_name, category,
       ((Revenue_after_promo - Revenue_before_promo)/Revenue_before_promo)*100 AS IR%
FROM mycte
ORDER BY IR% DESC
OFFSET 0 ROWS FETCH NEXT 5 ROWS ONLY;
```


#  ⚠️ Limitations

* Analysis limited to **historical campaigns** (Diwali & Sankranti).
* External market conditions (competitor offers, inflation) not included.
* No customer-level segmentation (loyalty, demographics).



#  📈 Results & Insights

* Identified **high-value products** impacted by heavy BOGOF promotions.
* Store distribution insights show cities with **highest AtliQ Mart presence**.
* Campaign revenue reports highlight **financial impact pre vs post promotions**.
* ISU% ranking revealed **categories with strongest festive sales lift**.
* Top products driving **incremental revenue growth** were identified.


#  🚀 Future Work

* Add **customer segmentation** (new vs repeat buyers).
* Evaluate **promotion ROI** with cost vs incremental revenue.
* Use **predictive modeling** for future festive campaigns.
* Develop **Power BI Dashboard** for interactive insights.


