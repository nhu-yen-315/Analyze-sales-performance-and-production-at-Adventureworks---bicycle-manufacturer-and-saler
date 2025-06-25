# 📊 Project title: Bicycle sales, production and purchasing analysis with SQL

Author: Huỳnh Như Yến  
Date: 25/6/2025 <br>
Tool Used: Power BI

---
## 📑 Table of Contents  
1. 📌 Background & Overview 
2. 📂 Dataset Description
3. 🔎 Questions and Queries

---
## 1. 📌 Background & Overview
### 📖 What is this project about? 
This project aims to answer questions related to the sales, production and purchasing activities of a bicycle manufacturer. It will answer questions such as:
- Sales: What are revenue and quantity sold by subcategories?
- Production: What are trends in stock levels?
- Purchasing: What is the number of order and value at pending status?

### 👤 Who is this project for?  
✔️ Sales managers <br>
✔️ Data analysts & business analysts <br>
✔️ Recruiters

---
## 2. 📂 Dataset Description 
- The dataset is named AdventureWorks retrieved from BigQuery. This is the link to data dictionary: https://drive.google.com/file/d/1bwwsS3cRJYOg1cvNppc1K_8dQLELN16T/view. 
- Dataset includes data for multiple departments like production, sales, purchasing in a bicycle manufacturer. To answer business questions, I select and join multiple tables from these three functions.
- From Sales department, Sales.SalesOrderDetail, Sales.SalesOrderHeader, Sales.Customer tables are selected.
- From Production department, Production.WorkOrder, Production.Product, Production.ProductSubcategory tables are selected.
- From Purchasing department, Purchasing.PurchaseOrderHeader table is selected.

---
## 3. 🔎 Questions and Queries
### Sales questions
#### Q1. Calculate Quantity of items, Sales value & Order quantity by each Subcategory in L12M
```sql
SELECT 
      FORMAT_DATETIME('%b %Y', a.ModifiedDate) AS period,
      c.Name,
      SUM(OrderQty) AS qty_item,
      SUM(LineTotal) AS total_sales,
      COUNT(DISTINCT SalesOrderID) AS order_cnt
FROM `adventureworks2019.Sales.SalesOrderDetail` AS a
LEFT JOIN `adventureworks2019.Production.Product` AS b ON a.ProductID = b.ProductID
LEFT JOIN `adventureworks2019.Production.ProductSubcategory` AS c ON CAST(b.ProductSubcategoryID AS int) = c.ProductSubcategoryID
WHERE FORMAT_DATETIME('%b %Y', a.ModifiedDate) = 'Sep 2013'
GROUP BY 1,2
ORDER BY Name;
```

Output <br>
<img width="844" alt="image" src="https://github.com/user-attachments/assets/88dd7f00-6e28-48f6-95ec-aabb83131106" />


#### Q2. Calculate % YoY growth rate by SubCategory & release top 3 cat with highest grow rate. Can use metric: quantity_item. Round results to 2 decimal qty_diff = qty_item / prv_qty - 1
```sql
WITH present_quantity AS   
  (SELECT 
        FORMAT_DATETIME('%Y', a.ModifiedDate) AS period,
        c.Name,
        SUM(OrderQty) AS present_qty
  FROM `adventureworks2019.Sales.SalesOrderDetail` AS a
  LEFT JOIN `adventureworks2019.Production.Product` AS b ON a.ProductID = b.ProductID
  LEFT JOIN `adventureworks2019.Production.ProductSubcategory` AS c ON CAST(b.ProductSubcategoryID AS int) = c.ProductSubcategoryID
  GROUP BY 1,2)

SELECT 
      Name,
      present_qty,
      LAG(present_qty) OVER(PARTITION BY Name ORDER BY period) AS previous_qty,
      ROUND((present_qty - LAG(present_qty) OVER(PARTITION BY Name ORDER BY period))*1.0/(LAG(present_qty) OVER(PARTITION BY Name ORDER BY period)), 2) AS qty_diff
FROM present_quantity
ORDER BY qty_diff DESC
LIMIT 3;
```

