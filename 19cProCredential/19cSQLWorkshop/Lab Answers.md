## Lesson 3 – HR Queries

1. **Employees earning more than 12,000**

   ```sql
   SELECT last_name,
          salary
   FROM   employees
   WHERE  salary > 12000;
   ```

2. **Last name and department for employee 176**

   ```sql
   SELECT last_name,
          department_id
   FROM   employees
   WHERE  employee_id = 176;
   ```

3. **Employees not in the 5,000–12,000 salary range**

   ```sql
   SELECT last_name,
          salary
   FROM   employees
   WHERE  salary NOT BETWEEN 5000 AND 12000;
   ```

4. **Matos and Taylor – job and hire date, ordered by hire date**

   ```sql
   SELECT last_name,
          job_id,
          hire_date
   FROM   employees
   WHERE  last_name IN ('Matos', 'Taylor')
   ORDER  BY hire_date;
   ```

5. **Employees in departments 20 or 50, ordered by last name**

   ```sql
   SELECT last_name,
          department_id
   FROM   employees
   WHERE  department_id IN (20, 50)
   ORDER  BY last_name;
   ```

6. **Employees earning 5,000–12,000 in departments 20 or 50**

   ```sql
   SELECT last_name   AS "Employee",
          salary      AS "Monthly Salary"
   FROM   employees
   WHERE  salary BETWEEN 5000 AND 12000
   AND    department_id IN (20, 50);
   ```

