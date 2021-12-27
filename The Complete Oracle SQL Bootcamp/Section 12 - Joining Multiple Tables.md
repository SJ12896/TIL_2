## Section 12 - Joining Multiple Tables

### 89. What is a Join? & Oracle SQL Join Types

- 두 개 이상의 테이블을 한 쿼리에서 검색할 수 있게 해주는 것. 실제 테이블에 영향을 미치지 않는다.
- Normalization(정규화) :  key를 사용하고 특정 데이터를 테이블에 저장해 데이터 중복을 줄이는 것
- Oracle Join Types: Natural Join, Inner Join, Outer Join(LEFT OUTER, RIGHT OUTER, FULL OUTER), Equijoin, Non-Equijoin, Self Join, Cross Join(Cartesian product)

### 90. Creating a Join

- 양 테이블에 적어도 하나의 같은 칼럼이 있어야 한다. (same name, same data type) / 같은 값으로 연결한다.

```sql
-- 여러 JOIN_TYPE 중 하나를 사용. general syntax
SELECT columns FROM table1 JOIN_TYPE table2 ON table.column_name = table2.coulmn_name;
```

### 91. Natural Join

- Source table은 FROM에 쓰는 table, Target table은 JOIN 뒤에 사용하는 테이블 / WHERE 문으로 데이터 제한
- DESC 테이블명으로 테이블 내의 column 정보 확인
- NATUAL JOIN: ON 뒤의 join column을 따로 지정하지 않고 자동으로 매치한다. 공통 칼럼 / source table / target table순으로 보인다.
- 만약 함께 있는 칼럼에서 null 값이 있다면 JOIN의 결과에서 보이는 테이블에는 해당 행이 없다.

### 92. Join with the USING Clause

- 같은 이름의 칼럼이 여러개라면 USING을 사용해 특정할 수 있다. 이런 join을 Equijoin이라고 한다.

```sql
-- 이 때 같은 이름인 칼럼이 또 있다면 알아서 각각 나온다.
SELECT first_name, last_name, department_name, department_id 
FROM employees JOIN departments USING (department_id);
```

### 93. Handling Ambiguous Column Names

```sql
-- 다른 공통 칼럼 중 하나만 보이게 하고 싶을 때 table aliases를 정해주지 않으면 ambiguously defined 에러 발생 / 또는 alias를 지정한 후 테이블명을 다 적으면 에러가 발생한다.
SELECT first_name, last_name, department_name, e.manager_id FROM employees e JOIN departments d USING (department_id);
```

- error 방지를 위해 alias name을 쓰는 걸 추천
- NATURAL join에서 쓰이거나 USING 절에 쓰이는 join의 columns에는 alias를 줄 수 없다.

### 94. Inner Join & Join with the ON Clause

- 두 테이블에서 join 조건을 만족하거나 ON/USING 절 표현에 대해서 모든 행을 반환한다.
- 매치되지 않은 결과는 보이지 않는다.
- USING: 같은 이름 가진 칼럼
- ON: 한 개 이상의 조인 조건을 쓸 수 있다. 다른 이름 가진 칼럼도 가능하다.

```sql
SELECT columns FROM table1 [INNER] JOIN table2
ON (join_condition) / USING(column_name); --  둘 중 하나만 사용하기

SELECT e.first_name, e.last_name, d.manager_id, d.department_name
FROM employees e JOIN departments d
ON (e.department_id = d.department_id AND e.manager_id = d.manager_id);
```

### 95. Multiple Join Operations

- 두 개 이상의 테이블을 조인할 때 USING, ON, NATURAL JOIN을 사용한다.

```sql
-- 먼저 e, d가 join한다. 그리고 l이 그 결과값과 조인한다. 마지막으로 c가 이전 결과값과 join한다. join 순서가 결과에 영향을 미친다.
-- employee는 107행이지만 department와 join하면 106행이 나온다. 그리고 NATURAL JOIN으로 countries를 쓰면 2650개가 나온다.
-- country_id를 SELECT에 추가할 경우 106행이 나온다. 왜냐면 SELECT에서 선택된 열들만 저장되어 사용한다.
-- natural join은 때로 문제가 될 수 있는데 서버도 어떤 열이 조인될지 사전에 모르고 있기 때문이다. 사용을 권장하지 않는다.
SELECT first_name, last_name, d.department_name, city, postal_code, street_address
FROM employees e JOIN department d
ON (e.department_id = d.department_id)
JOIN locations l
-- ON (l.location_id = d.location_id);
USING(location_id)
NATURAL JOIN countries;
```

