## Section 13 - Using Subqueries

### 106. Using Subqueries

- nested queires
- 가장 간단한 정의는 쿼리 안의 쿼리. SQL SELECT 문이 다른 SELECT 문 안에 포함된 것이다.
- 한 사람보다 월급이 높은 사람을 찾으려고 한다면 그 사람의 월급을 먼저 찾고 그 다음에 누가 더 많이 버는지 찾아내야 한다.

```sql
SELECT salary FROM employees
WHERE employee_id = 145;

SELECT * FROM employees
WHERE salary > 14000;

-- 하지만 존의 월급이 올랐다면 1년 후에는 2번 sql문을 수동으로 변경해야 한다.
```

- inner Query(sub query)는 먼저 수행된 후 그 결과를 사용하러 outer query(main, parent query)로 간다. outer query가 수행된 후 최종 결과가 나온다. 
- 서브쿼리는 parentheses(괄호) 안에 쓰며 SELECT, WHERE, HAVING, FROM 절과 함께 쓰일 수 있다.

```sql
SELECT first_name, last_name, salary
FROM employees
WHERE salary >
            (SELECT salary
              FROM employees
              WHERE employee_id = 145);
```

- 3가지 서브쿼리
  - Single-row
  - Multiple-row
  - Multiple-column

### 107. Single Row Subqueries

- inner query에서 오직 한 행만 반환하며 single-row 비교 연산자와 사용(=, >, <, >=, <=, <>, !=)

```sql
SELECT * FROM employees
WHERE department_id = 
                    (SELECT department_id FROM employees
                    WHERE employee_id = 145);
```

- 하나 이상의 서브 쿼리를 한 메인 쿼리에서 사용할 수 있다.

```sql
SELECT * FROM employees
WHERE department_id = 
                    (SELECT department_id FROM employees
                    WHERE employee_id = 145)
AND salary <
            (SELECT salary FROM employees
            WHERE employee_id = 145);
```

- group function을 사용할 수 있다.

```sql
SELECT * FROM employees
WHERE hire_date = 
                (SELECT min(hire_date) FROM employees);
```

- single-row 서브쿼리가 여러 행을 반환하면 에러 발생. nothing or NULL 반환시 메인쿼리는 아무것도 반환하지 않음.

```sql
-- 에러 발생
SELECT * FROM employees
WHERE hire_date = 
                (SELECT min(hire_date) FROM employees
                  GROUP BY department_id);
```

### 108. Multiple Row Subqueries

- inner queries가 한 개 이상의 행을 반환
- IN, ANY, ALL 을 사용한다.
  - IN : 가장 많이 사용됨. 서브 쿼리중 맞는 값
  - ANY: 서브쿼리와 맞는 값 적어도 하나 / 비교 연산자와 함께 사용한다.(=, >, <, >=, <=, <>, !=) / < ANY means less than the maximum / =ANY means equal to one of the elements / > ANY means more than the minimum
  - ALL: 서브쿼리와 맞는 값 모두 / 비교 연산자와 함께 사용한다.(=, >, <, >=, <=, <>, !=) / < ALL means less than the minimum / =ALL means nothing if there are more than one records /  >ALL means more than the maximum
- FROM, WHERE, HAVING절과 함께 사용

```sql
-- IN
SELECT first_name, last_name, department_id, salary
FROM employees
WHERE salary IN (14000, 15000, 10000);

SELECT first_name, last_name, department_id, salary
FROM employees
WHERE salary IN (SELECT min(salary)
                 FROM employees
                 GROUP BY department_id);

-- ANY
SELECT first_name, last_name, department_id, salary
FROM employees
WHERE salary > ANY (SELECT salary
                 FROM employees
                 WHERE job_id = 'SA_MAN');

-- ALL
SELECT first_name, last_name, department_id, salary
FROM employees
WHERE salary > ALL (SELECT salary
                 FROM employees
                 WHERE job_id = 'SA_MAN');

SELECT first_name, last_name, department_id, salary
FROM employees
WHERE department_id IN (SELECT department_id
                 FROM departments
                 WHERE location_id IN (SELECT location_id
                                       FROM locations
                                       WHERE country_id IN (SELECT country_id
                                                               FROM countries
                                                               WHERE country_name = 'UK')));
```

### 109. Multiple Column Subqueries

- outer query에 한 개 이상의 column 반환
- FROM, WHERE, HAVING 절과 쓰임
- 일반적으로 한 outer query에 하나 이상의 inner query를 사용할 때 유용
- 보통 IN, NOT IN과 함께 사용
- 2종류
  - Non-pairwise Comparison Subquery: multiple-row subqueries가 IN과 함께 사용된 것으로 여러 열 값을 각각의 서브쿼리와 비교한다. 독립적으로 각각의 값을 비교한다.
  - Pairwise Comparison Subquery: pair로 비교하기 때문에 non-pairwise과 결과가 다르다.

```sql
-- Non-pairwise Comparison Subquery
SELECT employee_id, first_name, last_name, department_id, salary
FROM employees
WHERE department_id IN 
                    (SELECT department_id FROM employees
                       WHERE employee_id IN (103,105,110))
AND salary IN
            (SELECT salary FROM employees
              WHERE employee_id IN (103,105,110));

-- Pairwise Comparison Subquery
SELECT employee_id, first_name, last_name, department_id, salary
FROM employees
WHERE (department_id,salary) IN 
                    (SELECT department_id,salary FROM employees
                       WHERE employee_id IN (103,105,110))
```

### 110. Using Subqueries as a Table

- Inline Views
- 서브쿼리를 FROM 뒤에 써서 table처럼 사용할 수 있다. 하나 이상 사용할 수 있다. 반드시 괄호 안에 써야한다.

