# 📈 PL/SQL Window Functions Analysis: E-commerce Performance & Segmentation

# Umutoniwase Aliane
# ID: 27883

## 🎯 Project Objective

This project demonstrates mastery of **PL/SQL Window Functions** by solving a realistic business challenge for a large e-commerce retailer. The goal is to move beyond simple aggregate reports and implement advanced analytical queries to provide **dynamic, localized, and time-series insights** necessary for strategic marketing and inventory decisions.

---

## 💼 Business Problem & Expected Outcome

| Section | Description |
| :--- | :--- |
| **Business Context** | A Large E-commerce Retailer's Sales & Marketing Department, operating in the Retail Industry. |
| **Data Challenge** | The retailer struggles to **efficiently segment its customer base** and **identify localized, high-performing products** due to static reporting that lacks comparative and trend analysis over time and across different geographic regions. |
| **Expected Outcome** | The analysis provides **actionable insights** to **optimize inventory allocation** (via localized product rankings), **refine marketing budget allocation** (via customer segmentation), and **improve forecasting** (via rolling averages and growth calculations). |

---

## 🛠️ Technology Stack & Requirements

* **Database:** Oracle Database (or any database supporting standard SQL window functions like PostgreSQL, MySQL 8.0+)
* **Language:** SQL (Focus on Analytical Window Functions)
* **Documentation:** GitHub, Professional README
* **Required Scripts:** `create_tables.sql`, `insert_data.sql`, `analysis_queries.sql` (These files must be included in the repository root.)

# create tables

**customer**
![Example Image](Table%20Creation%20customers.PNG)

![Example Image](customers%20data.PNG)

**product**

![Example Image](Table%20creatio%20products.PNG)

![Example Image](product%20Data.PNG)

**transaction**

![Example Image](Table%20creation%20transactions.PNG)

![Example Image](Transaction%20data.PNG)

---

## 📊 Database Schema (ER Diagram)

The data model consists of three interconnected tables designed to capture customer location, product details, and sales history

**DATABASE RELATIONSHIP**

![Example Image](Transaction%20model.PNG)

---

## 💻 SQL Scripts & Analytical Queries

The core of this project is the implementation of exactly five analytical queries using various SQL Window Functions to meet the defined success criteria.

### 1. Top 5 Products per Region/Quarter (Ranking)

**Function Used:** `RANK()`

```sql
SELECT
    product_rank, -- Changed from "rank"
    region,
    sale_quarter,
    product_name,
    quarterly_revenue
FROM
    (
    SELECT
        c.region,
        p.name AS product_name,
        TRUNC(t.sale_date, 'Q') AS sale_quarter,
        SUM(t.amount) AS quarterly_revenue,
        -- Changed alias to product_rank
        RANK() OVER (
            PARTITION BY c.region, TRUNC(t.sale_date, 'Q')
            ORDER BY SUM(t.amount) DESC
        ) AS product_rank
    FROM
        transactions t
    JOIN
        customers c ON t.customer_id = c.customer_id
    JOIN
        products p ON t.product_id = p.product_id
    GROUP BY
        c.region,
        p.name,
        TRUNC(t.sale_date, 'Q')
    )
WHERE
    product_rank <= 5 -- Referenced new alias without quotes
ORDER BY
    region,
    sale_quarter,
    product_rank;
```

### 2. Running monthly sales totals 

**Function Used:** `SUM() OVER()`

```sql
SELECT
    TO_CHAR(sale_month, 'YYYY-MM') AS month_year,
    monthly_total_sales,
    -- Calculate running total by ordering only on the sale_month
    SUM(monthly_total_sales) OVER (
        ORDER BY sale_month
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total_sales
FROM
    (
    -- Subquery to aggregate sales to the monthly level first
    SELECT
        TRUNC(sale_date, 'MM') AS sale_month,
        SUM(amount) AS monthly_total_sales
    FROM
        transactions
    GROUP BY
        TRUNC(sale_date, 'MM')
    )
ORDER BY
    sale_month;
```

### 3. Month-over-month growth

**Function Used:** `LAG()/LEAD()`

```sql
SELECT
    TO_CHAR(sale_month, 'YYYY-MM') AS month_year,
    monthly_sales,
    previous_month_sales,
    -- Calculate MoM Growth Percentage
    ROUND(
        ((monthly_sales - previous_month_sales) / previous_month_sales) * 100,
        2
    ) AS mom_growth_percentage
FROM
    (
    SELECT
        sale_month,
        monthly_sales,
        -- Use LAG() to fetch the sales amount from the previous month
        LAG(monthly_sales, 1) OVER (ORDER BY sale_month) AS previous_month_sales
    FROM
        (
        -- Subquery to aggregate sales to the monthly level
        SELECT
            TRUNC(sale_date, 'MM') AS sale_month,
            SUM(amount) AS monthly_sales
        FROM
            transactions
        GROUP BY
            TRUNC(sale_date, 'MM')
        )
    )
ORDER BY
    sale_month;
```

### 4. Customer quartiles

**Function Used:** `NTILE(4)`

```sql
SELECT
    customer_id,
    customer_name,
    total_lifetime_spending,
    -- NTILE(4) assigns a group number (1 to 4) based on the spending order
    NTILE(4) OVER (
        ORDER BY total_lifetime_spending DESC
    ) AS spending_quartile -- Quartile 1 is the highest value
FROM
    (
    -- Subquery to calculate each customer's lifetime spending
    SELECT
        c.customer_id,
        c.name AS customer_name,
        SUM(t.amount) AS total_lifetime_spending
    FROM
        customers c
    JOIN
        transactions t ON c.customer_id = t.customer_id
    GROUP BY
        c.customer_id,
        c.name
    )
ORDER BY
    spending_quartile,
    total_lifetime_spending DESC;
```


### 5. 3-month moving averages

**Function Used:** `AVG() OVER()`

```sql
SELECT
    TO_CHAR(sale_month, 'YYYY-MM') AS month_year,
    monthly_sales,
    -- Calculate the average of the current month and the two preceding months
    AVG(monthly_sales) OVER (
        ORDER BY sale_month
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS three_month_moving_average
FROM
    (
    -- Subquery to aggregate sales to the monthly level
    SELECT
        TRUNC(sale_date, 'MM') AS sale_month,
        SUM(amount) AS monthly_sales
    FROM
        transactions
    GROUP BY
        TRUNC(sale_date, 'MM')
    )
ORDER BY
    sale_month;
```


