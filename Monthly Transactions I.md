# Monthly Transactions I — Detailed Analysis and Comparison

# Intuition
The problem requires summarizing transactions per country and per month. The natural idea is to group records by `country`, but we also needs a `month` column in the format `YYYY-MM`, we must compute this from the transaction date. Therefore, the grouping must be on both `country` and `month`.

# Approach
- Extract the month from `trans_date` using `DATE_FORMAT(trans_date, '%Y-%m')`.
- Group by both `country` and the computed `month`.
- Within each group, compute:
  - `COUNT(*)` for the total number of transactions.
  - `SUM(CASE WHEN state = 'approved' THEN 1 ELSE 0 END)` for the number of approved transactions.
  - `SUM(amount)` for the total transaction amount.
  - `SUM(CASE WHEN state = 'approved' THEN amount ELSE 0 END)` for the total approved amount.
- Return the grouped results with the required columns.

# Complexity
### Step-by-step breakdown
1. **Table scan**: The database must read all `n` rows in the `Transactions` table → O(n).
2. **Grouping**:
   - If the database uses a hash table or has an index on `country` and `trans_date`, grouping can be done in linear time → O(n).
   - If the database needs to sort by `country` and `month`, the cost becomes O(n log n).
3. **Aggregate calculations**: Each record contributes to counters and sums in its group → O(n).

👉 **Overall complexity**:
- Best case (hash grouping or index): **O(n)**.
- Worst case (requires sorting): **O(n log n)**.

### Space complexity
- The database must store groups in memory. The maximum number of groups equals the number of distinct `(country, month)` pairs → O(k), where *k* is the number of unique pairs.

# Comparison
- **Group by country only**: Incorrect, because it collapses all months together and loses the time dimension.
- **Group by (country, month)**: Correct and efficient, producing the exact granularity required.
- **Window function (`COUNT(...) OVER (PARTITION BY country, month)`)**: Also possible, but attaches aggregates to each row and requires an extra DISTINCT or outer query to collapse results. More flexible, but less direct here.
- **Nested subquery**: Possible but inefficient (O(n²)), since the table would be scanned repeatedly for each country/month combination.

👉 Therefore, grouping by both `country` and `month` is the most concise and efficient solution for this problem.

# Code
```mysql
# Write your MySQL query statement below
SELECT 
    DATE_FORMAT(trans_date, '%Y-%m') AS month,
    country,
    COUNT(*) AS trans_count,
    SUM(CASE WHEN state = 'approved' THEN 1 ELSE 0 END) AS approved_count,
    SUM(amount) AS trans_total_amount,
    SUM(CASE WHEN state = 'approved' THEN amount ELSE 0 END) AS approved_total_amount
FROM Transactions
GROUP BY country, month;
