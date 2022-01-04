### 44. BETWEEN... AND Operator

- BETWEEN AND에서 위 아래 제한선으로 들어가는 값도 포함된다.
- number, date, character 등 많은 타입에서 사용할 수 있다.



### 45. IN Operator

- number, date, character 등 많은 타입에서 사용할 수 있다.
- 보통 SELECT 결과로 받은 resultSet은 데이터가 삽입된 순서대로 보여주지만 IN 연산자를 사용했을 경우 IN절에 index를 가지고 있으면 IN 안의 나열된 특정 값 순서대로(IN 안에 나열된 순서가 아니라 작은 값부터 큰 값 순서로) 결과 데이터가 보인다.  
- 날짜 형식에서 사용할 때 text형식으로 적혀있어도 Oracle이 date로 변경해서 비교한다.



### 46. LIKE Operator

- string에서 일부 text만 가지고 찾고 싶을 때 특정 글자 패턴만 비교한다.
- Oracle Wildcard operators
  -  %: 0 또는 그 이상을 포함해 숫자에 관계없는 걸 의미
  - _: 정확히 한 글자를 의미
- 텍스트는 대소문자 구분 정확히

```sql
SELECT * FROM employees
WHERE job_id LIKE 'SA%';
```



### 47. IS NULL Operator

- WHERE column_name = NULL로는 수행할 수 없다. IS [NOT] NULL을 사용한다.



### 48. Logical Operators (AND, OR, NOT)

- WHERE을 여러 개 사용해 검색 조건을 좀 더 제한할 수 있게 한다.

- AND: NULL & TRUE -> NULL / NULL & FALSE -> FALSE / NULL & NULL -> NULL
- OR: NULL& TRUE -> TRUE / NULL & FALSE -> NULL / NULL & NULL -> NULL
- NOT: NULL -> NULL



### 49. Rules of Precedence

- WHERE절 순서:
  - Arithmetic Operators
  - Concatenation Operator
  - Comparison Conditions
  - IS [NOT] NULL, LIKE, [NOT] IN
  - [NOT] BETWEEN
  - Not Equal To
  - NOT Logical Operator
  - AND Logical Operator
  - OR Logical Operator
- ()를 사용해 순서를 다르게 할 수 있다. 순서를 바꾸지 않더라도 가독성을 위해 ()를 사용하면 좋다.

```sql
-- job_id = 'ST_CLERK' and salary > 5000 인 경우 or job_id = 'IT_PROG'인 경우
SELECT fisrt_name, last_name, job_id, salary FROM employees
WHERE job_id = 'IT_PROG' or job_id = 'ST_CLERK' and salary > 5000;

-- job_id = 'IT_PROG'인 경우 또는 job_id = 'ST_CLERK'인 경우를 만족하면서 salary > 5000인 경우
SELECT fisrt_name, last_name, job_id, salary FROM employees
WHERE (job_id = 'IT_PROG' or job_id = 'ST_CLERK') and salary > 5000;
```

