# Pesaflix Revenue SQL Analysis

This document contains SQL queries and insights designed to analyze and track revenue performance across the PesaFlix platform.
It consolidates data from both active and archived subscription tables to provide a complete financial picture from total earnings to detailed transaction-level insights.

📑 Table of Contents

1. [Total Revenue per Currency](#total-revenue-per-currency)
2. [Total Revenue by Date and Currency](#total-revenue-by-date-and-currency)
3. [Monthly Revenue by Currency](#monthly-revenue-by-currency)
4. [Detailed Transactions Showing User and Payment Data](#detailed-transactions-showing-user-and-payment-data)
5. [GCP Payments](#gcp-payments)

## Problem 1: Total Revenue per Currency

**Insight:** This query provides a comprehensive view of total earnings across all currencies, combining data from both active and archived subscription tables.
It reveals the overall financial performance of PesaFlix, highlighting which currencies contribute the most to total revenue.

## SQL Query: 

```sql

SELECT 
    currency,
    SUM(total_revenue) AS total_revenue
FROM (
    SELECT 
        JSON_UNQUOTE(JSON_EXTRACT(s.data, '$.curreny')) AS currency,
        SUM(CAST(JSON_UNQUOTE(JSON_EXTRACT(s.data, '$.amount')) AS DECIMAL)) AS total_revenue
    FROM subscriptions s
    JOIN configurations c ON s.content_id = c.id
    WHERE s.id > 0
      AND s.updatedAt IS NOT NULL
    GROUP BY currency

    UNION ALL

    SELECT 
        JSON_UNQUOTE(JSON_EXTRACT(s.data, '$.curreny')) AS currency,
        SUM(CAST(JSON_UNQUOTE(JSON_EXTRACT(s.data, '$.amount')) AS DECIMAL)) AS total_revenue
    FROM subscriptions_21_01_2025 s
    JOIN configurations c ON s.content_id = c.id
    WHERE s.id > 0
      AND s.updatedAt IS NOT NULL
    GROUP BY currency
) AS combined_revenue
GROUP BY currency
ORDER BY total_revenue DESC;

```

## Problem 2: Total Revenue by Date and Currency

**Insight:** This query calculates the total daily revenue generated in each currency, combining data from both the primary subscriptions table and the archived subscriptions_21_01_2025 table. 
It provides a unified view of platform earnings over time, useful for business performance tracking and financial reporting.

## SQL Query 

```sql

SELECT currency AS currency,
       SUM(total_revenue) AS `SUM(total_revenue)`
FROM
  (SELECT JSON_UNQUOTE(JSON_EXTRACT(s.data, '$.curreny')) AS currency,
          SUM(CAST(JSON_UNQUOTE(JSON_EXTRACT(s.data, '$.amount')) AS DECIMAL)) AS total_revenue
   FROM subscriptions s
   JOIN configurations c ON s.content_id = c.id
   WHERE s.id > 0
     AND s.updatedAt IS NOT NULL
   GROUP BY currency
   
   UNION ALL
   
   SELECT JSON_UNQUOTE(JSON_EXTRACT(s.data, '$.curreny')) AS currency,
          SUM(CAST(JSON_UNQUOTE(JSON_EXTRACT(s.data, '$.amount')) AS DECIMAL)) AS total_revenue
   FROM subscriptions_21_01_2025 s
   JOIN configurations c ON s.content_id = c.id
   WHERE s.id > 0
     AND s.updatedAt IS NOT NULL
   GROUP BY currency) AS virtual_table
GROUP BY currency
ORDER BY revenue_date DESC; 

```

## Problem 3: Monthly Revenue by Currency

**Insight:** This query provides a month-by-month breakdown of revenue per currency, combining both current and historical subscription data.
It highlights trends in revenue growth, identifies peak months, and shows which currencies contribute most consistently over time.
This analysis is valuable for forecasting, strategic planning, and evaluating the effectiveness of pricing or promotional campaigns across different markets.


## SQL Query

```sql

SELECT currency,
          revenue_month,
          SUM(total_revenue) AS total_revenue
   FROM
     (SELECT JSON_UNQUOTE(JSON_EXTRACT(s.data, '$.curreny')) AS currency,
             DATE_FORMAT(s.createdat, '%Y-%m') AS revenue_month,
             SUM(CAST(JSON_UNQUOTE(JSON_EXTRACT(s.data, '$.amount')) AS DECIMAL)) AS total_revenue
      FROM subscriptions s
      JOIN configurations c ON s.content_id = c.id
      WHERE s.id > 0
        AND s.updatedAt IS NOT NULL
      GROUP BY currency,
               revenue_month
      UNION ALL SELECT JSON_UNQUOTE(JSON_EXTRACT(s.data, '$.curreny')) AS currency,
                       DATE_FORMAT(s.createdat, '%Y-%m') AS revenue_month,
                       SUM(CAST(JSON_UNQUOTE(JSON_EXTRACT(s.data, '$.amount')) AS DECIMAL)) AS total_revenue
      FROM subscriptions_21_01_2025 s
      JOIN configurations c ON s.content_id = c.id
      WHERE s.id > 0
        AND s.updatedAt IS NOT NULL
      GROUP BY currency,
               revenue_month) AS combined_data
   GROUP BY currency,
            revenue_month
   ORDER BY revenue_month,
            currency) AS virtual_table
GROUP BY revenue_month,
         currency
ORDER BY revenue_month DESC;

```

## Problem 4: Detailed Transactions Showing User and Payment Data

**Insight:** This query gives a granular, transaction-level view of revenue, enabling PesaFlix to
- Audit and verify payments, 
- Trace individual user activity and payments over time
- Identify anomalies or unusual patterns
- Build detailed dashboards or reports for operational and financial monitoring
  
## SQL Query

```sql

SELECT currency AS currency,
          revenue_date AS revenue_date,
          user_id AS user_id,
          amount AS amount,
          payment_data AS payment_data
   FROM
     (SELECT JSON_UNQUOTE(JSON_EXTRACT(s.data, '$.curreny')) AS currency,
             DATE(s.createdat) AS revenue_date,
             s.user_id AS user_id,
             CAST(JSON_UNQUOTE(JSON_EXTRACT(s.data, '$.amount')) AS DECIMAL) AS amount,
             s.data AS payment_data
      FROM subscriptions s
      JOIN configurations c ON s.content_id = c.id
      WHERE s.id > 0
        AND s.updatedAt IS NOT NULL
      UNION ALL SELECT JSON_UNQUOTE(JSON_EXTRACT(s.data, '$.curreny')) AS currency,
                       DATE(s.createdat) AS revenue_date,
                       s.user_id AS user_id,
                       CAST(JSON_UNQUOTE(JSON_EXTRACT(s.data, '$.amount')) AS DECIMAL) AS amount,
                       s.data AS payment_data
      FROM subscriptions_21_01_2025 s
      JOIN configurations c ON s.content_id = c.id
      WHERE s.id > 0
        AND s.updatedAt IS NOT NULL ) AS virtual_table
   ORDER BY revenue_date DESC; 
        
```

## Problem 5: GCP Payments

**Insight:** This query provides a focused view of GCP-related subscription activity, allowing PesaFlix to:
- Monitor recent GCP payment transactions
- Audit purchase IDs and subscription periods
- Track subscription lifecycle (start and end dates)
- Quickly access raw JSON data for further validation or debugging
It’s essentially a transaction log for GCP payments, giving a clear and concise view of recent activity.

## SQL Query

```sql
SELECT 
    s.createdat AS revenue_date,
    s.user_id AS user_id,
    s.data AS payment_data,
    s.startat AS start_at,
    s.endat AS end_at
FROM subscriptions s
WHERE JSON_EXTRACT(s.data, '$.purchaseID') IS NOT NULL
ORDER BY s.createdat DESC;

```
