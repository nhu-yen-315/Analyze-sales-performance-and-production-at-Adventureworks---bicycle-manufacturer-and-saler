# Sales analysis in Adventure Works Cycles - a bicycle manufacturer with SQL

Author: Huỳnh Như Yến  
Date: 9-2025 <br>
Tool Used: SQL

---
## 📑 Table of Contents  
1. 📌 Background & Overview 
2. 📂 Dataset Description
3. 🔎 Queries & Insights
4. 💡 Recommendations

---
## 1. 📌 Background & Overview
### 📖 What is this project about? 


### 👤 Who is this project for?  
- Sales managers 
- Data analysts 
- Business analysts 

---
## 2. 📂 Dataset Description 

---
## 3. 🔎 Queries & Insights

#### ⚒ Query 1: Which subcategories contribute most to the total revenue by year?
```sql
WITH sales AS  
  (SELECT 
      FORMAT_DATE('%Y', sales.ModifiedDate) AS year,
      subcat.Name AS subcategory,
      ROUND(SUM(LineTotal),2) AS total_sales,
      SUM(OrderQty) AS item_qty,
      COUNT(DISTINCT SalesOrderID) AS order_qty,
  FROM `adventureworks2019.Sales.SalesOrderDetail` AS sales
  LEFT JOIN `adventureworks2019.Production.Product` AS product 
      ON sales.ProductID = product.ProductID
  LEFT JOIN `adventureworks2019.Production.ProductSubcategory` AS subcat
      ON CAST(product.ProductSubcategoryID AS int) = subcat.ProductSubcategoryID
  WHERE CAST(FORMAT_DATE('%Y', sales.ModifiedDate) AS int) IN (2012, 2013)
  GROUP BY 1,2
)

SELECT * FROM 
  (SELECT *
        , DENSE_RANK() OVER(PARTITION BY year ORDER BY total_sales DESC) AS ranked
  FROM sales)
WHERE ranked IN (1,2,3)
ORDER BY year
```

#### 👉🏻 Results:
<p align='center'>
      <img width="677" height="187" alt="image" src="https://github.com/user-attachments/assets/5cfd1000-15ca-4410-bd66-07cee80e3d4f" />
</p>

#### 🔎 Insights:
- **Bikes** and **bicycle frames** are two **important product categories**, contributing most to total revenue.
- **Road Bikes** rank **#1** in **2012 and 2013**. However, sales **drop** from **$17.1M** in 2012 to **$14.8M** in 2013.
- **Mountain Bikes** rank **#2** in both years and experience an **increase in sales** from **$11.7M to $13M**.
- No presence in 2012, but **Touring Bikes** emerge to the top list in 2013 with **$8.4M**. **Road Frames** is **out of top 3** in 2013, suggesting a **decline in demand** for only-frame products.

#### ⚒ Query 2: What subcategories have the highest or lowest YoY growth rates based on quantities sold?
```sql
WITH quantity AS   
  (SELECT 
        FORMAT_DATE('%Y', sales.ModifiedDate) AS year,
        subcat.Name AS subcategory,
        SUM(OrderQty) AS item_qty
    FROM `adventureworks2019.Sales.SalesOrderDetail` AS sales
    LEFT JOIN `adventureworks2019.Production.Product` AS product 
        ON sales.ProductID = product.ProductID
    LEFT JOIN `adventureworks2019.Production.ProductSubcategory` AS subcat
        ON CAST(product.ProductSubcategoryID AS int) = subcat.ProductSubcategoryID
    WHERE CAST(FORMAT_DATE('%Y', sales.ModifiedDate) AS int) IN (2012, 2013)
    GROUP BY 1,2)

SELECT *,
        (item_qty - prev_item_qty)*100/prev_item_qty AS YoY_gr
FROM 
    (SELECT year,
            subcategory,
            item_qty,
            LAG(item_qty) OVER(PARTITION BY subcategory ORDER BY year) AS prev_item_qty
    FROM quantity)
WHERE year = '2013'
ORDER BY YoY_gr DESC;
```
#### 👉🏻 Results:
<p align='center'>
  <img width="672" height="270" alt="image" src="https://github.com/user-attachments/assets/3d6a34d3-9aa3-4f14-bbdc-dd6fbcc93b2c" />

  <img width="672" height="269" alt="image" src="https://github.com/user-attachments/assets/5744f05f-ecbc-41b2-bc77-036fbb3f0f4d" />

