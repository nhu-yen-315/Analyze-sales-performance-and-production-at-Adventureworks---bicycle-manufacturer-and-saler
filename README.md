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
      <img width="835" height="350" alt="image" src="https://github.com/user-attachments/assets/fcb5ec95-a8e3-4b44-90f0-902ce39497f7" />
</p>

#### 🔎 Insights:
- **Bikes** and **bicycle frames** are two **important product categories**, contributing most to total revenue.
- **Road Bikes** ranked **#1** in **2011, 2012 and 2013** before **Mountain Bikes** **overtook** it in **2014**.
- No presence in 2011-2012, but **Touring Bikes** have become **increasingly important** since 2013. It ranked #3 in 2013 with §8.4M, then reaching #2 in 2014. That indicates a **growing demand**. 
- **Mountain Frames** and **Road Frames** also **dominated** sales in **2011 and 2012**. However, in the next two years, they are **out of top three** highest sales subcategories. This **needs special attention** to understand the reason behind. Whether **demand** for these products **declines** or whether **sales team** **didn't pay enough attention** on promoting these products.

