## The Complete Oracle SQL Bootcamp

### Section 1 - Database Concepts

### 1. What is a Database?

- Data: 나중에 사용하기 위해 어딘가에 저장된 정보. 수첩, excel파일, 텍스트 파일 등에 저장되어 있음.모든 행동에서 생성된다. 애플리케이션을 사용하거나 쇼핑하거나 하는 모든 행동이 저장된다. 처음에 종이에 저장되었다가 전자기기의 텍스트 파일이나 폴더 안으로 옮겨갔다. 하지만 더 구조적이고 조직적으로 저장될 필요성을 느껴 excel 스프레드 시트 같은 걸 사용해 쉽게 검색할 수 있게 됐다. 함수나 매크로를 사용해 복잡한 일을 할 수 있다. 하지만 데이터가 급격하게 증가하면서 빅데이터에 적합하지 않았다. 그래서 데이터베이스에 저장하게 됐다. 
- Database: 기본적으로 데이터의 조직적인 컬렉션으로 접근하고 조작, 검색이 쉽게 되어있다. 
- DBMS(Database Management System) : 사용자가 데이터베이스에 접근하고 조작, 검색할 수 있게 만들어주는 프로그램들의 집합.
  - hierarchical dbms: 부모 자식 관계로 저장한다. 노드와 브랜치라는 트리 형태로 저장된다. 요즘에 드물다.
  - network dbms : 다대다 관계
  - relational dbms: 요즘 가장 많이 사용된다. oracle, mysql, mssql등. 다대다관계를 지원하지 않는다.
  - object-relational dbms: own type을 가진다. postgre sql이 있다.

### 2. Why Oracle Database?

- security, performance(optimizer로 튜닝 옵션을 사용), scalability(많은 양 가능), powerful coding(oracle sql and pl/sql), support(oracle 직원의 지원)

### 3. What is a Table?

- 모든 데이터는 데이터베이스상의 테이블에 저장된다.
- Table 상의 한 행: Record(Row)
- Table 상의 한 열: Columns(특정한 한 종류의 데이터들)

### 6. What is a Relational Database(RDBMS)?

- 