### 96. Restricting Joins

- WHERE 절 또는 AND 연산자 사용

```sql
SELECT e.first_name, e.last_name, d.department_id, d.department_name, l.city FROM employees e
JOIN departments d
ON (e.department_id = d.department_id)
JOIN location l
ON (d.location_id = l.location_id)
WHERE d.department_id = 100;
-- AND d.department_id = 100; 바로 위 WHERE절 대신 써도 같은 결과가 나온다.
```

### 97. Self Join

- 스스로와 조인하는 것. 같은 테이블의 행을 비교하거나 계층적인 데이터에 쿼리문을 사용할 때 사용된다.

```sql
-- 예를 들어 회사는 계층이 있고 모든 직원은 id가 있고 사원급은 매니저를 가진다(자신의 매니저를 매니저 id로 기록). 매니저 급은 직원 id는 있지만 매니저 id는 없다.
-- 직원과 직원의 매니저를 한 테이블에 볼 수 있다.
SELECT worker.first_name, worker.last_name, worker.employee_id, worker.manager_id, manager.employee_id, manager.first_name, manager.last_name, worker.salary, manager.salary
FROM employees worker JOIN employees manager
ON (worker.manager_id = manager.employee_id);
```

### 98. Non-Equijoins (Joining Unequal Tables)

- Joining Unequal Tables = Non-Equijoins
- Equijoin은 같은 칼럼을 비교해서 조인
- Non-Equi는 BETWEEN이나 =>, <, <>, <=, >=를 사용한다.
- 중복 확인에도 사용 가능

```sql
SELECT e.employee_id, e.first_name, e.last_name, e.job_id, e.salary, j.min_salary, j.max_salary, j.job_id
FROM employees e
JOIN jobs j
ON e.salary > j.max_salary
AND j.job_id = 'SA_REP';

SELECT e1.employee_id, e1.first_name, e1.last_name
FROM employees e1 JOIN employees e2
ON e1.employee_id <> e2.emloyee_id
AND e1.first_name = e2.first_name;
```

### 99. Outer Joins

```sql
-- INNER JOIN
SELECT first_name, last_name, department_name
FROM employees JOIN departments
USING (department_id);


SELECT d.department_id, d.department_name, e.first_name, e.last_name
FROM departments d JOIN employees e
ON (d.manager_id = e.employee_id);
```

- INNER JOIN은 이 경우 원래 employees 테이블의 행 수보다 하나 적게 나온다. department_id가 null인 행이 unmatch라서 사라진 것이다.

- Outer Joins: Left, Right, Full

### 100. Left (Outer) Joins

- 모든 데이터를 왼쪽 테이블을 기준으로 검색한다. 왼쪽 테이블에 있지만 오른쪽 테이블에 없어서 unmatched한 행도 가져온다. 매칭되지 않은 오른쪽 테이블의 컬럼 값은 NULL로 맨 마지막에 나온다.

```sql
-- inner join
SELECT frist_name, last_name, department_id, department_name
FROM employees JOIN departments
USING(departemnt_id);

-- left outer join (department_id가 없는 employees의 행도 나온다.)
SELECT frist_name, last_name, department_id, department_name
FROM employees LEFT OUTER JOIN departments
USING(departemnt_id);

-- 위와 같은 쿼리를 ON을 사용해서 쓰기. OUTER는 선택사항
SELECT e.frist_name, e.last_name, d.department_id, d.department_name
FROM employees e LEFT JOIN departments d
ON (e.department_id = d.department_id)
```

### 101. Right (Outer) Joins

- left joins에서 right로 바뀌었다.

```sql
SELECT first_name, last_name, department_name
FROM employees e RIGHT OUTER JOIN departments d
ON(e.department_id = d.department_id);
```

### 102. Full (Outer) Join

- left, right 테이블에서 모든 행을 검색한다. 

```sql
SELECT first_name, last_name, department_name
FROM employees e FULL OUTER JOIN departments d
ON(e.department_id = d.department_id);
```

