# Chapter 16: Retrieving Data by Using Subqueries

## Unit 5, Lesson 16

### Lesson Objectives
After completing this lesson, you should be able to:
- Write a multiple-column subquery.
- Use scalar subqueries in SQL.
- Solve problems with correlated subqueries.
- Use the EXISTS and NOT EXISTS operators.
- Use the WITH clause.

### Introduction to Subqueries
Retrieving data using a subquery as a source can be beneficial. For example, you might want to create a view and check if the syntax for the view is suitable to join two other tables. You can write a SELECT statement as a subquery to verify the data before creating the view.

### Multiple-Column Subqueries
In multiple-column subqueries, each row of the main query is compared to values from a multiple-row and multiple-column subquery. 

**Example Problem:** Write a query to display employees who work with John and are managed by John's manager, excluding John himself.

- If employees work with John, they likely share the same department.
- If they are managed by John's manager, they are under the same management.

When running the SELECT statement, you may find multiple department IDs and managers associated with different Johns. To refine the query, you can specify to display employees who work with the specific John and are managed by that John's manager.

### Pairwise Comparison Subqueries
A pairwise comparison subquery looks at paired results. For example, if you want to find employees in department 50 managed by manager 123, you would need to ensure that the query only returns specific combinations rather than all possible combinations.

### Nonpairwise Comparison Subqueries
Nonpairwise comparisons involve breaking out columns individually. This can yield different datasets compared to pairwise comparisons.

### Scalar Subqueries
A scalar subquery returns exactly one column from one row and can be used in various SQL clauses, including the DECODE function and CASE expressions.

**Example:** Using a scalar subquery in a CASE expression:
```sql
SELECT employee_id,
       last_name,
       CASE WHEN department_id = (SELECT department_id FROM departments WHERE department_name = 'Sales') 
            THEN 'Sales' 
            ELSE 'Other' 
       END AS department_type
FROM employees;
```

### Correlated Subqueries
Correlated subqueries operate differently than standard subqueries. The outer query provides a value that is used in the inner query. This process continues until all rows are processed.

**Example Syntax:**
```sql
SELECT e1.last_name
FROM employees e1
WHERE e1.salary > (SELECT AVG(e2.salary) 
                   FROM employees e2 
                   WHERE e2.department_id = e1.department_id);
```

### EXISTS and NOT EXISTS Operators
The EXISTS operator tests for the existence of rows in the results set of the subquery. If a row value is found, the condition is TRUE; if not, it is FALSE.

**Example:**
```sql
SELECT last_name
FROM employees e
WHERE EXISTS (SELECT 1 
              FROM employees m 
              WHERE e.employee_id = m.manager_id);
```

### The WITH Clause
The WITH clause allows you to define a query block that can be reused within a SELECT statement. This can improve performance by storing the results temporarily.

**Example:**
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
The Recursive WITH clause enables the formulation of recursive queries, complying with ANSI standards.

**Example:**
```sql
WITH RECURSIVE Reachable_From AS (
    SELECT source, destination, flight_time
    FROM FLIGHTS
    UNION ALL
    SELECT f.source, r.destination, f.flight_time + r.flight_time
    FROM FLIGHTS f
    JOIN Reachable_From r ON f.destination = r.source
)
SELECT * FROM Reachable_From;
```

### Conclusion
In this lesson, you learned how to write multiple-column subqueries, use scalar subqueries, solve problems with correlated subqueries, utilize the EXISTS and NOT EXISTS operators, and implement the WITH clause. 

### Practice
This practice covers the following topics:
- Creating multiple-column subqueries.
- Writing correlated subqueries.
- Using the EXISTS operator.
- Using scalar subqueries.
- Using the WITH clause.

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