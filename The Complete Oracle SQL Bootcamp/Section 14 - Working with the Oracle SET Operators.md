## Section 14 - Working with the Oracle SET Operators

### 119. Introduction to SET Operators in Oracle SQL

- 두 개 이상의 쿼리의 결과를 합치고 하나의 결과로 반환한다.

```sql
-- 책, 영화 테이블 존재. 해리포터, 대부는 두 테이블에 모두 존재

-- 중복으로 존재하는 해리포터, 대부는 한번만 나온다. UNION ALL은 두 번 반환
-- INTERSECT인 경우 해리포터, 대부만 반환. 
SELECT * FROM books
UNION
SELECT * FROM movies;
```

- 4 SET operators in Oracle SQL: UNION, UNION ALL, INTERSECT, MINUS
  
  - UNION: 두 쿼리의 중복된 행을 제거한 후 모든 행을 반환한다.
  
  - UNION ALL: 두 쿼리에서 중복된 행까지 포함해 모든 행을 반환한다.
  
  - INTERSECT: 두 쿼리의 공통으로 존재하는 행만 반환한다.
  
  - MINUS: 첫 테이블에서 공통으로 존재하는 행을 제외하고 반환

- SET 연산자를 사용한 쿼리를 Compound Queries라고 한다.

- 모든 SET 연산자는 동일한 우선순위를 가진다.

- SET 연산자는 위에서 아래로 실행된다. 괄호를 사용해서 순서를 변경할 수 있다.

- 표현식에서 SELECT 뒤의 컬럼 수는 위와 아래 쿼리에서 반드시 일치해야 한다.

- 컬럼들의 데이터 타입도 반드시 일치해야한다.

- ORDER BY는 반드시 맨 마지막에 한 번만 사용해야 한다.

- 결과는 기본적으로 오름차순이다.

- 중복되는 행은 UNION ALL을 제외하고는 자동으로 제거되어 나온다.

- 컬럼 헤딩은 첫번째 쿼리를 기준으로 나온다.

- Join은 컬럼을 합치는 것이지만 SET은 행을 합친다. JOIN은 두 테이블이 양옆으로 붙지만 SET은 위아래로

### 120. UNION and UNION ALL Operators

```sql
SELECT * FROM retired_employees
UNION
SELECT * FROM employees
```

- 위 아래 쿼리의 department_id, salary  컬럼이 같은 위치여면 논리적으로 오류가 있는 것이지만 데이터 타입이 같기 때문에 잘 출력된다.

### 121. INTERSECT Operator

### 122. MINUS Operator
