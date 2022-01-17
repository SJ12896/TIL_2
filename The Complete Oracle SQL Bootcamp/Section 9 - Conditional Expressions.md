## Section 9 - Conditional Expressions

### 75. Oracle Conditional Expressions: CASE Expressions

- IF-THEN-ELSE 체크를 SQL에서 실행한다.
- 3개 이상의 파라미터가 필수로 필요하다.
- ELSE문이 undefined인 경우 NULL 값이 리턴된다.

```sql
-- 예시 expression은 모든 조건들과 비교할 값을 의미한다.
-- expression, comparison_expression_n은 같은 데이터 타입이어야 한다.
-- result_n도 전부 같은 데이터 타입이어야 한다.
CASE expression WHEN comparison_expression_1 THEN result_1
				WHEN comparison_expression_2 THEN result_2
				...
				WHEN comparison_expression_n THEN result_n
				ELSE result
END
```

- Simple CASE Expression: 표현식이 처음에 위치한다. 가능한 결과는 조건 값에 따라 체크된다.

```sql
CASE first_name
		WHEN 'Alex' THEN 'Name is Alex'
		WHEN 'John' THEN 'Name is John'
	ELSE 'Another'
END
```

- Searched CASE Expression: CASE 표현식의 시작에 언급하지 않고 사용된다. 이 경우 조건식마다 다른 조건을 사용해 비교할 수 있다.

```sql
CASE 
		WHEN first_name = 'Alex' THEN 'Name is Alex'
		WHEN first_name = 'John' THEN 'Name is John'
	ELSE 'Another'
END
```

- SELECT에서 사용하기

```sql
SELECT first_name, last_name, job_id, salary, hire_date,
	CASE job_id WHEN 'ST_MAN' THEN 1.20 * salary
				WHEN 'SH_MAN' THEN 1.30 * salary
				WHEN 'SA_MAN' THEN 1.40 * salary
				ELSE salary END "UPDATED_SALARY"
FROM EMPLOYEES WHERE job_id
IN ('ST_MAN', 'SH_MAN', 'SA_MAN');
```

- WHERE에서 사용하기

```sql
SELECT first_name, last_name, job_id, salary
FROM EMPLOYEES 
WHERE
	(CASE WHEN 'IT_PROG' AND salary > 5000 THEN 1
		  WHEN 'SA_MAN' AND salary > 10000 THEN 1
	 ELSE 0
     END) = 1;
```



### 76. Oracle Conditional Expressions: DECODE Expressions

- Oracle에서 사용하는 CASE의 대체 표현식으로 사용하기 쉽다.
- if-then-else 로직을 구현한다.
- CASE는 표현식이지만 DECODE는 함수다.
- DECODE (col|expression, search1, result1, [, search2, result2], ... [, default])
- search들은 같은 데이터 타입이어야하고 result들 역시 같은 데이터 타입이어야 한다. 둘은 다른 타입이어도 된다.

```sql
SELECT first_name, last_name, job_id, salary, hire_date,
	DECODE(job_id, 'ST_MAN', salary * 1.20,
          		   'SH_MAN', salary * 1.30,
          		   'SA_MAN', salary * 1.40) UPDATED_SALARY
FROM EMPLOYEES WHERE job_id
IN ('ST_MAN', 'SH_MAN', 'SA_MAN');
```

