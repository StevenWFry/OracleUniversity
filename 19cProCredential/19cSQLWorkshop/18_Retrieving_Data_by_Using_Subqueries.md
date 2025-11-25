# Chapter 16: Retrieving Data by Using Subqueries

## Unit 5, Lesson 16

### Table of Contents
- [Lesson Objectives](#lesson-objectives)
- [Introduction to Subqueries](#introduction-to-subqueries)
- [Multiple-Column Subqueries](#multiple-column-subqueries)
- [Pairwise Comparison Subqueries](#pairwise-comparison-subqueries)
- [Nonpairwise Comparison Subqueries](#nonpairwise-comparison-subqueries)
- [Scalar Subqueries](#scalar-subqueries)
- [Correlated Subqueries](#correlated-subqueries)
- [EXISTS and NOT EXISTS Operators](#exists-and-not-exists-operators)
- [The WITH Clause](#the-with-clause)
- [Recursive WITH Clause](#recursive-with-clause)
- [Conclusion](#conclusion)
- [Practice](#practice)

### Lesson Objectives
After completing this lesson, you should be able to:
- Write a multiple-column subquery.
- Use scalar subqueries in SQL.
- Solve problems with correlated subqueries.
- Use the EXISTS and NOT EXISTS operators.
- Use the WITH clause, including recursive forms.

### Introduction to Subqueries
A subquery is a SELECT statement nested inside another SQL statement. Subqueries let you validate data or build intermediate result sets before finalizing the outer query. Common uses include testing joins, preparing data for a view, or simplifying complex filters.

### Multiple-Column Subqueries
Multiple-column subqueries return more than one column, and each row in the outer query is compared against the set of rows returned by the subquery.

**Example problem:** Display employees who work with John and are managed by John's manager, excluding John himself.
- Employees who work with John likely share his department.
- Employees managed by John's manager share that manager ID.
- Use the specific employee to avoid matching on other people named John.

### Pairwise Comparison Subqueries
Pairwise comparisons evaluate column pairs together so that only matching combinations are returned. This ensures you filter on the exact column pairs rather than all possible combinations.

### Nonpairwise Comparison Subqueries
Nonpairwise comparisons evaluate each column independently. This can return a broader set of rows than a pairwise comparison because values are not tied together by row.

### Scalar Subqueries
A scalar subquery returns exactly one column and one row. Scalar subqueries can appear anywhere an expression is allowed, including SELECT lists, CASE expressions, and functions like DECODE.

```sql
SELECT employee_id,
       last_name,
       CASE
           WHEN department_id = (
               SELECT department_id
               FROM departments
               WHERE department_name = 'Sales'
           ) THEN 'Sales'
           ELSE 'Other'
       END AS department_type
FROM employees;
```

### Correlated Subqueries
Correlated subqueries execute once for every row in the outer query, using values from that row in the subquery. They are useful for row-by-row comparisons such as department-level averages.

```sql
SELECT e1.last_name
FROM employees e1
WHERE e1.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e1.department_id
);
```

### EXISTS and NOT EXISTS Operators
The EXISTS operator tests for the presence of rows returned by a subquery; NOT EXISTS checks for absence. The subquery often uses a correlated condition to tie it to the outer row.

```sql
SELECT last_name
FROM employees e
WHERE EXISTS (
    SELECT 1
    FROM employees m
    WHERE e.employee_id = m.manager_id
);
```

### The WITH Clause
The WITH clause (common table expression) defines reusable query blocks inside a statement, improving clarity and sometimes performance.

```sql
WITH avg_salary AS (
    SELECT department_id, AVG(salary) AS average_salary
    FROM employees
    GROUP BY department_id
)
SELECT e.employee_id, e.salary, a.average_salary
FROM employees e
JOIN avg_salary a ON e.department_id = a.department_id
WHERE e.salary > a.average_salary;
```

### Recursive WITH Clause
A recursive WITH clause repeatedly executes a union of an initial query (anchor) and a recursive query until no new rows are produced. This pattern suits hierarchical data such as org charts, file systems, or route calculations.

```sql
WITH RECURSIVE reachable_from AS (
    SELECT source, destination, flight_time
    FROM flights
    UNION ALL
    SELECT f.source, r.destination, f.flight_time + r.flight_time
    FROM flights f
    JOIN reachable_from r ON f.destination = r.source
)
SELECT *
FROM reachable_from;
```

### Conclusion
You can combine subqueries with pairwise or nonpairwise comparisons, scalar expressions, correlated logic, EXISTS tests, and common table expressions (regular or recursive) to answer complex questions while keeping SQL readable.

### Practice
Practice these skills:
- Create multiple-column subqueries.
- Write correlated subqueries.
- Use the EXISTS operator.
- Use scalar subqueries.
- Use the WITH clause (including recursive CTEs).