Output <br>
<img width="649" alt="image" src="https://github.com/user-attachments/assets/aa5266cb-2481-4baf-9d2a-9a1bedd58c59" />


#### Q3. Ranking Top 3 TeritoryID with biggest Order quantity of every year. If there's TerritoryID with same quantity in a year, do not skip the rank number
```sql
WITH count_quantity AS  -- compute the order quantity by year and territory
  (SELECT 
        FORMAT_DATETIME('%Y', a.ModifiedDate) AS yr,
        c.TerritoryID,
        SUM(OrderQty) AS order_cnt
  FROM `adventureworks2019.Sales.SalesOrderDetail` AS a
  LEFT JOIN `adventureworks2019.Sales.SalesOrderHeader` AS b ON a.SalesOrderID = b.SalesOrderID
  LEFT JOIN `adventureworks2019.Sales.Customer` AS c ON b.CustomerID = c.CustomerID
  GROUP BY 1,2)

, ranking AS -- rank the territory 
  (SELECT *,
        DENSE_RANK() OVER(PARTITION BY yr ORDER BY order_cnt DESC) AS rn
  FROM count_quantity
  ORDER BY yr)

-- select territories in the top three of each year 
SELECT * 
FROM ranking
WHERE rn <= 3
ORDER BY yr DESC;
```

Output <br>
<img width="648" alt="image" src="https://github.com/user-attachments/assets/3d60e238-084e-4b0f-9d28-6572a9c61b46" />

 
#### Q4. Calculate Total Discount Cost belongs to Seasonal Discount for each SubCategory
```sql
SELECT  
      FORMAT_DATETIME('%Y', a.ModifiedDate) AS yr,
      c.Name,
      SUM(OrderQty*UnitPrice*UnitPriceDiscount) AS total_cost
FROM `adventureworks2019.Sales.SalesOrderDetail` AS a
LEFT JOIN `adventureworks2019.Production.Product` AS b ON a.ProductID = b.ProductID
LEFT JOIN `adventureworks2019.Production.ProductSubcategory` AS c ON CAST(b.ProductSubcategoryID AS int) = c.ProductSubcategoryID
WHERE a.SpecialOfferID IN ((SELECT SpecialOfferID
                            FROM `adventureworks2019.Sales.SpecialOffer`
                            WHERE type = 'Seasonal Discount'))
GROUP BY 1,2;
```

Output <br>
<img width="596" alt="image" src="https://github.com/user-attachments/assets/4e4654b4-b7af-4172-95bd-44a881c2e4cb" />

#### Q5. Retention rate of Customer in 2014 with status of Successfully Shipped (Cohort Analysis)
```sql
WITH raw_data AS  
  (SELECT  DISTINCT 
        CustomerID,
        EXTRACT(month FROM ModifiedDate) AS mth
  FROM `adventureworks2019.Sales.SalesOrderHeader` 
  WHERE Status = 5 AND EXTRACT(year FROM ModifiedDate) = 2014
  ORDER BY CustomerID, mth)

, join_month AS 
  (SELECT 
        CustomerID,
        MIN(mth) AS mth_join
  FROM raw_data
  GROUP BY CustomerID)

, cal_mth_diff AS 
  (SELECT CustomerID,
          mth_join,
          concat('M-', mth - mth_join) AS mth_diff
  FROM join_month
  INNER JOIN raw_data
  USING(CustomerID))

SELECT
      mth_join,
      mth_diff,
      COUNT(DISTINCT CustomerID) AS customer_cnt
FROM cal_mth_diff
GROUP BY mth_join, mth_diff
ORDER BY mth_join;
```
Output <br>
<img width="516" alt="image" src="https://github.com/user-attachments/assets/050055c6-cc29-46ee-a8fd-7dbbdf28d06b" />