</p>

#### 🔎 Insights:

#### ⚒ Query 3: Which territories have the highest sales based on quantities sold?
```sql
WITH sales_quantity AS     
    (SELECT 
            FORMAT_DATE('%Y', sales.ModifiedDate) AS year,
            territory.Name AS territory,
            SUM(OrderQty) AS item_qty
    FROM `adventureworks2019.Sales.SalesOrderDetail` AS sales
    LEFT JOIN `adventureworks2019.Sales.SalesOrderHeader`AS header 
        ON sales.SalesOrderID = header.SalesOrderID
    LEFT JOIN `adventureworks2019.Sales.SalesTerritory` AS territory
        ON header.TerritoryID = territory.TerritoryID 
    WHERE FORMAT_DATE('%Y', sales.ModifiedDate) IN ('2012','2013')
    GROUP BY 1,2)

SELECT * FROM
    (SELECT *,
            DENSE_RANK() OVER(PARTITION BY year ORDER BY item_qty DESC) AS ranked
    FROM sales_quantity)
WHERE ranked IN (1,2,3);
```
#### 👉🏻 Results:
<p align='center'>
  <img width="511" height="190" alt="image" src="https://github.com/user-attachments/assets/4187f71c-2b3b-482a-afea-8ae3b6d66522" />
</p>

#### 🔎 Insights:

#### ⚒ Query 4: Which territories have the fastest YoY growth rates based on quantities sold?

```sql
WITH sales_quantity AS     
    (SELECT 
            FORMAT_DATE('%Y', sales.ModifiedDate) AS year,
            territory.Name AS territory,
            SUM(OrderQty) AS item_qty
    FROM `adventureworks2019.Sales.SalesOrderDetail` AS sales
    LEFT JOIN `adventureworks2019.Sales.SalesOrderHeader`AS header 
        ON sales.SalesOrderID = header.SalesOrderID
    LEFT JOIN `adventureworks2019.Sales.SalesTerritory` AS territory
        ON header.TerritoryID = territory.TerritoryID 
    WHERE FORMAT_DATE('%Y', sales.ModifiedDate) IN ('2012','2013')
    GROUP BY 1,2)

SELECT *,
       ROUND((item_qty - prev_item_qty)*100/prev_item_qty, 2) AS YoY_gr 
FROM
    (SELECT *,
            LAG(item_qty) OVER(PARTITION BY territory ORDER BY year) AS prev_item_qty
    FROM sales_quantity)
WHERE year = '2013'
ORDER BY YoY_gr DESC;
```
#### 👉🏻 Results:
<p align='center'>
  <img width="681" height="296" alt="image" src="https://github.com/user-attachments/assets/eae7d19a-f092-409b-96a1-91519994c6bc" />
</p>

#### 🔎 Insights:

#### ⚒ Query 5: What subcategory has the highest sales based on quantities sold by each territory?
```sql
WITH sales_quantity AS    
    (SELECT    
        territory.Name AS territory,
        subcat.Name AS subcategory,
        SUM(OrderQty) AS item_qty
    FROM `adventureworks2019.Sales.SalesOrderDetail` AS sales
    LEFT JOIN `adventureworks2019.Production.Product` AS product 
        ON sales.ProductID = product.ProductID
    LEFT JOIN `adventureworks2019.Production.ProductSubcategory` AS subcat
        ON CAST(product.ProductSubcategoryID AS int) = subcat.ProductSubcategoryID
    LEFT JOIN `adventureworks2019.Sales.SalesOrderHeader`AS header 
        ON sales.SalesOrderID = header.SalesOrderID
    LEFT JOIN `adventureworks2019.Sales.SalesTerritory` AS territory
        ON header.TerritoryID = territory.TerritoryID 
    WHERE FORMAT_DATE('%Y', sales.ModifiedDate) IN ('2012','2013')
    GROUP BY 1,2)

SELECT territory, subcategory
FROM
    (SELECT *,
        DENSE_RANK() OVER(PARTITION BY territory ORDER BY item_qty DESC) AS ranked
    FROM sales_quantity)
WHERE ranked = 1;
```
#### 👉🏻 Results:
<p align='center'>
  <img width="400" height="296" alt="image" src="https://github.com/user-attachments/assets/cefa05a9-a9d9-4855-9d7c-24a73a92c928" />
