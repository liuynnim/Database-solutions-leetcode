# Intuition
The problem asks us to find classes that have at least 5 students. The natural idea is to group records by `class` and count the number of students in each group. Then, we filter out classes that do not meet the threshold.

# Approach
- Use `GROUP BY class` to group all records by class.
- Apply the aggregate function `COUNT(student)` to count the number of students in each class.
- Use `HAVING COUNT(student) > 4` to keep only classes with at least 5 students.
- Finally, select the class names that satisfy the condition.

# Complexity
- **Table scan**: The database must read all `n` rows in the `Courses` table → O(n).
- **Grouping**:
  - If the database uses a hash table or has an index on `class`, grouping can be done in linear time → O(n).
  - If the database needs to sort by `class` first, the cost becomes O(n log n).
- **Counting and filtering**: Each record increments a counter for its group, then groups are filtered → O(n).

👉 Overall:
- Best case (hash grouping or index): **O(n)**.
- Worst case (requires sorting): **O(n log n)**.

- **Space complexity**: The database must store groups in memory. The maximum number of groups equals the number of distinct classes → O(k), where *k* is the number of unique classes.

# Comparison
- **Nested subquery**: For each row, a subquery scans the table again → O(n²). This is inefficient for large datasets.
- **Window function (`COUNT(...) OVER (PARTITION BY class)`)**: Scans the table once, computes aggregates per partition, and attaches results to each row → O(n) (roughly 2n operations: scan + assign).
- **GROUP BY**: Groups and counts in one pass, typically O(n), sometimes O(n log n) if sorting is required.

👉 Therefore, `GROUP BY + HAVING` is the most concise and efficient solution for this problem, producing the desired summary directly.

# Code
```mysql
# Write your MySQL query statement below
SELECT class
FROM Courses
GROUP BY class
HAVING COUNT(student) > 4;