- matched한 행과 left, right의 모든 unmatched 행도 보여진다.
- LEFT JOIN, RIGHT JOIN을 합치면 FULL JOIN이 된다.

### 103. Cross Join (Cartesian Product / Cross Product)

- Cross Join: 두 테이블 행 간 모든 조합을 반환하기 위해 사용한다. 테이블간 join을 특정하지 않거나 실수로 일어난다.

```sql
SELECT e.first_name, e.last_name, d.department_name, j.job_title
FROM employees e CROSS JOIN department d
CROSS JOIN jobs j;

-- 직원 수를 세고싶을 때 아래처럼 COUNT(*) 사용하면 오른쪽 테이블에 오는 컬럼값이 비어있어도 하나로 세기 때문에 *가 아닌 employee_id처럼 특정해서 써야 한다.
SELECT c.department_name, c.job_title, COUNT(*) AS employee_count
FROM 
    (SELECT d.department_name, j.job_title, j.job_id, d.department_id
            FROM departments s CROSS JOIN jobs j) c -- ()를 써서 테이블 새로 만들기
LEFT OUTER JOIN employees e
    on (e.job_id = c.job_id AND e.department_id = c.department_id)
GROUP BY c.department_name, c.job_title
ORDER BY c.department_name, c.job_title;
```

### 104. Oracle's Old Style Join Syntax (ANSI vs Non-ANSI Joins) (Part 1)

- ANSI Syntax를 사용하는 걸 추천

- Oracle's Old Join Syntax에선 WHERE이 1) 테이블을 조인하는 역할 2) 행을 필터링하는 역할 두 가지의 역할을 가진다.

```sql
-- Inner Join
-- ANSI
SELECT e.first_name, e.last_name, d.department_name
FROM employees e JOIN departments d
ON (e.department_id = d.department_id);

-- Non-ANSI
SELECT e.first_name, e.last_name, d.department_name, l.city, l.street_address
FROM employees e, departments d, locations l
WHERE e.department_id = d.department_id
AND d.location_id = l.location_id
AND d.department_name = 'Finance';


```

```sql
-- Outer Join
-- ANSI
SELECT e.first_name, e.last_name, d.department_name
FROM employees e LEFT OUTER JOIN departments d
ON (e.department_id = d.department_id);

-- Non-ANSI
SELECT e.first_name, e.last_name, d.department_name
FROM employees e, departments d
WHERE e.department_id = d.department_id(+);
-- + 사인을 더 많은 행(unmatched rows)을 보여주고 싶은 쪽의 반대에 쓴다.
```

```sql
-- Full Outer Join
-- ANSI
SELECT e.first_name, e.last_name, d.department_name
FROM employees e FULL OUTER JOIN departments d
ON (e.department_id = d.department_id);

-- Non-ANSI
SELECT e.first_name, e.last_name, d.department_name
FROM employees e, departments d
WHERE e.department_id(+) = d.department_id
UNION
SELECT e.first_name, e.last_name, d.department_name
FROM employees e, departments d
WHERE e.department_id = d.department_id(+);
```

### 105. Part2

- 다수의 조인을 다룰 때 예전 문법은 오류 가능성이 높다.

```sql
-- ANSI (차례로 실행)
SELECT first_name, last_name, department_name, 
e.department_id, d.department_id, l.location_id
FROM employees e RIGHT OUTER JOIN departments d
ON (e.department_id = d.department_id)
RIGHT OUTER JOIN locations
USING(location_id);

-- Non-ANSI (위와 같은 결과)
SELECT first_name, last_name, department_name, 
e.department_id, d.department_id, l.location_id
FROM employees e, departments d, locations l
WHERE d.location_id(+) = l.location_id
```

```sql
-- ANSI (차례로 실행)
SELECT first_name, last_name, department_name, job_title
FROM employees e RIGHT JOIN departments d
ON (e.department_id = d.department_id)
RIGHT JOIN jobs j
USING(job_id);

-- Non-ANSI (위와 같은 결과)
SELECT first_name, last_name, department_name, job_title
FROM employees e, departments d, ㅓ
WHERE d.location_id(+) = l.location_id
```

### 108. Entity-Relationship Models in DBMS - How to Use Them with Joins?

- One to Many, Many to One

- One to One

- Many to Many : linking, jction, joining, bridging table 등의 이름을 갖는 중간 중개 테이블이 생긴다.
