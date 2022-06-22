# 자바 ORM 표준 JPA 프로그래밍 - 기본편

- 튜토리얼과 다르게 실무에선 도입하면 수십개 이상의 객체와 테이블이 존재해 설계, 매핑하기 어렵다. 
- 목표: 객체와 테이블 제대로 설계하고 매핑하기. 기본키와 외래키 매핑 / 일대일부터 다대다 매핑까지 / 성능까지 고려 / 복잡한 시스템도 jpa로 설계 가능 / jpa 내부 동작 방식 이해 - jpa가 어떤 sql을 만들어내고 언제 실행하는지 이해 필요



## JPA 소개

### SQL 중심적인 개발의 문제점

- 객체를 관계형 DB에 관리해야하는 시대. sql 중심적인 개발이 되며 지루한 코드를 반복하게 된다. 거의 CRUD니까
- 패러다임의 불일치. 객체 지향(추상화, 캡슐화, 정보은닉, 상속, 다형성)과 관계형 데이터베이스의 차이. 객체를 SQL에 저장하기 위해 개발자가 SQL 매퍼로 일하게 된다.
  - 상속: 객체에 존재. 관계형 DB에는 유사하지만 같은건 없다.
  - 연관관계: 객체는 참조로 가져오지만 DB는 PK, FK로 조인해서 가져온다. 그런데 객체는 부모에서 자식으로 갈 수 있지만 반대는 참조가 없어 불가능하다. db에는 fk를 반대로 조인할 수 있어 양방향으로 가능하다. 그래서 보통 테이블에 맞춰 객체를 모델링한다. 하지만 이건 객체지향같지 않아 객체 자체를 부모 객체에 넣으면 insert into할 때 넣을 외래키 값이 없어져 넣은 객체를 가져오고 거기서 다시 id값을 가져온다. 하지만 또 이러면 조회할 때 join이 필요하고 각각의 객체를 가져오고 set으로 관계설정을 직접 해줘야해서 복잡해진다.
  - 데이터 타입, 데이터 식별 방법
  - 조회할 때 JOIN 쿼리를 가져와서 각각에 맞는 객체를 생성하는 등의 단계를 거쳐야한다. 하지만 자바 컬렉션에서 조회하면 get으로 바로 조회하고 부모 타입으로 조회해 다형성 활용도 가능해 단순하다.

- 객체는 서로 참조가 있다는 가정하에 자유롭게 객체 그래프를 탐색할 수 있어야 한다. 하지만 처음 실행하는 sql에 따라 탐색 범위가 결정되어 가져오지 않은 테이블은 찾아볼 수 없다. 
  - 엔티티 신뢰 문제 발생. 어떻게 조회해서 가져오는지 보지않은 이상 신뢰하기 힘들다. 
  - 하지만 모든 객체를 미리 로딩해 둘 수는 없다.
  - 상황에 따라 회원 조회 메서드를 여러개 만들어두게 되면 계층형 아키텍처에서 진정한 의미의 계층 분할이 어렵다.
- 객체를 자바 컬렉션에 저장하듯 db에 저장하기 위해 JPA가 탄생



### JPA 소개

- Java Persistance API: 자바 진영의 ORM 표준
- ORM: Object-relational mapping / 객체는 객체대로 설계하고 관계형 db는 관계형 db대로 설계하면 orm 프레임워크가 중간에서 매핑
- JPA는 애플리케이션과 JDBC사이에서 동작한다. 삽입의 경우 JPA에게 객체를 넘기면 객체를 분석해 SQL을 생성하고 JDBC API를 사용해 DB에 SQL을 보낸다. 또한 조회의 경우 JDBC API에 결과가 반환되면 JPA가 ResultSet을 매핑
- EJB(엔티티 빈, 자바 표준)이란게 있었으나 모든 면에서 그닥이라 하이버네이트라는 오픈소스 ORM 프레임워크가 등장한다. 그 이후 하이버네이트를 만들던 사람이 똑같이 그 기반으로 JPA를 만든다.(거의 비슷)
- JPA 표준 명세: JPA는 인터페이스의 모음. JPA 2.1 표준 명세를 구현한 3가지 구현체 존재. 하이버네이트, Eclipse Link, DataNucleus
- 생산성
  - 저장: jpa.persist
  - 조회: jpa.find
  - 수정: member.setName -> 이것만 해도 db에 업데이트 알아서 됨.
  - 삭제: jpa.remove
