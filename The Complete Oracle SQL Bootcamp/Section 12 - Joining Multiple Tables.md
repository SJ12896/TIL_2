## Section 12 - Joining Multiple Tables

### 85. What is a Join? & Oracle SQL Join Types

- 두 개 이상의 테이블을 한 쿼리에서 검색할 수 있게 해주는 것. 실제 테이블에 영향을 미치지 않는다.
- Normalization(정규화) :  key를 사용하고 특정 데이터를 테이블에 저장해 데이터 중복을 줄이는 것
- Oracle Join Types: Natural Join, Inner Join, Outer Join(LEFT OUTER, RIGHT OUTER, FULL OUTER), Equijoin, Non-Equijoin, Self Join, Cross Join(Cartesian product)

### 86. Creating a Join

- 양 테이블에 적어도 하나의 같은 칼럼이 있어야 한다. (same name, same data type) / 같은 값으로 연결한다.

```sql
-- 여러 JOIN_TYPE 중 하나를 사용. general syntax
SELECT columns FROM table1 JOIN_TYPE table2 ON table.column_name = table2.coulmn_name;
```



### 87. Natural Join

- Source table은 FROM에 쓰는 table, Target table은 JOIN 뒤에 사용하는 테이블 / WHERE 문으로 데이터 제한
- DESC 테이블명으로 테이블 내의 column 정보 확인
- NATUAL JOIN: ON 뒤의 join column을 따로 지정하지 않고 자동으로 매치한다. 공통 칼럼 / source table / target table순으로 보인다.
- 만약 함께 있는 칼럼에서 null 값이 있다면 JOIN의 결과에서 보이는 테이블에는 해당 행이 없다.



### 88. Join with the USING Clause

- 같은 이름의 칼럼이 여러개라면 USING을 사용해 특정할 수 있다. 이런 join을 Equijoin이라고 한다.

```sql
-- 이 때 같은 이름인 칼럼이 또 있다면 알아서 각각 나온다.
SELECT first_name, last_name, department_name, department_id FROM employees JOIN departments USING (department_id);
```



### 89. Handling Ambiguous Column Names

```sql
-- 다른 공통 칼럼 중 하나만 보이게 하고 싶을 때 table aliases를 정해주지 않으면 ambiguously defined 에러 발생 / 또는 alias를 지정한 후 테이블명을 다 적으면 에러가 발생한다.
SELECT first_name, last_name, department_name, e.manager_id FROM employees e JOIN departments d USING (department_id);
```

- error 방지를 위해 alias name을 쓰는 걸 추천
- NATURAL join에서 쓰이거나 USING 절에 쓰이는 join의 columns에는 alias를 줄 수 없다.



### 90. Inner Join & Join with the ON Clause

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

