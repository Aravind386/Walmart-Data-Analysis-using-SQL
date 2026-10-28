
# 🛒 Walmart Sales Data Analysis Using MySQL

## Overview

This project focuses on analyzing Walmart sales data using MySQL to uncover business insights related to customer payment preferences, product category ratings, branch performance, sales trends, and profitability.

The analysis uses SQL queries to explore transaction data, evaluate key business metrics, identify customer purchasing patterns, and compare branch revenue across different years.

This project demonstrates how SQL can be used to solve real-world business problems and support data-driven decision-making.

## Objectives

- Explore and understand Walmart sales transaction data.
- Analyze payment methods, transaction counts, and quantities sold.
- Identify the highest-rated product categories in each branch.
- Determine the busiest days for individual branches.
- Analyze product category ratings across cities.
- Calculate total profit by product category.
- Identify the most common payment method for each branch.
- Analyze transaction volume across morning, afternoon, and evening shifts.
- Identify branches experiencing a decline in revenue between 2022 and 2023.

## Dataset

The project uses a Walmart sales dataset containing transaction-level information.

- **Dataset Table:** `walmart`
- **Database:** MySQL
- **Dataset Source:** The original dataset source was not specified in the SQL queries. Add the dataset link here if you obtained it from Kaggle or another source.

### Dataset Columns Used

| Column | Description |
|---|---|
| `branch` | Branch identifier |
| `city` | City where the branch is located |
| `date` | Transaction date |
| `time` | Transaction time |
| `payment_method` | Payment method used |
| `quantity` | Quantity of items sold |
| `category` | Product category |
| `rating` | Customer rating |
| `unit_price` | Unit price of the product |
| `profit_margin` | Profit margin associated with the transaction |
| `total` | Total transaction value |

*Note: Column descriptions are based on how the fields are used in the supplied SQL queries.*

## Business Problems and Solutions

### 1. Explore the Dataset

**Objective:** Retrieve the sales records and understand the available data.

```sql
SELECT *
FROM walmart;
```

### 2. Count Total Records

**Objective:** Determine the total number of records in the dataset.

```sql
SELECT COUNT(*) AS total_records
FROM walmart;
```

### 3. Analyze Payment Methods

**Objective:** Identify the number of transactions for each payment method.

```sql
SELECT
    payment_method,
    COUNT(*) AS no_payments
FROM walmart
GROUP BY payment_method
ORDER BY no_payments DESC;
```

### 4. Count Distinct Branches

**Objective:** Determine how many unique branches are represented in the dataset.

```sql
SELECT
    COUNT(DISTINCT branch) AS total_branches
FROM walmart;
```

### 5. Find the Minimum Quantity Sold

**Objective:** Identify the smallest quantity recorded in a transaction.

```sql
SELECT
    MIN(quantity) AS minimum_quantity
FROM walmart;
```

### 6. Analyze Transactions and Quantity Sold by Payment Method

**Objective:** Compare the number of transactions and total items sold for each payment method.

```sql
SELECT
    payment_method,
    COUNT(*) AS no_payments,
    SUM(quantity) AS no_qty_sold
FROM walmart
GROUP BY payment_method
ORDER BY no_payments DESC;
```

### 7. Identify the Highest-Rated Category in Each Branch

**Objective:** Find the product category with the highest average customer rating in each branch.

```sql
SELECT
    branch,
    category,
    avg_rating
FROM (
    SELECT
        branch,
        category,
        AVG(rating) AS avg_rating,
        RANK() OVER (
            PARTITION BY branch
            ORDER BY AVG(rating) DESC
        ) AS category_rank
    FROM walmart
    GROUP BY branch, category
) AS ranked
WHERE category_rank = 1;
```

### 8. Identify the Busiest Day for Each Branch

**Objective:** Determine the day of the week with the highest number of transactions for each branch.

```sql
SELECT
    branch,
    day_name,
    no_transactions
FROM (
    SELECT
        branch,
        DAYNAME(STR_TO_DATE(date, '%d/%m/%Y')) AS day_name,
        COUNT(*) AS no_transactions,
        RANK() OVER (
            PARTITION BY branch
            ORDER BY COUNT(*) DESC
        ) AS day_rank
    FROM walmart
    GROUP BY
        branch,
        DAYNAME(STR_TO_DATE(date, '%d/%m/%Y'))
) AS ranked
WHERE day_rank = 1;
```

