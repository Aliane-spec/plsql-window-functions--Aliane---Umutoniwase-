# 📈 PL/SQL Window Functions Analysis: E-commerce Performance & Segmentation

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



---

## 📊 Database Schema (ER Diagram)

The data model consists of three interconnected tables designed to capture customer location, product details, and sales history.

**Schema Details:**

| Table | Purpose | Key Columns | Relationship |
| :--- | :--- | :--- | :--- |
| `customers` | Customer info and geographical region. | `customer_id` (PK), `region` | 1-to-Many with `transactions` |
| `products` | Product catalog and category. | `product_id` (PK), `name` | 1-to-Many with `transactions` |
| `transactions` | Detailed sales records. | `transaction_id` (PK), `sale_date`, `amount` | Bridge table connecting `customers` and `products` |



---

## 💻 SQL Scripts & Analytical Queries

The core of this project is the implementation of exactly five analytical queries using various SQL Window Functions to meet the defined success criteria.

### 1. Top 5 Products per Region/Quarter (Ranking)

**Function Used:** `RANK()`

```sql
SELECT *
FROM (
    SELECT
        c.region,
        p.name AS product_name,
        TRUNC(t.sale_date, 'Q') AS sale_quarter,
        -- ...
        RANK() OVER (
            PARTITION BY c.region, TRUNC(t.sale_date, 'Q')
            ORDER BY SUM(t.amount) DESC
        ) AS product_rank
    FROM transactions t 
    -- ... (JOINs and GROUP BY)
)
WHERE product_rank <= 5;
