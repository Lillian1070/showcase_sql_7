# [SQL] Calculating Repeated Payments

_This SQL practice is based on a problem from [DataLemur](https://datalemur.com/questions/repeated-payments) and is intended for personal learning and educational purposes._

- **Objective**: Calculate the count of repeated payments.
- **Practice Purpose**: Self-learning and reinforcement of SQL data cleaning, aggregation, joins, subqueries, and window functions.
- **Outline**:
    - [**Practice**](#section-1) (practice problem and query output)
    - [**Solution**](#section-2) (step-by-step explanation)
    - [**Query Optimization**](#section-3)


## <a name="section-1"></a>🧪 Practice

Sometimes, payment transactions are repeated by accident; it could be due to user error, API failure or a retry error that causes a credit card to be charged twice.

Using the transactions table, identify any payments made at the same merchant with the same credit card for the same amount __within 10 minutes__ of each other. Count such repeated payments.

__Assumptions:__

- The first transaction of such payments should not be counted as a repeated payment. This means, if there are two transactions performed by a merchant with the same credit card and for the same amount within 10 minutes, there will only be 1 repeated payment.


__Table:__ `transactions`

| transaction_id | merchant_id | credit_card_id | amount | transaction_timestamp | 
| -------------- | ----------- | -------------- | ------ | --------------------- | 
| 1 | 101 | 1 | 100 | 09/25/2022 12:00:00 | 
| 2 | 101 | 1 | 100 | 09/25/2022 12:08:00 | 
| 3 | 101 | 1 | 100 | 09/25/2022 12:28:00 | 
| 4 | 102 | 2 | 300 | 09/25/2022 12:00:00 | 
| 6 | 102 | 2 | 400 | 09/25/2022 14:00:00 | 
| ... | ... | ... | ... | ... | 

### Expected_Output: 

| payment_count |
| ------------- |
| 1 |



## <a name="section-2"></a>🧠 Solution 

*This section outlines my thought process for solving the problem.*

### Step 1: Identify the Required Fields 



### Step 2a: Create a Temporary Table `prev_trans` to xxx

- Using [`LAG()`](https://www.geeksforgeeks.org/sql/sql-server-lag-function-overview/) to pull the previous same-amount payment to the same merchant made with the same card.

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
```


### Step 2b: Create a Temporary Table `prev_time_diff` 

- Calculate time between repeated same-amount payments to the same merchant with the same card.

```sql
prev_time_diff AS (
  SELECT 
    *,
    (transaction_timestamp - prev_transaction) AS time_diff
  FROM prev_trans
)
```

### Step 3: Calculate the Count of Repeated Payment 

- Using [`INTERVAL`](https://hightouch.com/sql-dictionary/sql-interval) to filter for payments where the previous one occurred within 10 minutes
- Using `COUNT()` to calculate the number of repeated payments

```sql
SELECT COUNT(transaction_id) AS repeated_payment_count 
FROM prev_time_diff
WHERE time_diff <= INTERVAL '10 minutes';
```


#### Final Syntax and Output using PostgreSQL

##### * Syntax

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
    (transaction_timestamp - prev_transaction) AS time_diff
  FROM prev_trans
)

SELECT COUNT(transaction_id) AS repeated_payment_count 
FROM prev_time_diff
WHERE time_diff <= INTERVAL '10 minutes';
```


##### * Output

| payment_count |
| ------------- |
| 1 |


## <a name="section-3"></a>🛠️ Query Optimization using PostgreSQL

*Note: This section was last updated on 07/22/2025.*

Upon reviewing my previous query, I realized it can be simplified into a single query without using additional CTEs.


```sql
SELECT COUNT(*) AS repeated_payment_count
FROM (
  SELECT 
    transaction_timestamp,
    LAG(transaction_timestamp) OVER (
      PARTITION BY merchant_id, credit_card_id, amount 
      ORDER BY transaction_timestamp
    ) AS prev_transaction
  FROM transactions
) t
WHERE transaction_timestamp - prev_transaction <= INTERVAL '10 minutes';
```

_💬 I’d love to hear your thoughts! If you have any suggestions or questions, please feel free to reach out._