- 기존엔 유지보수를 할 때 한 필드가 변경되면 모든 관련 sql을 수정해야 했다. jpa를 사용하면 필드를 추가해도 sql은 알아서 jpa가 처리한다.
- 패러다임의 불일치 해결
  - 상속: insert를 하위 테이블에 insert하면 알아서 pk, fk로 연결된 테이블에도 insert된다.
- JPA의 성능 최적화 기능
  - 1차 캐시와 동일성 보장: 동일한 트랜잭션에서 조회한 엔티티는 같으므로 약간의 조회 성능 향상 -> 근데 짧은 시간 캐싱이라 사실상 그렇게 큰 이득은 없다고 한다?
  - 트랜잭션을 지원하는 쓰기 지연: 트랜잭션을 커밋(transaction.commit())할 때까지 INSERT를 모음. JDBC BATCH SQL 기능을 사용해 한 번에 전송
  - 지연 로딩: 객체가 실제 사용될 때 로딩. 사용하는 객체만 조회하지만 그 안의 객체를 조회하는 순간 알아서 그 테이블까지 조회해준다. 즉시 로딩은 join sql로 연관 객체까지 미리 조회(옵션 설정하면 됨).



## JPA 시작하기

### Hello JPA - 프로젝트 생성

pom.xml

- spring reference에서 내가 사용하는 스프링 부트 버전에 맞는 dependency 버전 보고 입력 필요
- hibernate-entitymanager를 받으면 core, javax.persistence-api(jpa인터페이스의 구현체로 하이버네이트를 선택했는데 하이버네이트가 jpa인터페이스를 가지고 여기에 있음)
- h2는 다운로드 받은 버전과 맞추기

```xml
    <dependencies>
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>5.6.8.Final</version>
        </dependency>

        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>2.1.212</version>
        </dependency>
    </dependencies>
```

- JPA 설정파일 persistence.xml

- 데이터베이스 방언(사투리)
  - jpa는 특정 db 종속x
  - 각 db sql 문법, 함수 조금 다름. sql표준을 지키지 않는 특정 db의 고유 기능.

resources/META-INF/persistence.xml

- dialect를 통해 mysql, h2, oracle등을 지정할 수 있다.
- javax로 시작하는건 hibernate말고 다른 구현 라이브러리에서도 사용(표준)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
    <persistence-unit name="hello">
        <properties>
            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>

            <!-- 옵션. 실행한 SQL보여주기, 포맷팅, 코멘트 추가 -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>
            <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
        </properties>
    </persistence-unit>
</persistence>
```



### Hello JPA - 애플리케이션 개발

- h2데이터베이스 실행 안됨 -> 서버 완전히 껐다가 다시 켜서 하기
- jdbc:h2:~/test: 데이터베이스 파일 생성
  jdbc:h2:tcp://localhost/~/test: 이후 연결



Member.java

- JPA가 처음 로딩될 때 객체를 인식하기 위해 필수적으로 @Entity를 붙여줘야 한다.
- 만약 DB테이블 명과 객체 명이 다르면 @Table(name = "USER") 처럼 적어줘야하고 column명 역시 @Column(name="username") 이런식으로 적어주면 된다.
- 생성자 만들거면 기본 생성자도 하나 만들어줘야한다.

```java
package hellojpa;

import javax.persistence.Entity;
import javax.persistence.Id;

@Entity
public class Member {

    @Id
    private Long id;
    private String name;
    ...
}
```



JpaMain.java

- EntityManagerFactory는 로딩 시점에 딱 하나만 만들어야 한다. 이름은 persistence.xml에 적은 persistence-unit name과 일치해야 한다.
- EntityManager는 쓰레드가 절대 공유하면 안되고 요청왔을 때 사용하고 버려야한다. em은 내부적으로 db connection을 물고 동작하기 때문에 끝나면 꼭 닫아줘야한다.
- 실제 DB를 변경할 때는 반드시 트랜잭션 안에서 해야하며 entityManager를 사용한다.
- 객체 값을 수정후 persist같은 db저장 메서드를 시행하지 않아도 반영되는데, commit시점에 jpa가 변경된 내용을 살펴 알아서 update쿼리를 시행한다.

```java
package hellojpa;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;
import java.util.List;

