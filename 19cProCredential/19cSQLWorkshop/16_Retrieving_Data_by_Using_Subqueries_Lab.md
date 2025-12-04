# Lesson 16 Lab – Retrieving Data by Using Subqueries (Inception, But For `SELECT` Statements)

And look, at some point a single `SELECT` stops being enough and you start nesting queries inside queries like a Russian doll of database pain. This lab is where we make that pain productive.

You’ll practice:

- Multiple‑column subqueries (pairwise comparisons)
- Correlated subqueries
- `EXISTS` / `NOT EXISTS`
- Scalar subqueries
- The `WITH` clause as a sanity‑preserving device

Schema: `ora21`, `EMPLOYEES`, `DEPARTMENTS`, `LOCATIONS`.

---

## Task 1 – Match salary + department with “people who get commission”

Goal: employees whose **salary and department** match someone who earns a commission.

Multiple‑column subquery:

```sql
SELECT last_name,
       department_id,
       salary,
       commission_pct
FROM   employees
WHERE  (salary, department_id) IN (
         SELECT salary,
                department_id
         FROM   employees
         WHERE  commission_pct IS NOT NULL
       );
```

Key point: `(salary, department_id)` is matched as a **pair** against the subquery results.

---

## Task 2 – Match salary + job with employees in location 1700

Goal:  
> Last name, department name, and salary for any employee whose **salary and job** match an employee who works in location `1700`.

```sql
SELECT e.last_name,
       d.department_name,
       e.salary
FROM   employees  e
JOIN   departments d
       ON e.department_id = d.department_id
WHERE  (e.salary, e.job_id) IN (
         SELECT e2.salary,
                e2.job_id
         FROM   employees  e2
         JOIN   departments d2
                ON e2.department_id = d2.department_id
         WHERE  d2.location_id = 1700
       );
```

This is another pairwise multiple‑column subquery, just to prove the first one wasn’t a fluke.

---

## Task 3 – Same salary and manager as Kochhar (but not Kochhar)

Goal: find coworkers with the **same salary** and **same manager** as Kochhar, excluding Kochhar.

Step 1 – subquery to get Kochhar’s salary and manager:

```sql
SELECT salary,
       manager_id
FROM   employees
WHERE  last_name = 'Kochhar';
-- 17000, 100
```

Step 2 – main query:

```sql
SELECT last_name,
       hire_date,
       salary
FROM   employees
WHERE  (salary, manager_id) IN (
         SELECT salary,
                manager_id
         FROM   employees
         WHERE  last_name = 'Kochhar'
       )
AND    last_name <> 'Kochhar';
```

You should see `De Haan`, who is apparently playing “clone the VP” with compensation.

---

## Task 4 – Earn more than **all** sales managers

Sales managers (`job_id = 'SA_MAN'`) have opinions. Let’s find the people who are paid more than *every* one of them.

```sql
SELECT last_name,
       job_id,
       salary
FROM   employees
WHERE  salary > ALL (
         SELECT salary
         FROM   employees
         WHERE  job_id = 'SA_MAN'
       )
ORDER  BY salary DESC;
```

`> ALL` = “greater than the maximum in that set.”  
It’s basically `salary > (SELECT MAX(salary) …)` but more explicit.

---

## Task 5 – Employees in cities starting with “T”

Goal: employee ID, last name, and department ID for employees working in **cities whose names start with `T`**.

Step 1 – locations where the city starts with `T`:

```sql
SELECT location_id
FROM   locations
WHERE  city LIKE 'T%';
```

Step 2 – departments in those locations:

```sql
SELECT department_id
FROM   departments
WHERE  location_id IN (
         SELECT location_id
         FROM   locations
         WHERE  city LIKE 'T%'
       );
```

Step 3 – employees in those departments:

```sql
SELECT employee_id,
       last_name,
       department_id
FROM   employees
WHERE  department_id IN (
         SELECT department_id
         FROM   departments
         WHERE  location_id IN (
                  SELECT location_id
                  FROM   locations
                  WHERE  city LIKE 'T%'
                )
       );
```

This is nesting subqueries like matryoshka dolls, but at least it’s logically tidy.

---

## Task 6 – Earn more than the **average** salary in your department

Correlated subquery time.

Goal: last name, salary, department ID, and **average department salary** for employees whose salary is above their department’s average.

```sql
SELECT e.last_name       AS ename,
       e.salary          AS salary,
       e.department_id   AS deptno,
       ROUND(AVG(e.salary)
             OVER (PARTITION BY e.department_id), 2) AS dept_avg
FROM   employees e
WHERE  e.salary >
       (SELECT AVG(i.salary)
        FROM   employees i
        WHERE  i.department_id = e.department_id)
ORDER  BY dept_avg;
```

