 
# SQL Queries

## Important queries to practise for SQL
- `CASE WHEN`
- Self joins.
- `DISTINCT` and `GROUP BY`
- Left vs Outer Joins
- `SUM` and `COUNT`
- Date-time manipulation
- String formatting, substring
- Window functions like `RANK` and `ROW`
- Subqueries
- `HAVING` and `WHERE`
- `LAG` and `LEAD`
- Understand indexes
- Running totals
- `MIN` and `MAX`

<details>
	<summary> Conditional Logic in a SELECT Statement </summary>
Q :
PROBLEM
You want to perform IF-ELSE operations on values in your SELECT statement. For example, you would like to produce a result set such that, if an employee is paid $2000 or less, a message of “UNDERPAID” is returned, if an employee is paid $4000 or more, a message of “OVERPAID” is returned,
if they make somewhere in between, then “OK” is returned. The result set should look like this:
	
```
	ENAME             SAL  STATUS
	---------- ----------  ---------
	SMITH             800  UNDERPAID
	ALLEN            1600  UNDERPAID
	WARD             1250  UNDERPAID
	JONES            2975  OK
	MARTIN           1250  UNDERPAID
	BLAKE            2850  OK
	CLARK            2450  OK
	SCOTT            3000  OK
	KING             5000  OVERPAID
	TURNER           1500  UNDERPAID
	ADAMS            1100  UNDERPAID
	JAMES             950  UNDERPAID
	FORD            3000   OK
	MILLER          1300   UNDERPAID
```  
  
  Ans 
  ```
  1 select ename,sal,
	2        case when sal <= 2000 then 'UNDERPAID'
	3             when sal >= 4000 then 'OVERPAID'
	4             else 'OK'
	5        end as status
	6   from emp
  ```
 </details>
 
 
 
<details>
	<summary> Transforming Nulls into Real Values </summary>
	
**PROBLEM**
You have rows that contain nulls and would like to return non-null values in place of those nulls.

**SOLUTION**
Use the function COALESCE to substitute real values for nulls:

	1 select coalesce(comm,0)
	2   from emp
	
</details>
 
 
<details>
	<summary>Dealing with Nulls when Sorting</summary>
PROBLEM
You want to sort results from EMP by COMM, but the field is nullable. You need a way to specify whether nulls sort last:

	ENAME              SAL        COMM
	----------  ----------  ----------
	TURNER            1500           0
	ALLEN             1600         300
	WARD              1250         500
	MARTIN            1250        1400
	SMITH              800
	JONES             2975
	JAMES              950
	MILLER            1300
	FORD              3000
	ADAMS             1100
	BLAKE             2850
	CLARK             2450
	SCOTT             3000
	KING              5000
or whether they sort first:

	ENAME              SAL        COMM
	----------  ----------  ----------
	SMITH              800
	JONES             2975
	CLARK             2450
	BLAKE             2850
	SCOTT             3000
	KING              5000
	JAMES              950
	MILLER            1300
	FORD              3000
	ADAMS             1100
	MARTIN            1250        1400
	WARD              1250         500
	ALLEN             1600         300
	TURNER            1500           0
SOLUTION

Depending on how you want the data to look (and how your particular RDBMS sorts NULL values), you can sort the nullable column in ascending or descending order:

	1 select ename,sal,comm
	2   from emp
	3  order by 3

	1 select ename,sal,comm
	2   from emp
	3  order by 3 desc
	
This solution puts you in a position such that if the nullable column contains non-NULL values, they will be sorted in ascending or descending order as well, according to what you ask for; this may or may not what you have in mind. If instead you would like to sort NULL values differently than non-NULL values, for example, you want to sort non-NULL values in ascending or descending order and all NULL values last, you can use a CASE expression to conditionally sort the column.