public class JpaMain {
    public static void main(String[] args) {

        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");

        EntityManager em = emf.createEntityManager();

        EntityTransaction tx = em.getTransaction();
        tx.begin();

        try {
            
            // insert
//            Member member = new Member();
//            member.setId(1L);
//            member.setName("helloA");
//            em.persist(member);

            // select
            Member findMember = em.find(Member.class, 1L);

            // delete
            // em.remove(findMember);

            // update
            findMember.setName("helloJpa");
            
            System.out.println("findMember = " + findMember.getName());

            // JPQL 대상은 테이블이 아니라 멤버 객체다. 실제 실행 쿼리를 보면 select로 필드 전부 나열했지만 여기선 member entity를 선택한것.
            List<Member> result = em.createQuery("select m from Member as m", Member.class).getResultList();

            // .setFirstResult(5).setMaxResult(8).getResultList()처럼 응용해 사용하면 sql별로 문법에 맞게 지정한것만 가져온다.
           for (Member m: result) {
                System.out.println(m.getName());
            }

            tx.commit();
        } catch (Exception e) {
            tx.rollback();
        } finally {
            em.close();
        }
        emf.close();
    }
}
```

- 나이가 18살이상 이상인 회원을 모두 검색하고 싶다면? JPQL사용 필요
- JPQL: JPA에서 SQL을 추상화해 제공하는 객체 지향 쿼리 언어로 가장 단순한 조회 방법
  - JPA를 사용하면 엔티티 객체 중심으로 개발하는데 검색할 때도 객체를 대상으로 한다. 하지만 모든 DB 데이터를 객체로 변환해 검색하는건 불가능해 필요한 데이터만 가져오기위해 조건이 포함된 SQL 필요
  - SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 등 가능



## 영속성 관리 - 내부 동작 방식

### 영속성 컨텍스트 1

- JPA에서 가장 중요한 것 2가지: 객체와 관계형 데이터베이스 매핑 / **영속성 컨텍스트**
- 엔티티 매니저 팩토리와 엔티티 매니저: 웹 애플리케이션 안에서 고객 요청이 오면 EMF가 EM을 생성한다. EM은 내부적으로 DB 커넥션을 통해 DB를 사용한다.
- 영속성 컨텍스트: JPA를 이해하는데 가장 중요한 용어로 `엔티티를 영구 저장하는 환경` / EntityManager.persist(entity)는 db에 저장한다는게 아니라 엔티티를 영속성 컨텍스트라는 곳에 저장하는 것이다.
  - 영속성 컨텍스트는 논리적인 개념으로 em을 통해 접근한다.
  - em을 생성하면 1:1로 영속성 컨텍스트가 생성된다. 
- 엔티티의 생명주기
  - 비영속: new/transient, 영속성 컨텍스트와 관계 없는 새로운 상태. 그냥 객체 생성함
  - 영속: managed, 영속성 컨텍스트에 관리되는 상태. persist로 객체를 저장하면 영속성 컨텍스트에 들어가서 영속 상태가 되는 것. 아직 db에는 저장되지 않음(commit안했으니까)
  - 준영속: detached, 영속성 컨텍스트에 저장됐다가 분리된 상태. em.detach("entity")를 하면 영속성 컨텍스트에서 분리된 상태
  - 삭제: removed, 삭제된 상태. em.remove("entity")로 객체에서 삭제한 상태

### 영속성 컨텍스트 2

- 영속성 컨텍스트의 이점

  - 1차 캐시: 영속성 컨텍스트는 내부에 1차 캐시를 가지고 있다. persist로 저장하면 내부에 1차 캐시가 있는데 db상 pk가 id가 되고 값은 entity 좀전에 저장한 객체 자체다.  이러면 조회할 때 영속성 컨텍스트 안에서 1차 캐시를 뒤져서 바로 가져온다. 1차 캐시에 없으면 db를 조회해 가져오고 1차 캐시에 저장한 후 반환한다. 근데 사실 그렇게 큰 도움이 안된다. em은 트랙잭션 단위라 비지니스가 끝나면 같이 지워져서 짧은 순간에만 이득이 있다. 전체적으로 공유하는 2차 캐시가 따로 있다. 
  - 동일성(identity) 보장: 영속 엔티티의 동일성 보장. 1차 캐시로 반복 가능한 읽기(REPETABLE READ) 등급 트랜잭션 격리 수준을 db가 아닌 애플리케이션 차원에서 제공 -> 자바 컬렉션에서 객체를 가져오면 주소가 같은 것처럼 여기서도같은 객체로 인식된다.
  - 트랙잭션을 지원하는 쓰기 지연: 엔티티 매니저는 데이터 변경 시 트랜잭션을 시작해야 한지만 sql을 db에 보내는 것은 commit() 순간이다. 영속 컨텍스트는 1차 캐시뿐 아니라 쓰기 지연 저장소도 있는데 persist로 저장하면 1차 캐시에 저장하고 insert sql은 sql저장소에 두었다가 commit 되면 flush가 되면 db에 가서 실제로 저장된다. 순간순간 db에 요청하면 최적화하기 힘들다. persistence.xml에 batch-size를 지정해 한 번에 요청할 갯수를 정할 수 있다. 
  - 변경 감지: update문 자동으로 요청보내는 거. 커밋 시점에 flush()가 호출되고 1차 캐시안의 엔티티와 스냅샷(값을 읽어온 시점의 상태를 저장해둔 것)을 비교한다. 변경됐으면 update sql을 생성하고 DB에 반영하고 commit한다. / dirty checking
  - 지연 로딩

  

### 플러시

- 영속성 컨텍스트 변경 내용을 데이터베이스에 반영. 
- 플러시 발생하면 변경 감지, 수정된 엔티티 쓰기 지연 sql저장소에 등록, 쿼리를 데이터베이스에 전송(등록, 수정, 삭제)
- em.flush()로 직접 호출 / 트랜잭션 커밋시 자동 호출 / JPQL쿼리 실행시 자동 호출
- `flush를 해도 1차 캐시는 유지된다.` 오직 영속성 컨텍스트의 쓰기 지연 sql 저장소에 쌓인 쿼리들이 db에 반영되는 것 뿐이다. 
- 만약 persist를 한 후 JPQL을 실행해 데이터베이스에 조회하면 없기 때문에 자동으로 플러시가 실행한다.
- 플러시 모드 옵션으로 커밋이나 쿼리 실행 시 플러시를 할지(FlushModeType.AUTO / 그냥 손대지말고 이걸로 쓰는게 낫다.) 커밋할 때만 할지(FlushModeType.COMMIT)를 정할 수 있다.



### 준영속 상태

- 영속 상태 엔티티가 영속성 컨텍스트에서 분리(detached)되면 영속성 컨테이너가 제공하는 기능을 사용할 수 없다. 
- em.detach(entity)
- em.clear(): 1차 캐시에서 초기화해서 쿼리문을 보고싶을 때 flush후 clear한다.
- em.close()



## 엔티티 매핑

- 객체와 테이블 매핑: @Entity, @Table
- 필드와 컬럼 매핑: @Column
- 기본 키 매핑: @Id
- 연관관계 매핑: @ManyToOne, @JoinColumn

### 객체와 테이블 매핑

- @Entity: 이게 붙은 클래스는 JPA가 관리하며 `엔티티`라고 한다. `기본 생성자 필수`(파라미터 없는 public, protcted) / final, enum, interface, inner 클래스 사용X / 저장할 필드에 final 사용X
  - name: 속성, JPA에서 사용할 엔티티 이름 지정. 가급적 클래스 이름과 동일한 기본 값 사용 / 다른 패키지에 같은 이름으로 JPA에서 사용할 클래스가 있다면 사용
- @Table: 엔티티와 매핑할 테이블 지정. 속성으로 name, catalog, schema, uniqueConstraints등

### 데이터베이스 스키마 자동 생성

- JPA가 매핑 정보만 보면 어떤 테이블을 만들어야되는지 알 수 있으니까 애플리케이션 로딩 시점에 DB 테이블 생성기능을 제공한다. 실행 시점에 DDL 자동 생성. 
- persistence.xml에서 hibernate.hbm2ddl.auto value="create"로 지정한 후 실행하면 drop후 create를 해준다. 애플리케이션 로딩 시점에 @Entity매핑된 애들한테 다 해준다. 그런데 drop하기 싫고 column을 추가로 alter하고 싶으면 update모드로 하기. 그런데 지우는건 안됨. 컬럼 추가만 된다. validate는 엔티티와 테이블이 정상 매핑되었는지 확인해주는데 
- **운영에는 절대 create, create-drop, update 사용하지 말기.** 개발초기 단계는 create, update. 테스트 서버는 update, validate. 스테이징, 운영 서버는 validate, none
- @Column에서 unique, length같은 제약조건을 추가하는건 실행 자체에 영향이 없고 DDL 생성에만 영향이 있다. 

### 필드와 컬럼 매핑

- persistence.xml에서 <property name="hibernate.hbm2ddl.auto" value="create" /> value를 통해 create, update등을 설정

Member.java

- @Column
  - insertable
  - updatable: 등록하고 변경하면 안될 때 false 
  - nullable(DDL): false로 하면 not null 제약조건이 붙는다.
  - unique(DDL): UNIQUE 제약조건 걸기. 잘안씀. 이름이 랜덤으로 이상하게 만들어져서 / @Table에서 uniqueConstraints는 이름을 지정할 수 있어 선호
  - length
  - columnDefinition: 'varchar(100) default 'EMPTY''로 적으면 그대로 이렇게 만들어진다.
  - precision, scale: BigDecimal 타입에서 사용한다.
- @Enumerated: ORDINAL은 순서를 DB에 저장, STRING은 이름을 DB에 저장. **ORDINAL이 기본인데 사용하지 말기** / ENUM클래스에 값을 추가하면 순서가 그대로 반영돼서 이전 데이터와 기준이 달라진다.
- @Temporal: LocalDate, LocalDateTime을 지원하게 돼서 별로 필요없어졌다. 그냥 private LocalDate createDate2; 
- @Lob: 문자면 CLOB매핑, 나머지는 BLOB매핑
- @Transient: 필드매핑하지 않고 DB저장X, 메모리상에서만 임시로 값 보관하고 싶을 때

```java
package hellojpa;
import javax.persistence.*;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.util.Date;
@Entity
public class Member {
    @Id
    private Long id;

