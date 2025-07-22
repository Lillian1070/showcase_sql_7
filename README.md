# Repeated Payments

[link](https://datalemur.com/questions/repeated-payments)



## Practice 

Sometimes, payment transactions are repeated by accident; it could be due to user error, API failure or a retry error that causes a credit card to be charged twice.

Using the transactions table, identify any payments made at the same merchant with the same credit card for the same amount within 10 minutes of each other. Count such repeated payments.

__Assumptions:__

- The first transaction of such payments should not be counted as a repeated payment. This means, if there are two transactions performed by a merchant with the same credit card and for the same amount within 10 minutes, there will only be 1 repeated payment.


__Table:__ `transactions`

transaction_id	merchant_id	credit_card_id	amount	transaction_timestamp
1	101	1	100	09/25/2022 12:00:00
2	101	1	100	09/25/2022 12:08:00
3	101	1	100	09/25/2022 12:28:00
4	102	2	300	09/25/2022 12:00:00
6	102	2	400	09/25/2022 14:00:00


### Expected_Output: 

payment_count
1



## My PostgreSQL Query

- Using `LAG()`
- Using `INTERVAL`



```sql
WITH prev_trans AS (
  SELECT
    transaction_id,
    merchant_id,
    credit_card_id,
    amount,
    transaction_timestamp,
    LAG(transaction_timestamp) OVER 
      (PARTITION BY merchant_id, credit_card_id, amount 
        ORDER BY transaction_timestamp) AS prev_transaction
  FROM transactions
),
prev_time_diff AS (
  SELECT 
    *,
    (transaction_timestamp	- prev_transaction) AS time_diff
  FROM prev_trans
)

SELECT COUNT(transaction_id) AS repeated_payment_count 
FROM prev_time_diff
WHERE time_diff <= INTERVAL '10 minutes';
```
