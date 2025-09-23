## PROBLEM: What is the total number of subscriptions since inception?
**Insight:** This query combines the current table and the archived table and gets all total subscriptions. This gives the business a complete overview of all sign ups since inception. 

## SQL QUERY 
'''sql
WITH subscription_counts AS (

    SELECT COUNT(*) AS total_subscriptions
    FROM subscriptions
    WHERE id > 0

    UNION ALL

    SELECT COUNT(*) AS total_subscriptions
    FROM subscriptions_21_01_2025
    WHERE id > 0
)

SELECT SUM(total_subscriptions) AS total_subscriptions
FROM subscription_counts;

## PROBLEM: How many subscriptions are created each month?
**Insight:** This query combines both the current and archived table to show monthly subscriptions. This enables the business to analyze the growth over the months, determine their best perfoming months and align with marketing activities. 

## SQL QUERY

'''sql
WITH monthly_counts AS (

    SELECT DATE_FORMAT(s.createdat, '%Y-%m') AS subscription_month,
           COUNT(*) AS total_subscriptions
    FROM subscriptions s
    WHERE s.id > 0
    GROUP BY DATE_FORMAT(s.createdat, '%Y-%m')

    UNION ALL

    SELECT DATE_FORMAT(s.createdat, '%Y-%m') AS subscription_month,
           COUNT(*) AS total_subscriptions
    FROM subscriptions_21_01_2025 s
    WHERE s.id > 0
    GROUP BY DATE_FORMAT(s.createdat, '%Y-%m')
)

SELECT 
    subscription_month,
    SUM(total_subscriptions) AS total_subscriptions
FROM monthly_counts
GROUP BY subscription_month
ORDER BY subscription_month ASC  
LIMIT 1000;


## PROBLEM:How many unique users exist?
**Insight:** This query returns all unique users in the system. This is useful to the business to determine their customer base and effective in marketing strategies. 

## SQLQUERY

'''sql 

SELECT COUNT(DISTINCT user_id) AS unique_users
FROM (
    SELECT user_id FROM subscriptions
    UNION ALL
    SELECT user_id FROM subscriptions_21_01_2025
) AS combined;

## PROBLEM: How many subscriptions were paid in the KSH currency? 
**Insight:** This query returns the total number of paid subscriptions in KSH currency.
             Since the system records all subscriptions paid and unpaid, the supdated at helps shows users who made payment and filtering it in KSH helps the business understand the local market.
             This query was also used to filter out other currencies. 

## QUERY

'''sql 

WITH daily_ksh_subscriptions AS (

    SELECT DATE(s.updatedat) AS subscription_day,
           JSON_UNQUOTE(JSON_EXTRACT(s.data, '$.curreny')) AS currency,
           COUNT(s.user_id) AS total_subscriptions
    FROM subscriptions s
    JOIN configurations c ON s.content_id = c.id
    WHERE s.updatedat IS NOT NULL
    GROUP BY DATE(s.updatedat), currency

    UNION ALL

    SELECT DATE(s.updatedat) AS subscription_day,
           JSON_UNQUOTE(JSON_EXTRACT(s.data, '$.curreny')) AS currency,
           COUNT(s.user_id) AS total_subscriptions
    FROM subscriptions_21_01_2025 s
    JOIN configurations c ON s.content_id = c.id
    WHERE s.updatedat IS NOT NULL
    GROUP BY DATE(s.updatedat), currency
)

SELECT SUM(total_subscriptions) AS total_ksh_subscriptions
FROM daily_ksh_subscriptions
WHERE currency = 'KSH';


## PROBLEM: How many transactoions were processed using GCP Payments?
**Insight:** While some payments are under mobile payment, international clients use GCP Payments. This query returns how many transactions were made using Google Pay. 
             This helps the business determine the adoption and reliability of Google Pay as a payment method. 

## SQL QUERY

'''sql

SELECT sum(total_transactions) AS `SUM(total_transactions)`
FROM
  (SELECT COUNT(*) AS total_transactions
   FROM subscriptions s
   WHERE JSON_EXTRACT(s.data, '$.purchaseID') IS NOT NULL) AS virtual_table
LIMIT 50000;

## PROBLEM: How many GCP transactions take place in a day?
**Insight:** This query shows the dail;y trends of GCP transactions. This helps in monitoring growth, sudden drops and seasonality in payment activities. 

## SQL QUERY 

'''sql 

SELECT DATE(transaction_date) AS transaction_date,
       SUM(daily_transactions) AS total_daily_transactions
FROM (
    SELECT DATE(s.createdat) AS transaction_date,
           COUNT(*) AS daily_transactions
    FROM subscriptions s
    WHERE JSON_EXTRACT(s.data, '$.purchaseID') IS NOT NULL
    GROUP BY DATE(s.createdat)
    ORDER BY transaction_date ASC
    LIMIT 1000
) AS virtual_table
GROUP BY DATE(transaction_date)
ORDER BY transaction_date ASC
LIMIT 1000;

## PROBLEM : How many GCP transactions take plasce in a month? 
**Insight:** The results show GCP payment trend over a month enabling the business to know the perfomance trend over the month, determine the highest perfoming month and align with the marketing team accordingly. 
             It alsop helps monitor the long term reliablity of the payment model overtime.

## SQL QUERY

''sql

SELECT transaction_month AS transaction_month,
       SUM(monthly_transactions) AS total_monthly_transactions
FROM (
    SELECT DATE_FORMAT(s.createdat, '%Y-%m') AS transaction_month,
           COUNT(*) AS monthly_transactions
    FROM subscriptions s
    WHERE JSON_EXTRACT(s.data, '$.purchaseID') IS NOT NULL
    GROUP BY DATE_FORMAT(s.createdat, '%Y-%m')
    ORDER BY transaction_month DESC
    LIMIT 1000
) AS virtual_table
GROUP BY transaction_month
ORDER BY total_monthly_transactions DESC
LIMIT 1000;