DB2, MySQL, PostgreSQL, and SQL Server
Use a CASE expression to “flag” when a value is NULL. The idea is to have a flag with two values: one to represent NULLs, the other to represent non-NULLs. Once you have that, simply add this flag column to the ORDER BY clause. You’ll easily be able to control whether NULL values are sorted first or last without interfering with non-NULL values:

	/* NON-NULL COMM SORTED ASCENDING, ALL NULLS LAST */
	1  select ename,sal,comm
	2    from (
	3  select ename,sal,comm,
	4         case when comm is null then 0 else 1 end as is_null
	5    from emp
	6         ) x
	7    order by is_null desc,comm

	ENAME     SAL        COMM
	------  -----  ----------
	TURNER   1500           0
	ALLEN    1600         300
	WARD     1250         500
	MARTIN   1250        1400
	SMITH     800
	JONES    2975
	JAMES     950
	MILLER   1300
	FORD     3000
	ADAMS    1100
	BLAKE    2850
	CLARK    2450
	SCOTT    3000
	KING     5000

	/* NON-NULL COMM SORTED DESCENDING, ALL NULLS LAST */

	
	1  select ename,sal,comm
	2    from (
	3  select ename,sal,comm,
	4         case when comm is null then 0 else 1 end as is_null
	5    from emp
	6         ) x
	7   order by is_null desc,comm desc

	ENAME     SAL        COMM
	------  -----  ----------
	MARTIN   1250        1400
	WARD     1250         500
	ALLEN    1600         300
	TURNER   1500           0
	SMITH     800
	JONES    2975
	JAMES     950
	MILLER   1300
	FORD     3000
	ADAMS    1100
	BLAKE    2850
	CLARK    2450
	SCOTT    3000
	KING     5000

	/* NON-NULL COMM SORTED ASCENDING, ALL NULLS FIRST */

	
	1 select ename,sal,comm
	2   from (
	3 select ename,sal,comm,
	4        case when comm is null then 0 else 1 end as is_null
	5   from emp
	6        ) x
	7  order by is_null,comm

	ENAME    SAL       COMM
	------ ----- ----------
	SMITH    800
	JONES   2975
	CLARK   2450
	BLAKE   2850
	SCOTT   3000
	KING    5000
	JAMES    950
	MILLER  1300
	FORD    3000
	ADAMS   1100           
	TURNER  1500          0
	ALLEN   1600        300
	WARD    1250        500
	MARTIN  1250       1400

	/* NON-NULL COMM SORTED DESCENDING, ALL NULLS FIRST */

	
	1  select ename,sal,comm
	2    from (
	3  select ename,sal,comm,
	4         case when comm is null then 0 else 1 end as is_null
	5    from emp
	6         ) x
	7   order by is_null,comm desc

	ENAME    SAL       COMM
	------ ----- ----------
	SMITH    800
	JONES   2975
	CLARK   2450
	BLAKE   2850
	SCOTT   3000
	KING    5000
	JAMES    950
	MILLER  1300
	FORD    3000
	ADAMS   1100
	MARTIN  1250       1400
	WARD    1250        500
	ALLEN   1600        300
	TURNER  1500          0
</details>


<details>
	<summary> Sorting on a Data Dependent Key </summary>
PROBLEM
You want to sort based on some conditional logic. For example: if JOB is “SALESMAN” you want to sort on COMM; otherwise, you want to sort by SAL. You want to return the following result set:

	ENAME             SAL JOB             COMM
	---------- ---------- --------- ----------
	TURNER           1500  SALESMAN          0
	ALLEN            1600  SALESMAN        300
	WARD             1250  SALESMAN        500
	SMITH             800  CLERK
	JAMES             950  CLERK
	ADAMS            1100  CLERK
	MILLER           1300  CLERK
	MARTIN           1250  SALESMAN       1400
	CLARK            2450  MANAGER
	BLAKE            2850  MANAGER
	JONES            2975  MANAGER
	SCOTT            3000  ANALYST
	FORD             3000  ANALYST
	KING             5000  PRESIDENT
SOLUTION
Use a CASE expression in the ORDER BY clause:

	1 select ename,sal,job,comm
	2   from emp
	3  order by case when job = 'SALESMAN' then comm else sal end
DISCUSSION
You can use the CASE expression to dynamically change how results are sorted. The values passed to the ORDER BY look as follows:

	select ename,sal,job,comm,
	       case when job = 'SALESMAN' then comm else sal end as ordered
	  from emp
	 order by 5

	ENAME             SAL JOB             COMM    ORDERED
	---------- ---------- --------- ---------- ----------
	TURNER           1500 SALESMAN           0          0
	ALLEN            1600 SALESMAN         300        300
	WARD1             250 SALESMAN         500        500
	SMITH             800 CLERK                       800
	JAMES             950 CLERK                       950
	ADAMS            1100 CLERK                      1100
	MILLER           1300 CLERK                      1300
	MARTIN           1250 SALESMAN        1400       1400
	CLARK2            450 MANAGER                    2450
	BLAKE2            850 MANAGER                    2850
	JONES2            975 MANAGER                    2975
	SCOTT            3000 ANALYST                    3000
	FORD             3000 ANALYST                    3000
	KING             5000 PRESIDENT                  5000
	