```sql
-- *에 언급하지 않은 column 명을 쓰면 오류가 발생한다. *로 바꾸면 가능
SELECT *
FROM (SELECT department_id, department_name, state_province, city FROM departments JOIN locations
      USING (location_id)
      ORDER BY department_id);


SELECT e.employee_id, e.first_name, e.last_name, b.department_name, b.city
FROM employees e JOIN (SELECT department_id, department_name, state_province, city FROM departments JOIN locations
      USING (location_id)
      ORDER BY department_id) b
USING (department_id);
```

### 111. SCALAR Subqueries

- 서브쿼리가 한 행에 대해 한 열만 반환하면 Scalar Subquery라고 한다.
- 만약 NULL이나 0을 반환하면 메인 쿼리는 아무것도 반환하지 않는다.
- 한 행 이상을 반환하면 메인 쿼리는 에러가 발생한다.
- SELECT절, DECODE function과 CASE 표현, WHERE절, UPDATE절의 SET, INSERT의 VALUES, ORDER BY에서 사용된다.

```sql
-- 
SELECT avg(salary) FROM employees;

SELECT first_name, last_name, job_id
FROM employees
WHERE salary > (SELECT avg(salary) FROM employees);

-- 
SELECT department_id FROM employees
WHERE department_id = (SELECT department_id FROM employees
                     WHERE first_name = 'Luis');

-- 
SELECT employee_id, first_name, last_name,
       (CASE
               WHEN location_id = (SELECT location_id FROM locations WHERE postal_code = '99236') THEN             'San Francisco' ELSE 'Other'
        END) city
FROM employees e NATURAL JOIN departments d;
```

### 112. Correlated Subqueries

- 서브쿼리가 부모 쿼리의 column을 참조하면 correlated subquery(상호 연관 쿼리)라고 한다.
- correlated subqueries는 테이블의 행을 모두 읽고 각각의 값을 연결된 데이터와 비교한다.(메인 쿼리 행마다 서브쿼리가 실행된다.)
- 일반 서브쿼리는 처음 실행되고 결과가 parent query로 넘어간다. 따라서 메인 쿼리가 없어도 실행값이 나온다.
- corrleated subqueries에서 후보 행은 메인 쿼리에서 선택되어 사용되고 결과는 나중에 메인 쿼리에서 사용된다.
- repeating subquery, sunchronized subquery라고도 한다.
- 논리 연산자와 IN, ANY, ALL 연산자와 함께 사용할 수 있다.
- join을 쓸 때보다 성능이 안좋을 수 있으니 주의해서 사용하기. 

```sql
-- WHERE절의 조건 없이 메인 쿼리가 먼저 실행된다.
SELECT employee_id, first_name, last_name, department_id, salary
FROM employees a
WHERE salary = (SELECT max(salary) FROM employees b
                   WHERE b.department_id = a.department_id);

-- multiple columns subqueries가 correlated subqueries보다 성능이 좋을 때가 많다. / 같은 결과
SELECT employee_id, first_name, last_name, department_id, salary
FROM employees a
WHERE (salary, department_id) IN (SELECT max(salary), department_id 
                                  FROM employees b
                                   GROUP BY department_id);

-- 하지만 상호 연관 쿼리로만 가능한 연산도 있다.
SELECT employee_id, first_name, last_name, department_id, salary
FROM employees a
WHERE salary < (SELECT max(salary) FROM employees b
                   WHERE b.department_id = a.department_id);

-- 하지만 join으로도 할 수 있고 성능도 더 낫지만 correlated가 더 좋은 것도 있다.
SELECT employee_id, first_name, last_name, department_id, salary
FROM employees a
JOIN salary < (SELECT avg(salary) avg_sal, department_id FROM employees
                   GROUP BY department_id) b
ON a.department_id = b.department_id)
WHERE a.salary < b.avg_sal;

-- corrleated scalar subquery
SELECT employee_id, first_name, last_name, department_name, salary
        (SELECT round(avg(salary))
          FROM employees
          WHERE department_id = d.department_id) "DEPARTMENT'S AVERAGE SALARY"
FROM employees e JOIN departments d
ON (e.department_id = d.department_id);
```

### 117. EXISTS Operator & Semijoins

- subquery에서 행의 존재를 확인하고 메인 쿼리와 서브 쿼리의 레코드를 매치할 때 사용한다. 보통 상호연관된 서브쿼리에서 사용한다.

- 서브쿼리가 첫 행을 반환하면 과정을 종료한다. 상수, null 등을 반환해도 된다.

- 서브쿼리는 하나 이상의 값을 반환할 수도 있다.

```sql
SELECT employee_id, first_name, last_name, department_id
FROM employees a
WHERE EXISTS
           (SELECT 1, employee_id -- 이 두개는 그다지 의미없는 값이다. null도
            FROM employees
            WHERE manager_id = a.employee_id); 
        -- 중요한 건 여기서 a 테이블을 사용해 correlated하게 만든다는 것이다.
        -- 하나의 일치되는 결과만 찾아도 중단한다.
```

- 데이터가 크면 EXISTS가 낫고 작으면 IN이 낫다.

- Semijoin: ?



### 118. NOT EXISTS Operator

- exists의 반대로 메인 쿼리의 값이 서브 퀴리에 없을 때 테스트하기 위해 사용한다. 

```sql
-- 직원이 없는 부서들을 볼 수 있다.
SELECT * FROM departments d
WHERE NOT EXISTS 
                (SELECT null FROM employees e)
                 WHERE e.department_id = d. department_id);
```

- NOT IN 과 같은 결과를 낼 때도 있고 다른 결과를 낼 때도 있다.