If you want to stay truer to the transcript and avoid analytic functions, you can also do:

```sql
SELECT e.last_name       AS ename,
       e.salary          AS salary,
       e.department_id   AS deptno,
       ROUND(a.dept_avg, 2) AS dept_avg
FROM   employees e
JOIN   employees a
       ON e.department_id = a.department_id
GROUP  BY e.last_name, e.salary, e.department_id
HAVING e.salary >
       (SELECT AVG(i.salary)
        FROM   employees i
        WHERE  i.department_id = e.department_id)
ORDER  BY dept_avg;
```

Either way, the key is the correlated `AVG` restricted per department.

---

## Task 7 – Non‑managers with `NOT EXISTS` (and why `NOT IN` is cursed)

**Part A – `NOT EXISTS` solution**

```sql
SELECT e.last_name
FROM   employees e
WHERE  NOT EXISTS (
         SELECT 1
         FROM   employees m
         WHERE  m.manager_id = e.employee_id
       );
```

This returns all employees who are **never referenced** as a `manager_id` – i.e. non‑supervisors.

**Part B – The `NOT IN` “solution” that returns nothing**

```sql
SELECT e.last_name
FROM   employees e
WHERE  e.employee_id NOT IN (
         SELECT manager_id
         FROM   employees
       );
```

Because the subquery returns at least one `NULL` manager, the `NOT IN` comparison becomes “maybe” for every row, which SQL interprets as “no”.

Fixing it with an extra predicate (but still worse than `NOT EXISTS`):

```sql
SELECT e.last_name
FROM   employees e
WHERE  e.employee_id NOT IN (
         SELECT manager_id
         FROM   employees
         WHERE  manager_id IS NOT NULL
       );
```

**Moral:** if `NULL` is anywhere in the subquery, **do not** use `NOT IN`. Reach for `NOT EXISTS` instead.

---

## Task 8 – Employees earning less than their department’s average

As a depressing mirror image of Task 6:

```sql
SELECT e.last_name
FROM   employees e
WHERE  e.salary <
       (SELECT AVG(i.salary)
        FROM   employees i
        WHERE  i.department_id = e.department_id);
```

You should get a sizeable list. Try not to think too hard about the morale implications.

---

## Task 9 – Coworkers hired later **and** earning more

Goal: last names of employees who have **at least one coworker** in the same department who:

- Was hired **later**  
- Earns a **higher** salary

Correlated + `EXISTS`:

```sql
SELECT e.last_name
FROM   employees e
WHERE  EXISTS (
         SELECT 1
         FROM   employees c
         WHERE  c.department_id = e.department_id
         AND    c.hire_date    > e.hire_date
         AND    c.salary       > e.salary
       );
```

If you want to feel under‑valued, imagine being on this list.

---

## Task 10 – Scalar subquery for department name

Goal: employee ID, last name, and department **name**, using a scalar subquery in the `SELECT` list.

```sql
SELECT e.employee_id,
       e.last_name,
       (SELECT d.department_name
        FROM   departments d
        WHERE  d.department_id = e.department_id) AS department
FROM   employees e
ORDER  BY department, e.last_name;
```

This is essentially a join written in scalar‑subquery form. Use sparingly in large queries, but it demonstrates the pattern nicely.

---

## Task 11 – Departments whose total salary is > 1/8 of company total (WITH clause)

Time to use a common table expression so your query doesn’t turn into unreadable spaghetti.

Step 1 – Create the `WITH` query `summary`:

```sql
WITH summary AS (
  SELECT d.department_name,
         SUM(e.salary) AS dept_total
  FROM   employees  e
  JOIN   departments d
         ON e.department_id = d.department_id
  GROUP  BY d.department_name
)
SELECT department_name,
       dept_total
FROM   summary
WHERE  dept_total >
       (SELECT SUM(dept_total) * (1/8)
        FROM   summary)
ORDER  BY dept_total DESC;
```

What’s happening:

1. `summary` computes total salary per department.  
2. The outer query compares each `dept_total` to **one‑eighth** of the sum of all `dept_total` values.  
3. Only departments that individually consume more than 12.5% of the salary budget make the cut.

And if that doesn’t spark a conversation about “concentration of payroll spend,” nothing will.

---

At this point you’ve:

- Written multiple‑column subqueries  
- Used correlated subqueries with `EXISTS` / `NOT EXISTS`  
- Dropped scalar subqueries into `SELECT` lists  
- Used the `WITH` clause to tame complex logic  

…which means you’re now fully qualified to write queries that work perfectly and terrify whoever has to read them next. Use your powers wisely.  