### Production questions
#### Q6. Trend of Stock level & MoM diff % by all product in 2011. If %gr rate is null then 0. Round to 1 decimal
```sql
WITH raw_data AS  
  (SELECT 
        b.Name,
        EXTRACT(month FROM a.ModifiedDate) AS mth,
        EXTRACT(year FROM a.ModifiedDate) AS yr,
        StockedQty
  FROM `adventureworks2019.Production.WorkOrder` AS a
  LEFT JOIN `adventureworks2019.Production.Product` AS b
  USING (ProductID)
  WHERE EXTRACT(year FROM a.ModifiedDate) = 2011)

, cal_quantity AS 
  (SELECT
        Name,
        mth,
        yr,
        SUM(StockedQty) AS stock_qty
  FROM raw_data
  GROUP BY 1,2,3)

, cal_diff AS 
  (SELECT *,
        LAG(stock_qty) OVER(PARTITION BY Name ORDER BY mth) AS stock_prv,
        ROUND((stock_qty - LAG(stock_qty) OVER(PARTITION BY Name ORDER BY mth))*100.0/LAG(stock_qty) OVER(PARTITION BY Name ORDER BY mth),1) AS diff
  FROM cal_quantity)

SELECT
      *,
      CASE WHEN diff IS NULL THEN 0 END
FROM cal_diff
ORDER BY Name, mth DESC;
```

Output <br>
<img width="1012" alt="image" src="https://github.com/user-attachments/assets/290c562d-8932-4f94-bcd1-e59fcad27318" />

#### Q7. Calculate Ratio of Stock / Sales in 2011 by product name, by month
Order results by month desc, ratio desc. Round Ratio to 1 decimal

```sql
WITH cal_sales AS  -- create a CTE containing information about sales by product and month
  (SELECT  
        EXTRACT(month FROM a.ModifiedDate) AS mth,
        EXTRACT(year FROM a.ModifiedDate) AS yr,
        a.ProductID,
        Name, 
        SUM(OrderQty) AS sales
  FROM `adventureworks2019.Sales.SalesOrderDetail` AS a
  LEFT JOIN `adventureworks2019.Production.Product` AS b
  USING (ProductID)
  WHERE EXTRACT(year FROM a.ModifiedDate) = 2011 
  GROUP BY 1,2,3,4)

, cal_stock AS -- create a CTE containing information about stock by product and month
  (SELECT 
        ProductID,
        EXTRACT(month FROM ModifiedDate) AS mth,
        SUM(StockedQty) AS stock
  FROM `adventureworks2019.Production.WorkOrder`
  WHERE EXTRACT(year FROM ModifiedDate) = 2011
  GROUP BY 1,2)

-- join 2 tables and calculate the ratio
SELECT 
      cal_sales.mth,
      yr,
      cal_sales.ProductID,
      Name,
      sales,
      stock,
      ROUND(stock*1.0/sales, 2) AS ratio
FROM cal_sales
LEFT JOIN cal_stock
ON cal_sales.ProductID = cal_stock.ProductID
AND cal_sales.mth = cal_stock.mth
ORDER BY mth DESC, ratio DESC;
```

Output <br>
<img width="965" alt="image" src="https://github.com/user-attachments/assets/380a461d-dee9-4eb4-b743-dedfcde0193e" />

### Purchasing question
#### Q8. Number of order and value at Pending status in 2014
```sql
SELECT  
      '2014' AS yr,
      '1' AS status,
      COUNT(DISTINCT PurchaseOrderID) AS order_cnt,
      SUM(TotalDue) AS value
FROM `adventureworks2019.Purchasing.PurchaseOrderHeader` 
WHERE EXTRACT(year FROM ModifiedDate) = 2014
AND Status = 1;
```

Output <br>
<img width="722" alt="image" src="https://github.com/user-attachments/assets/fa1708d8-4045-46df-b091-d936a1c2f75b" />
