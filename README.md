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

**Adventure Works Cycles** is a **bicycle manufacturing** company. In addition to **finished bicycles**, the company also sells **individual bicycle components**, **sport clothes and equipments**. This project analyzes **sales performance** over the period from **2012 to 2013** using **SQL**.

The analysis aims to address the following **key business questions**:

- How did different **product subcategories** perform in terms of sales?

- What was the sales performance across various **sales territories**?

- What insights can be drawn about **customer retention**?

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
- **Clothing subcategories** have experienced the **rapid growth** in quantities sold. **Socks** have the **most impressive** growth rate at over **420%** while **socks and jerseys** rank **second and third** at over **263% and 183%** respectively. That indicates that **clothings** are **potential products** and require **further investments** in the future.
- In contrast, the sales of **bicyle components** such as **locks, forks and wheels** have **dropped significantly**.
- Notably, **wheels** were a **key subcategory** in **2012**, with **3,802 units** sold. However, sales **dropped sharply** to just **1,471 units** in 2013, highlighting a **substantial decrease in demand**.

#### ⚒ Query 3: Which territories have contributed most to total revenue?
```sql
WITH sales_quantity AS     
    (SELECT 
            FORMAT_DATE('%Y', sales.ModifiedDate) AS year,
            territory.Name AS territory,
            ROUND(SUM(LineTotal), 2) AS revenue
    FROM `adventureworks2019.Sales.SalesOrderDetail` AS sales
    LEFT JOIN `adventureworks2019.Sales.SalesOrderHeader`AS header 
        ON sales.SalesOrderID = header.SalesOrderID
    LEFT JOIN `adventureworks2019.Sales.SalesTerritory` AS territory
        ON header.TerritoryID = territory.TerritoryID 
    WHERE FORMAT_DATE('%Y', sales.ModifiedDate) IN ('2012','2013')
    GROUP BY 1,2)

SELECT * FROM
    (SELECT *,
            DENSE_RANK() OVER(PARTITION BY year ORDER BY revenue DESC) AS ranked
    FROM sales_quantity)
WHERE ranked IN (1,2,3);
```
#### 👉🏻 Results:
<p align='center'>
  <img width="709" height="187" alt="image" src="https://github.com/user-attachments/assets/342702a4-cacc-435e-bb53-a46125339b66" />
</p>

#### 🔎 Insights:
- **Southwest, Canada and Northwest** have **consistently contributed most** to total revenue during 2012 and 2013. All three territories have experienced **increase in revenues**.
- **Southwest** is the **most important** territory, contributing **$8.2M and $9.1M** in 2012 and 2013 respectively.
- **Canada** contributed **$5.8M** revenue in 2012 and then dropped to **$6.2M** in the following year.
- Similarly, **Northwest** generated **$4.7M and $6M** in two years.
  
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

- **All territories** experience **positive year-over-year growth** in terms of quantities sold with the lowest rate of 21.2%.
- Starting as a **young market** in 2012, **Germany** has the **most rapid** YoY growth rate at **3169%**, indicating that the **expansion strategy** to this market is **effective**.
- **Australia, United Kingdom and France** which are relatively young markets also see surge in sales at **960%, 243% and 220%**.
- Other **mature markets** have more **modest** growth rates ranging from **21% to 100%**.
  
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

- **Finished bikes** are the **most important subcategories** in **most markets**, except for **Australia**.
- **Road bikes** are **best-sellers** in **7 territories**.
- The young market with the strongest growth **Germany** favors **touring bikes** most, while **Australia**, the second fastest growth market, favors **tires and tubes**.
- **Mountain bikes** are best-selling in **Northwest** which is one of the most important territories in terms of revenue.

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
- **Seasonal discount** only applied to **Helmets** subcategory.
- The discount cost doubled from **$827** to **$1606** during the period.
  
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
</p>

- The **second half** of 2013 attracted approximately **three times more customers** than the first half. A likely explanation is the presence of **major holidays**—such as **Halloween, Christmas, and New Year**— which typically drive **higher consumer activity**.
- Interestingly, **purchase behavior** **differs** between the **January–June** and **July–December customer cohorts**. **Customers** acquired in the **first half** of the year tend to **repurchase after two months**, whereas those from the **second half** show a **higher purchase frequency**, often **repurchasing monthly**.

<P align='center'>
  <img width="1293" height="352" alt="image" src="https://github.com/user-attachments/assets/66394abd-e423-4e53-9c67-e49591323aa1" />
</p>

- Even though the number of new customers in the **first six months** is relatively low, the **average spendings** are **high**.
- **New customers** acquired in **January, February and March** have the **highest spendings** ranging from **$5000** to **$8000**.

## 4. 💡 Recommendations

- **Market expansion**
  + The company should continue to **expand operations** in **young and fast-growing markets** like **Germany (3169% yoy growth rate) , Australia (960%), United Kingdom (243%) and France (220%)**.
  + **Touring bikes** are the **key products** to **Germany**. The company should prepare the **manufacturing capacity** for touring bikes to meet huge demand in German market. 
  + **Tires and tubes** are **best-selling** products in **Australia** while both **United Kingdom and France** prefer **road bikes** most.
    
- **Product diversification**
  + **Clothing** subcategories like **socks, shorts and jerseys** had **low sales in 2012**; however, they experienced **surge in sales** in 2013, indicating clothings are the **new potential product lines** and will contribute to future growth in revenues. The company should **invest more on clothings** for sport activities.
    
- **Customer retention strategies**
  + Since **customer acquisition triples** in the  **second half** of 2013, the company can launch **seasonal marketing campaigns earlier** in the year (e.g. pre-summer promotions) to **spread out customer acquisition** across quarters.
  + **Last months** of the year acquires **high number of new customers** but **average spending** is relatively **low**, the company can improve spending by running **time-limited offers around holiday periods**.
  + It is found that **Jan-June** customers and **July-Dec** customers have **different repurchase pattern**. So the company should tailor follow-up communication by cohort:
    + For **Jan–June** customers, schedule **reminders or offers** around the **2-month** mark.
    + For **July–Dec** customers, consider **monthly loyalty perks** or **product recommendations** to **maintain their faster repurchase pace**.
  + **Customers** acquired in the **first six months** have **higher average spendings**. **Demographic analysis, products purchased and acquisition channels** for this group can be conducted to **identify patterns**. These information will be used to **target similar potential customers**. 