    @Column(name = "name")
    private String username;

    private Integer age;

    // Enum타입
    @Enumerated(EnumType.STRING)
    private RoleType roleType;

    // 날짜 타입은 Temporal쓰고 3가지 타입이 있다.
    @Temporal(TemporalType.TIMESTAMP)
    private Date createdDate;

    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate;

    // clob같은 큰 컨텐츠
    @Lob
    private String description;
    
    //Getter, Setter…
}
```

RoleType.java

```java
package hellojpa;

public enum RoleType {
    USER, ADMIN
}
```



### 기본 키 매핑

- @Id: 직접 할당

- @GeneratedValue: 자동 생성. 속성 strategey는 default로 GenarationType.AUTO다.(DB에 맞게 자동으로 설정) 그 외는

  - IDENTITY: DB에 위임, MYSQL(AUTO_INCREMENT) / 트랜잭션 커밋 시점에 INSERT SQL 실행. AUTO_INCREMENT는 INSERT 실행 후 ID 값을 알 수 있는데 영속성 컨텍스트는 PK 값을 알아야 1차 캐시에 저장된다. 그래서 em.persis()시점에 즉시 nsert 실행하고 db에서 식별자를 조회한다.
  - SEQUENCE: DB 시퀀스 오브젝트 사용. 유일한 값을 순서대로 생성하는 데이터베이스 오브젝트, ORACLE / 클래스에 @SequenceGenerator이걸로 sequence를 만들어주지 않으면 h2시퀀스를 사용  / Long타입을 써야 맞다. 요즘엔 성능에 큰 영향이 없다. 10억이 넘었는데 타입을 바꾸는게 더 힘들다.
    - @SequenceGenerator(name="MEMBER_DEQ_GENERATOR", sequenceName="MEMBER_SEQ", initialValue=1, allocationSize=1)
    - allocationSize 기본 값은 50
    - persist할 때 pk가 필요하기 때문에 call next value for MEMBER_SEQ가 실행된다. 
    - 성능상 서버를 왔다갔다하는 걸 고려해 allocationSize를 설정한다. 이걸로 미리 allocationSize숫자만큼 가져와두는 방법이다. 다되면 nextCall을 호출하는 방식이다. 여러 웹서버가 있어도 동시성 이슈없이 해결된다. 처음 호출하면 두 번 호출되어 DB SEQ값이 1, 51이다. 50개씩 써야되는데 1이라 한 번 더 호출한 상황. 애플리케이션은 1, 2 순서로 쓰고 DB는 계속 51. 크게 호출하면 웹 서버 내리는 시점에 구멍이 생긴다. 
  - TABLE: 키 생성 전용 테이블을 만들어 시퀀스 흉내. 모든 db에 적용 가능하지만 성능이 떨어진다. allocationSize, initalValue 속성 있음.

  ```java
  @Entity 
  @TableGenerator( 
   name = "MEMBER_SEQ_GENERATOR", 
   table = "MY_SEQUENCES", 
   pkColumnValue = “MEMBER_SEQ", allocationSize = 1) 
  public class Member { 
       @Id 
       @GeneratedValue(strategy = GenerationType.TABLE, 
       generator = "MEMBER_SEQ_GENERATOR") 
       private Long id; 
  }
  ```

- 권장하는 식별자 전략: null이 아니고 유일하고 변하지 않는 조건을 만족하는 자연키(주민등록번호, 전화번호 같은 것)는 찾기 어렵다. `대리키(대체키, 비지니스와 전혀 상관없음)`를 사용해야 한다. 권장은 **Long형 + 대체키 + 키 생성전략 사용**



### 실전 예제 1 - 요구사항 분석과 기본 매핑

- setter를 만들면 여기저기서 수정해야해서 유지보수에 안좋을 수 있다. 가급적 생성자로 세팅하게 되어있으면 좋다.
- 스프링부트에서 jpa로 하이버네이트 시행하면 orderId를 자동으로 order_id로 만들어주지만 스프링부트가 아니면 그대로 만들어준다. 매핑 정보가 다 보이길 원한다면 name을 따로 지정해주자.
- 주문한 사람을 찾기 위해서 주문 객체를 찾고 -> 주문 객체에서 멤버 아이디를 가져와서 -> 그걸로 다시 멤버를 찾으면 `객체지향` 느낌이 아니다. 대신 order에 Member객체가 있어 바로 꺼낼 수 있어야 한다. ORDER에 memberId가 있는 상황을 관계형 db에 맞춘 설계라고 한다. 테이블 외래키를 그대로 객체에 가져와 객체 그래프 탐색이 불가능하고 참조가 없어 uml도 잘못됨.



## 연관관계 매핑 기초

### 단방향 연관관계

- 객체의 참조와 테이블의 외래키 매핑
- 방향(단, 양) / 다중성(다대일, 일대일 등) / 연관관계의 주인(양방향 연관관계는 관리 주인 필요)
- 객체를 테이블에 맞추어 데이터 중심으로 모델링하면 협력 관계를 만들 수 없다. 테이블은 외래키로 조인해서 연관 테이블을 찾고 객체는 참조를 사용해 연관 객체를 찾는다.



- 객체지향 모델링으로 전환

Member.java

```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;
    private String username;
    
//    @Column(name="TEAM_ID")
//    private Long teamId;

    @ManyToOne
    @JoinColumn(name="TEAM_ID")
    private Team team;
    ...
}
```

- Main클래스에서 Member로 팀을 찾을 때 하이버네이트가 보여주는 쿼리에서는 join을 사용한다. @ManyToOne(fetch=FetchType.LAZY)로 지정하면 쿼리가 분리되어 나간다.



### 양방향 연관관계와 연관관계의 주인1 - 기본

- 테이블 연관관계는 그대로 일대다 관계면서 



- 연관관계 주인과 mappedBy: 객체와 테이블간 연관관계 맺는 차이를 이해해야 한다. 객체는 회원->팀, 팀->회원의 단방향 연관관계가 2개지만 테이블은 회원과 팀 연관관계가 양방향 하나다.