### 9. Calculate Total Quantity Sold by Payment Method

**Objective:** Understand which payment methods are associated with the highest sales quantities.

```sql
SELECT
    payment_method,
    SUM(quantity) AS no_qty_sold
FROM walmart
GROUP BY payment_method
ORDER BY no_qty_sold DESC;
```

### 10. Analyze Category Ratings by City

**Objective:** Compare minimum, maximum, and average customer ratings for each product category across cities.

```sql
SELECT
    city,
    category,
    MIN(rating) AS min_rating,
    MAX(rating) AS max_rating,
    AVG(rating) AS avg_rating
FROM walmart
GROUP BY city, category
ORDER BY city, avg_rating DESC;
```

### 11. Calculate Total Profit by Product Category

**Objective:** Identify the product categories contributing the highest calculated profit.

```sql
SELECT
    category,
    SUM(unit_price * quantity * profit_margin) AS total_profit
FROM walmart
GROUP BY category
ORDER BY total_profit DESC;
```

### 12. Identify the Most Common Payment Method for Each Branch

**Objective:** Determine the most frequently used payment method for each branch.

```sql
WITH payment_counts AS (
    SELECT
        branch,
        payment_method,
        COUNT(*) AS total_transactions,
        RANK() OVER (
            PARTITION BY branch
            ORDER BY COUNT(*) DESC
        ) AS payment_rank
    FROM walmart
    GROUP BY branch, payment_method
)
SELECT
    branch,
    payment_method AS preferred_payment_method,
    total_transactions
FROM payment_counts
WHERE payment_rank = 1;
```

### 13. Categorize Sales by Time of Day

**Objective:** Analyze transaction volumes across morning, afternoon, and evening shifts.

```sql
SELECT
    branch,
    CASE
        WHEN HOUR(TIME(time)) < 12 THEN 'Morning'
        WHEN HOUR(TIME(time)) BETWEEN 12 AND 17 THEN 'Afternoon'
        ELSE 'Evening'
    END AS shift,
    COUNT(*) AS num_invoices
FROM walmart
GROUP BY branch, shift
ORDER BY branch, num_invoices DESC;
```

### 14. Identify Branches with the Highest Revenue Decrease

**Objective:** Compare branch revenue in 2022 and 2023 and identify the five branches with the largest percentage decrease.

```sql
WITH revenue_2022 AS (
    SELECT
        branch,
        SUM(total) AS revenue
    FROM walmart
    WHERE YEAR(STR_TO_DATE(date, '%d/%m/%Y')) = 2022
    GROUP BY branch
),
revenue_2023 AS (
    SELECT
        branch,
        SUM(total) AS revenue
    FROM walmart
    WHERE YEAR(STR_TO_DATE(date, '%d/%m/%Y')) = 2023
    GROUP BY branch
)
SELECT
    r2022.branch,
    r2022.revenue AS last_year_revenue,
    r2023.revenue AS current_year_revenue,
    ROUND(
        (
            (r2022.revenue - r2023.revenue)
            / NULLIF(r2022.revenue, 0)
        ) * 100,
        2
    ) AS revenue_decrease_ratio
FROM revenue_2022 AS r2022
JOIN revenue_2023 AS r2023
    ON r2022.branch = r2023.branch
WHERE r2022.revenue > r2023.revenue
ORDER BY revenue_decrease_ratio DESC
LIMIT 5;
```

## Key Analysis Areas

- **Transaction Analysis:** Total records and transaction counts by payment method.
- **Payment Behavior:** Payment preferences and quantity sold by payment method.
- **Branch Performance:** Highest-rated categories and busiest days by branch.
- **Customer Satisfaction:** Category ratings across cities.
- **Profitability Analysis:** Calculated profit by product category.
- **Time-Based Analysis:** Transaction distribution across different shifts.
- **Revenue Trend Analysis:** Year-over-year branch revenue comparisons.


## Author

**Aravind K**  
Data Analyst | MIS Reporting | SQL | Power BI | Python  
Bengaluru, India


Thank you for checking out my project! Feedback and suggestions are welcome.