</p>

#### 🔎 Insights:

#### ⚒ Query 6: How much money is spent on 'Seasonal' discounts by subcategory each year?
```sql
SELECT 
    year,
    subcategory,
    ROUND(SUM(discount_cost), 2) AS total_discount_cost
FROM
    (SELECT 
        FORMAT_DATE('%Y', sales.ModifiedDate) AS year,
        subcat.Name AS subcategory,
        OrderQty*UnitPrice*UnitPriceDiscount AS discount_cost    
    FROM `adventureworks2019.Sales.SalesOrderDetail` AS sales
      LEFT JOIN `adventureworks2019.Production.Product` AS product 
          ON sales.ProductID = product.ProductID
      LEFT JOIN `adventureworks2019.Production.ProductSubcategory` AS subcat
          ON CAST(product.ProductSubcategoryID AS int) = subcat.ProductSubcategoryID
      LEFT JOIN `adventureworks2019.Sales.SpecialOffer` AS offer
          ON sales.SpecialOfferID = offer.SpecialOfferID
    WHERE LOWER(type) LIKE '%seasonal discount%'
    AND FORMAT_DATE('%Y', sales.ModifiedDate) IN ('2012','2013')
    )
GROUP BY year, subcategory;
```
#### 👉🏻 Results: 
<p align='center'>
  <img width="400" height="80" alt="image" src="https://github.com/user-attachments/assets/0955a484-f4d4-43bc-a28d-88acd0bd43da" />
</p>

#### 🔎 Insights:

#### ⚒ Query 7: Customer retention analysis: Customer count and average lifetime revenue per customer by cohort
```sql
WITH info AS
    (SELECT 
        CustomerID,
        EXTRACT(month FROM detail.ModifiedDate) AS month,
        LineTotal AS revenue,
        ROW_NUMBER() OVER(PARTITION BY CustomerID ORDER BY EXTRACT(month FROM detail.ModifiedDate)) month_num
    FROM `adventureworks2019.Sales.SalesOrderDetail` AS detail
    LEFT JOIN `adventureworks2019.Sales.SalesOrderHeader` As header
        ON detail.SalesOrderID = header.SalesOrderID
    WHERE EXTRACT(year FROM detail.ModifiedDate) = 2013
    )

, join_month_table AS
    (SELECT CustomerID,
            month AS join_month
    FROM info
    WHERE month_num = 1)

, monthly_purchase AS 
    (SELECT 
        CustomerID,
        month AS purchase_month,
        SUM(revenue) AS monthly_spending
    FROM info
    GROUP BY 1,2)

SELECT
    join_month,
    purchase_month - join_month AS month_diff,
    COUNT(DISTINCT monthly_purchase.CustomerID) AS customer_count,
    ROUND(SUM(monthly_purchase.monthly_spending)/COUNT(DISTINCT monthly_purchase.CustomerID),2) AS avg_spending
FROM monthly_purchase
LEFT JOIN join_month_table ON monthly_purchase.CustomerID = join_month_table.CustomerID
GROUP BY 1,2
ORDER BY join_month, month_diff
```
#### 👉🏻 Results: 
<p align='center'>
  <img width="555" height="295" alt="image" src="https://github.com/user-attachments/assets/2e91b6d5-97d5-4806-9303-b947145affe2" />
</p>

#### 🔎 Insights:
<p align='center'>
  <img width="1296" height="350" alt="image" src="https://github.com/user-attachments/assets/57aa05d9-c01e-4fb9-9de9-721c05ae3e81" />
  <br>
  <img width="1293" height="352" alt="image" src="https://github.com/user-attachments/assets/66394abd-e423-4e53-9c67-e49591323aa1" />

</p>