</details>

<details>
	<summary> Find keys present in one table and not in the other (with and w/o null)</summary>
	
	select distinct deptno
	  from dept
	 where deptno not in (select deptno from emp)


	select d.deptno
	  from dept d
	 where not exists (
	   select 1
	     from emp e
	    where d.deptno = e.deptno
	)
	
</details>

<details>
	<summary> SQL query to find employees who have the highest salary in each of the departments </summary>
	
# Write a SQL query to find employees who have the highest salary in each of the departments. For the above tables, your SQL query should return the following rows (order of rows does not matter).

```
SELECT 
    Department.name as 'Department',
    Employee.name as 'Employee',
    Employee.Salary 
FROM 
    Employee
    JOIN 
    Department
    ON (Employee.DepartmentId = Department.Id)
WHERE
    (Employee.DepartmentId, Employee.Salary) IN
    (   SELECT
            DepartmentId, MAX(SALARY)
        FROM
            Employee
        GROUP BY DepartmentId
        );
```
</details>


<details>
	<summary>TOP N Salaries from the entire company</summary>
	
```
SELECT DISTINCT Salary
FROM Employee A
WHERE 3 >= (SELECT COUNT(DISTINCT Salary) FROM Employee B WHERE A.Salary >= B.Salary)
ORDER BY A.Salary DESC;
```

</details>

<details>
	<summary> Select the top N salaries from each department </summary>


```
SELECT E.Name as 'Employee',
E.Salary,
D.Name as 'Department'
FROM
Employee E
JOIN
Department D
ON(E.DepartmentId = D.Id)
WHERE (
    
    3 > (
        SELECT COUNT(DISTINCT E2.Salary)
        FROM Employee E2 
        WHERE E2.Salary > E.Salary
        AND E.DepartmentId = E2.DepartmentId
    )
);
```

</details>

<details>
	<summary> Second Highest Salary </summary>

```
select max(salary) as 'SecondHighestSalary'
from Employee 
where salary not in (select max(salary) from Employee);
```

General for Nth max for max we sort in DESC, for min it would be ASC
For, OFFSET it is the index from which it will start displaying the answer so if result is 
`300,200,100`
If I say OFFSET is 1 and LIMIT is 1 it will display the second highest which is `200`.
If I say OFFSET is 2 and LIMIT is 1 it will display the third highest salary which is `100`.

```
Select (SELECT DISTINCT
    Salary
FROM
    Employee
ORDER BY Salary DESC
LIMIT 1 OFFSET 1) AS 'SecondHighestSalary'
```

</details>
 
 
<details>
	<summary> Trip Cancellation by Driver and/or Customer</summary>

```
SELECT
    Trips.Request_at as 'Day',
    round(sum(case when Trips.Status like 'cancelled_%' then 1 else 0 end)/count(*) ,2) as 'Cancellation Rate'
    FROM Trips
    JOIN
    Users
    ON(Trips.Client_Id = Users.Users_Id and Users.Banned='No')
WHERE Trips.Request_at BETWEEN '2013-10-01' AND '2013-10-03'
GROUP BY Trips.Request_at
```
	
</details>

<details>
	<summary>Higher Temperature compared to previous day</summary>
	
**Given a Weather table, write a SQL query to find all dates' Ids with higher temperature compared to its previous (yesterday's) dates.**
	
```
select w.Id from Weather w
join Weather w1
on (DATEDIFF(w.RecordDate, w1.RecordDate)=1)
where w.Temperature > w1.Temperature;
```

</details>


<details>
	<summary>DELETE Duplicates </summary>

**Write a SQL query to delete all duplicate email entries in a table named Person, keeping only unique emails based on its smallest Id.**
	
```
DELETE p1 from person p1,
    Person p2
WHERE
    p1.Email = p2.Email and p1.Id > p2.Id;
```
</details>

<details>
	<summary> </summary>
</details>



