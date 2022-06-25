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

- 연관관계 주인과 mappedBy: 객체와 테이블간 연관관계 맺는 차이를 이해해야 한다. 객체는 회원->팀, 팀->회원의 단방향 연관관계가 2개지만 테이블은 회원과 팀 연관관계가 양방향 하나다.
- 단방향 2개면 관련 데이터가 바꼈을 때 양쪽에서 다 바껴야 하는데 어떻게할까? db는 상관없이 그냥 테이블 데이터만 잘 바뀌면 된다. 이를 위해 `연관관계의 주인`이 필요하다.
- 연관관계의 주인: 객체 두 개중 하나를 연관관계 주인으로 지정해 외래 키를 관리하게 한다. 주인이 아니면 읽기만 가능하다. 주인은 mappedBy속성을 사용하면 안된다. 주인이 아니면 mappedBy 속성으로 주인을 지정해야 한다.
  - **외래 키가 있는 곳을 주인으로 정해라**. oneToMany에서 Many쪽이 주인.

Member.java

- 주인인 쪽

```java
    @ManyToOne
    @JoinColumn(name="TEAM_ID")
    private Team team;
```

Team.java

- 컬렉션 생성해주는게 관례. nullPoint가 안되도록?

```java
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>(); 
```



### 양방향 연관관계와 연관관계의 주인2 - 주의점, 정리

- 순수하게 객체지향적으로 생각하면 양쪽 다 값을 입력해야하는건 맞다. 주인쪽에서만 데이터를 넣고 반대쪽에서는 지정하지 않으면 생성한 team은 flush()를 하지않으면 1차 캐시에 있는 그대로 가져오기 때문에 members가 존재하지 않는다. 테스트할 때도 jpa없이 순수 자바로 진행하기 때문에 이런 오류가 발생할 수 있다.
- 따라서 `순수 객체 상태를 고려해` 항상 양쪽에 값을 설정하자. 하지만 main에서 넣어주는 대신 Member의 setTeam에서 자신을 넣어주는 코드를 추가해 한쪽만 호출해 양쪽에 값을 넣게 설정할 수 있다. 이런 로직이 추가되면 setTeam이란 이름보다 changeTeam처럼 해주면 좋다. 아니면 Team에서 addMember를 만들어주어도 된다.

```java
// JpaMain.java
            Team team = new Team();
            team.setName("teamA");
        //    team.getMembers().add(member); 연관관계 주인이 아니기 때문에 멤버를 먼저 생성하고 team에 add(member)를해도 db에서 null
            em.persist(team);

            Member member = new Member();
            member.setUsername("member1");
            member.setTeam(team);
            em.persist(member);

            team.addMember(member);

// Team.java
    public void addMember(Member member) {
        member.setTeam(this);
        members.add(member);
    }

// Member.java
    public void setTeam(Team team) {
        this.team = team;
        team.getMembers().add(this);
    }
```



- 양방향 매핑시엔 무한 루프를 조심해야한다. toString, lombok, JSON 생성 라이브러리 / toString쪽은 거의 쓰지말고 JSON 생성은  컨트롤러에는 엔티티 반환하지 말아야한다. 무한루프되거나 엔티티가 변경하면 API도 바뀌어버린다(?) 대신 DTO로 반환하는걸 추천

```java
    // team에서 toString생성할 때 members에서 toString을 호출해 무한루프
    @Override
    public String toString() {
        return "Member{" +
                "id= " + id +
                ", username='" + username + '\'' +
                ", team= " + team.toString() +
                '}';
    }
```



- 단방향 매핑만해도 연관관계 매핑은 완료된 거지만 양방향 매핑으로 반대 방향 조회(객체 그래프 탐색) 기능이 추가된 것이다. JPQL에서 역방향으로 탐색할 일이 많다. 단방향만 잘하고 `양방향은 필요할 때 추가(테이블 영향X)`



### 실전 예제 2 - 연관관계 매핑 시작

- ORDER클래스 같은 경우는 예약어일수도 있어서 ORDERS를 추천

- 단방향만 설계해도 충분하다고 말하면서 Member의 orders는 정말 필요없는 코드라고 했는데 굳이 member -> orders찾기보다 db에서 생각하면 where조건으로 member id를 걸어 생각하는게 낫다고 한다.

```java
@Entity
public class Member {
    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();
}
```



## 다양한 연관관계 매핑

- 다대일 단방향 주로 사용. 일대다 단방향을 실무에선 거의 사용하지 않는다. 일 테이블에서 다테이블 객체를 가져와서 거기에 add를 하기 때문에 결국 다테이블에 추가로 update가 실행된다. 내가 손대지 않은 엔티티와 관련된 쿼리가 실행되기때문에 운영할 때 힘들어진다.
  - 일이 연관관계 주인이지만 테이블에서 다쪽에 외래키가 있어 객체와 테이블 차이로 반대편 테이블 외래키 관리가 특이하다. 
  - 꼭 @JoinColumn을 사용해야한다. 아니면 조인 테이블이 하나 더 생겨버린다.
- 일대다 양방향으로 설정한다면? 다 테이블에서 @JoinColumn(name="sdf", insertable = false, updatable = false) 로 설정해야 일테이블을 연관관계 주인으로 유지할 수 있다. 읽기 전용 필드를 사용해 양방향처럼 사용하는 방법으로 공식적으로 존재하지 않다. 사용하지 않는 편이 좋다.

- 일대일: 주 테이블, 대상 테이블 중 외래 키 선택 가능. 외래 키에 유니크 제약조건 추가 / 다대일 단방향 매핑과 비슷하다. / 대상테이블에 외래 키 단방향 관계는 JPA지원X / 일대일이라 외래키가 어디에 있어도 상관없지만 추후 확장 가능성에 대해 고려해야한다. 성능상 많이 SELECT하는 테이블에 있게하면 좋다.
  - 주 테이블에 외래키: 객체지향 개발자 선호, JPA 매핑 편리. 주 테이블만 조회해도 대상 테이블에 데이터가 있는지 확인 가능하지만 값이 없으면 외래 키에 null허용(DBA입장에서 치명적?)
  - 대상 테이블에 외래 키가 존재: 전통적인 데이터베이스 개발자 선호. 주 테이블과 대상 테이블 관계가 일대다로 변할 때 테이블 구조 유지되는 장점이 있지만 프록시 기능 한계로 지연 로딩으로 설정해도 항상 즉시 로딩된다.(JPA는 프록시 만들려면 객체에 값이 있는지 알아야 한다. 대상 테이블에 값이 있는지 보려면 주 테이블만 보면 모르니까 대상 테이블에 확인해야하므로 쿼리가 실행되므로 프록시가 필요없다.)
- 다대다는 실무에서 사용하면 안된다. 객체는 컬렉션을 사용해 객체 2개로 다대다 관계 가능하다. 연결 테이블이 연결만 하고 끝나지 않고 의도하지 않은 쿼리가 시행될 수 있다. 대신 @OneToMany, @ManyToOne을 만들고 알아서 중간 테이블(연결 테이블용 엔티티 추가)을 만들면된다. 여기서 원하는 데이터를 더 넣을 수 있다.

```java
    // Category.java
    @ManyToMany
    @JoinTable(name="CATEGORY_ITEM", joinColumns = @JoinColumn(name = "CATEGORY_ID"), inverseJoinColumns = @JoinColumn(name = "ITEM_ID"))
    private List<Item> items = new ArrayList<>();

    // Item.java
    @ManyToMany(mappedBy = "items")
    private List<Category> categories = new ArrayList<>();
```

- 셀프매핑

```java
@Entity
public class Category {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne  // 자식에게 부모 하나
    @JoinColumn(name="PARENT_ID")
    private Category parent;

    @OneToMany(mappedBy = "parent")
    private List<Category> child = new ArrayList<>();

    @ManyToMany
    @JoinTable(name="CATEGORY_ITEM", joinColumns = @JoinColumn(name = "CATEGORY_ID"), inverseJoinColumns = @JoinColumn(name = "ITEM_ID"))
    private List<Item> items = new ArrayList<>();
}

```



## 고급 매핑

### 상속관계 매핑

- 관계형 데이터베이스는 상속관계 없음. 대신 슈퍼-서브타입이라는 모델링 기법이 유사하다. 3가지 모두 jpa 지원
- 슈퍼타입 엔티티에서 @Inheritance(strategy =)를 지정해 각 전략을 사용할 수 있다. InheritanceType.JOINED, SINGLE_TABLE, TABLE_PER_CLASS(class도 abstract로 변경 필요) 그리고 각 서브 타입 엔티티에서 extends 슈퍼타입을 지정해야 한다.

1. 각각 테이블로 변환 / `조인전략`: ITEM에 item_id(pk), ALBUM에 item_id(pk ,fk)이 존재하고 각각의 필드를 가져 insert를 2번한다. 또 item에는 album뿐 아니라 movie, book등도 존재하는데 슈퍼클래스에 @DiscriminatorColumn 애노테이션을 붙이면 DTYPE이 생성되며 각 타입값이 DB에 저장된다. name속성을 통해 dtype이 아닌 다른 이름으로 할 수도 있다.  그 다음 서브 클래스에 @DiscriminatorValue를 지정하면 DTYPE에 서브클래스명이 기록된다. 클래스명말고 다른 값을 쓰고싶다면 value속성을 이용하면 된다.
   - 장점: 정규화가 되어있음. 제약조건을 슈퍼에 걸어 맞출 수 있고 공통 필드가 필요하면 ITEM테이블만 봐도 되는 경우가 있다. 외래 키 참조 무결성 제약조건 활용가능(FK활용), 저장공간 효율화
   - 단점: 조회 시 조인 많이 사용해 성능 저하, 조회 쿼리 복잡, 데이터 저장시 INSERT 2번 / 심각한 단점은 아니지만 관리하기가 복잡하긴 하다.
   - 가장 정석적인 방법
2. 통합 테이블 변환 / `단일 테이블 전략`: 논리모델 단일 테이블로 합치기. 작가, 음악가, 배우등 모든 필드 내용이 item이라는 하나의 테이블에 존재 / insert문이 한번만 실행되고 select도 심플해 성능이 좋다. / DTYPE생성하는거 지정안해도 필수기때문에 알아서 생성된다.
   - 장점: 조인 필요 없어서 조회 빠르고 조회 쿼리 단순
   - 단점: 자식 엔티티가 매핑할 컬럼은 모두 null 허용, 단일 테이블에 모든걸 저장해 테이블이 커질 수 있고 상황에 따라 조회 성능 느려질 수 있음.
3. 서브타입 테이블 변환 / `구현 클래스마다 테이블 전략`: 1번과 똑같지만 item없이 item에 공통으로 들어갈 수 있는 이름, 가격 등도 전부 앨범, 영화, 책 테이블에 따로 만든다. 단순할 때는 좋은데 조회할 때는 UNION으로 전부 합쳐서 검색해 복잡한 쿼리가 생성된다.
   - **쓰면안됨**
   - 예를 들어 정산해야할 때가 있으면 모든 테이블에 각각 시행해야 함
   - 장점: 서브 타입 명확히 구분해 처리, not null 사용가능
   - 단점: 여러 테이블 함께 조회 시 느림(UNION), 자식테이블 통합 쿼리 어려움.



### Mapped Superclass - 매핑 정보 상속

- 공통 매핑 정보가 필요할 때. 여러 테이블에 공통으로 들어가는 필드가 있을 때
- 상속관계 매핑 아님. 엔티티 아님. 테이블과 매핑 안됨.
- 부모 클래스를 상속 받는 자식 클래스에 매핑 정보만 제공.
- 조회, 검색 불가(em.find(BaseEntity) 불가)
- 직접 생성해서 사용할 일 없으므로 추상 클래스 권장
- 테이블과 관계 없이 엔티티가 공통으로 사용하는 매핑 정보를 모으는 역할로 등록일, 수정일, 수정인같은 공통 적용 정보 모을 때 사용. @Entity 클래스는 Entity나 @MappedSuperclass로 지정한 클래스만 상속 가능하다.

```java
package jpabook.jpashop.domain;

import javax.persistence.MappedSuperclass;

@MappedSuperclass
public class BaseEntity {
    private String createdBy;

    public String getCreatedBy() {
        return createdBy;
    }

    public void setCreatedBy(String createdBy) {
        this.createdBy = createdBy;
    }
}

// Team.java
@Entity
public class Team extends BaseEntity{
    ...
}
```



## 프록시와 연관관계 관리

### 프록시

- 테이블 조회할 때 외래키에 연관된 테이블까지 함께 조회해야할까?

- em.find(): db를 통해 실제 엔티티 조회
- em.getReference(): db 조회를 미루는 가짜(프록시) 엔티티 객체 조회 / getId는 이미 영속성 컨텍스트에 있어 찾은 객체의 id를 프린트해도 select 쿼리가 실행되지 않았지만 실제 사용되는 지점(username을 print할 때)에서 jpa가 db에 쿼리를 날려 요청한다. 
  - getName()요청 -> jpa가 영속성 컨텍스트에 초기화 요청 -> 영속성컨텍스트가 db에 조회 요청 -> 실제 entity 생성
- getClass를 print해서 살펴보면 프록시 객체가 보여진다. em.find는 진짜 객체를 주지만 getReference는 hibernate 내부라이브러리로 프록시라고 하는 가짜 엔티티 객체를 준다. 껍데기는 같지만 내부는 비어있는 객체다.
- 실제 클래스를 상속 받아 만들어지고 실제 클래스와 겉 모양이 같아 사용자는 진짜인자 프록시인지 구분하지 않고 사용하면 된다. 하지만 타입 체크시 ==는 실패고 instance of를 사용해야 한다.
- 프록시 객체는 실제 객체의 참조(target)를 보관. 프록시 객체를 호출하면 프록시 객체는 실제 객체의 메소드를 호출한다.
- jpa표준 스펙에는 없고 하이버네이트가 구현하는 것
- `특징`: 처음 사용할 때 한 번만 초기화. 초기화할 때 실제 엔티티로 바뀌는 게 아니다. 초기화되면 프록시 객체를 통해 `실제 엔티티에 접근 가능`. 하지만 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 getReference()를 호출해도 실제 엔티티를 반환한다. 그 반대의 상황도 마찬가지라 프록시를 먼저 조회했으면 다음에 진짜 객체 찾아도 프록시가 반환된다.
- 가져온 프록시를 detach, close, clear를 통해 준영속상태로 만들었을 때 프록시를 초기화하면 문제 발생. 
- 프록시 인스턴스의 초기화 여부 확인: PersistenceUnitUtil.isLoaded()
- 프록시 클래스 확인: entity.getClass().getName()
- 프록시 강제 초기화: org.hibernate.Hibernate.initialize(); / JPA표준은 강제 초기화X



### 즉시 로딩과 지연 로딩

- 위에서 Main클래스에서 Member로 팀을 찾을 때 하이버네이트가 보여주는 쿼리에서는 join을 사용한다. @ManyToOne(fetch=FetchType.LAZY)로 지정하면 쿼리가 분리되어 나간다.라고 했는데
  - 이렇게하면 객체를 proxy객체로 반환한다. 다대일 관계에서 다를 기준으로 일을 프록시로 가져온다.
- 즉시로딩: @ManyToOne(fetch = FetchType.EAGER) / 한번에 다 가져와서 프록시가 아니라 진짜 객체를 가져온다. / 실무에서는 가급적 사용하지 말자. 예상하지 못한 sql 발생, JPQL에서 N+1문제 발생(select를 시행할 때 연관관계 테이블을 함께 바로 가져온다. 결과 N개 + 처음 쿼리 1개) / ManyToOne, OneToOne은 기본이 즉시 로딩 / OneToMany, ManyToMany는 기본이 지연로딩



### 영속성 전이: CASCADE

- 특정 엔티티 영속 상태로 만들며 연관 엔티티도 함께 영속 상태로 만들고 싶을 때
- 영속성 전이는 연관관계 매핑과 관련이 없다. 함께 영속화하는 편리함뿐
- ALL: 모두 적용 / PERSIST: 영속 / REMOVE: 삭제 주로 사용 (그 외: MERGE, REFRESH, DETACH)
- 예를 들어 게시판-첨부파일 관계라면 쓸 수 있다. 그런데 파일이 다른 엔티티에서도 연관있다면 사용하면 안된다. `소유자가 하나일 때`만 써야한다.
- 전제조건: 라이프사이클이 같고(등록, 삭제 등) / 단일 소유자일 때만 사용하자.
- 고아객체: 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제. / orphanRemoval = true
  - 참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제하는 기능
  - **참조하는 곳이 하나일 때 사용**
  - 특정 엔티티가 개인 소유할 때 사용
  - OneToOne, OneToMany만 가능
  - CasCadeType.REMOVE처럼 동작
- CasCadeType.ALL + orphanRemoval = true로 지정하면 스스로 생명주기를 관리하는 엔티티는 em.persist()로 영속화, em.remove()로 제거한다. 그리고 부모 엔티티로 자식 생명주기를 관리해 도메인 주도 설계(DDD)의 Aggregate Root 개념(Aggregate Root만 컨택하고 그 외 리포지토리를 만들지 않고 root를 통해 생명주기를 관리한다?)을 구현할 때 유용하다.

```java
// Parent.java
@Entity
public class Parent {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Child> childList = new ArrayList<>();

    public void addChild(Child child) {
        childList.add(child);
        child.setParent(this);
    }
    
    // getters and setters
}

// Child.java
@Entity
public class Child {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "parent_id")
    private Parent parent;
    
    // getters and setters
}

// JpaMain.java

try {
            
            Child child1 = new Child();
            Child child2 = new Child();

            Parent parent = new Parent();
            parent.addChild(child1);
            parent.addChild(child2);
    
// 3번이나 persist 하지말고 parent중심으로 짜서 child를 관리하고 싶어서 cascade를 지정
            em.persist(parent);

            em.flush();
            em.clear();
            Parent findParent = em.find(Parent.class, parent.getId());
            findParent.getChildList().remove(0);
}
```





## 값 타입

- JPA의 데이터 타입 분류
  - 엔티티 타입: @Entity로 정의하는 객체. 데이터가 변해도 식별자로 지속해서 추적 가능
  - 값 타입: int, Integer, String처럼 단순 값으로 사용하는 자바 기본 타입과 객체. 식별자 없고 값만 있어 변경시 추적 불가 / 값 타입을 소유한 엔티티에 생명주기를 의존함. / 복잡한 객체를 단순화하긴 개념. 값 타입을 단순하고 안전하게 다룰 수 있어야 한다.



### 기본값 타입

- 자바 기본 타입(int, double)
- 래퍼 클래스(Integer, Long)
- String
- 생명주기를 엔티티에 의존. (회원 삭제하면 이름, 나이도 삭제)
- 값 타입은 공유하면 안된다. (회원 이름 변경 시 다른 회원 이름이 변경되면 안된다.)
- 자바의 기본 타입은 절대 공유X: 기본 타입은 항상 값을 복사한다. 저장공간이 서로 다르므로 원래 변수 값을 바꿔도 그걸 복사했던 변수의 값은 바뀌지 않는다. 래퍼 클래스(참조를 가져오기 때문에 공유가 된다)나 String같은 특수 클래스는 공유 가능 객체지만 변경X



### 임베디드 타입(복합 값 타입)

- `중요`

- x, y 좌표 같은것?
- 새로운 값 타입을 직접 정의할 수 있다. 주로 기본 값 타입을 모아서 만들어 복합 값 타입이라고도 한다. int, String과 같은 값 타입
- 장점: 클래스라서 재사용, 높은 응집도, 해당 값 타입만 사용하는 의미있는 메소드 생성 가능
- 임베디드 타입과 테이블 매핑: 임베디드 타입은 엔티티 값일 뿐이다. 사용 전후 매핑하는 테이블은 같다. 그런데 객체와 테이블을 `아주 세밀하게(find-grained)` 매핑하는 것이 가능하다. 잘 설계한 ORM 애플리케이션은 매핑한 테이블 수보다 클래스 수가 더 많다.
- @Embedded로 같은 클래스를 2번 넣으면 에러 발생. 둘 중 하나에 @AttributeOverrides(value=AttributeOverride(name="city", column=@Column("WORK_CITY") ... )로 지정해주면 컬러 명 속성을 재정의한다.
- 임베디드 타입의 값이 null이면 매핑한 컬럼은 모두 null이 된다.

```java
// Address.java
@Embeddable
public class Address {
    private String zipcode;
    private String street;
    
    // constructors, getters and setters
}

// Member.java
@Entity
public class Member {
    ...
    @Embedded
    private Address homeAddress;
}
```



### 값 타입과 불변 객체

- 임베디드 타입 같은 값 타입을 여러 엔티티에서 공유하면 부작용이 일어날 수 있다. 
- 진짜 둘 다 변경하고 싶었어도 값 타입이 아니라 엔티티를 써야 한다.

```java
// JpaMain.java
try {
            Address address = new Address("street", "zipcode");

            Member member = new Member();
            member.setUsername("member1");
            member.setHomeAddress(address);
            em.persist(member);

            Member member2 = new Member();
            member2.setUsername("member2");
            member2.setHomeAddress(address);
            em.persist(member2);

            member.getHomeAddress().setStreet("newStreet"); // 첫번째 멤버 주소만 바꾸고 싶지만 실행해보면 둘 다 바껴있다.
    

            tx.commit();

        }
```

- 값 타입의 실제 인스턴스인 값을 공유하는 것은 위험하므로 대신 값을 복사해서 사용해야 한다. 그런데 실수로 복사한 값을 넣으려다 원래 값을 넣어도 컴파일러 레벨에서 막을 수 있는 방법이 없다. 

```java
Address copyAddress = new Address(address.getZipcode(), address.getStreet());
            Member member2 = new Member();
            member2.setUsername("member2");
            member2.setHomeAddress(copyAddress);
```



- `객체 타입의 한계`: 객체 타입은 참조 값을 직접 대입하는 것을 막을 방법이 없어 객체의 공유 참조는 피할 수 없다.
- 불변 객체: 객체 타입을 수정할 수 없게 만들어 부작용 원천 차단. `값 타입은 생성 시점 이후 절대 값을 변경할 수 없는 객체인 불변 객체`로 설계해야 함. 생성자로만 값을 설정하고 수정자를 만들지 않거나 private으로 만들면 된다. Integer, String은 자바가 제공하는 대표적인 불변객체다.
- 그렇다면 값을 바꾸고싶을 때는? 생성자를 새로 만들어 다시 set



### 값 타입의 비교

- 값 타입은 인스턴스가 달라도 값이 같으면 같다고 봐야한다. 그런데 기본타입말고 객체타입은 값 같아도 false로 나온다.
- 동일성(identity) 비교: 인스턴스 참조 값 비교, == 사용
- 동등성(equivalence) 비교: 인스턴스 값 비교, equals()사용
- 값 타입은 equals를 사용해 동등성 비교를 해야 하므로 적절히 재정의(주로 모든 필드에서 사용) / hashCode도 같이 해줘야함.



### 값 타입 컬렉션

- `중요`
- 자바 컬렉션에 기본이나 임베디드 값을 넣어준 것
- 값 타입을 하나 이상 저장할 때 사용.  @ElementCollection, @CollectionTable 사용. db는 컬렉션을 같은 테이블에 저장할 수 있고 이를 위한 별도의 테이블이 필요하다.
- 모든 라이프 사이클가 원래 클래스에 소속되어 있어 의존한다. 별도로 persist, update할 필요 없다.
- @Embedded가 아닌 값 타입 컬렉션들은 지연로딩이기 때문에 원 테이블을 조회할 때 함께 나오지 않는다.
- 각 타입은 불변이기 때문에 수정할 때 객체 컬렉션은 set하지 않고 새로운 생성자로 다시 넣는다. String 같은 기본값 컬렉션은 remove, add를 활용한다. 객체 컬렉션을 remove, add하면 원 객체와 관련있는 데이터를 모두 지운 후 삭제 하지 않은 데이터를 넣고 수정을 위해 새로 add한 데이터를 넣는다.
- 영속성 전이와 고아객체 제거 기능을 필수로 가진다.
- 값 타입은 엔티티와 다르게 식별자 개념이 없어 변경하면 추적이 어렵다. 변경 사항이 발생하면 주인 엔티티와 연관된 모든 데이터를 삭제하고 현재 값을 모두 다시 저장한다. @OrderColumn(name=...)같은 걸 써서 순서 값을 넣고 pk지정이 되지만 의도하지 않게 동작하는 경우가 많다.
- 값 타입 컬렉션을 매핑하는 테이블은 모든 컬럼을 묶어 기본 키를 구성해야 한다. nullX, 중복 저장X
- 너무 복잡하게 쓰려면 차라리 다르게 써야 한다. 실무에서 상황에 따라 값 타입 컬렉션 대신 일대다 관계를 고려한다. 일대다 관계를 위한 `엔티티를 만들고` 여기서 값 타입을 사용해서 `영속성 전이 + 고아 객체 제거를 사용`해 값 타입 컬렉션처럼 사용한다.

```java
// Member.java
    @ElementCollection
    @CollectionTable(name="FAVORITE_FOOD", joinColumns = @JoinColumn(name = "MEMBER_ID"))
    @Column(name = "FOOD_NAME")  // 얘만 매핑하게 허용해줌? 값이 하나고 내가 정의한게 아니라
    private Set<String> favoriteFoods = new HashSet<>();

    @ElementCollection
    @CollectionTable(name="ADDRESS", joinColumns = @JoinColumn(name = "MEMBER_ID"))
    private List<Address> addressHistory = new ArrayList<>();
```

- 정리
  - 엔티티 타입: 식별자 O, 생명 주기 관리, 공유
  - 값 타입: 식별자X, 생명 주기 엔티티 의존, 공유하지 않는 것이 안전(복사해서 사용), 불변객체가 안전
- 값 타입은 정말 값타입이라고 판단될 때만 사용한다. 식별자가 필요하고 지속해서 값 추적, 변경해야 하면 값 타입이 아닌 `엔티티`



## 객체지향 쿼리 언어1 - 기본 문법

### 소개

- JPA는 다양한 쿼리 방법을 지원

- 가장 단순한 조회 방법인 em.find()대신 where조건이 필요한 조회를 해야한다면?

- JPQL: 표준 문법. JPA를 사용하면 엔티티 객체 중심으로 개발하는데 문제는 검색할 때 테이블이 아니라 엔티티 객체 대상으로 검색한다. 모든 DB데이터를 객체로 변환해 검색하는 것은 불가능하므로 애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL 필요 -> JPA가 SQL을 추상화한` JQPL이라는 객체 지향 쿼리 언어`를 제공. SQL과 문법이 유사하며 SELECT, FROM, WHERE, JOIN같은 ANSI표준이 지원하는 문법 지원하며 엔티티 객체 대상으로 쿼리하므로 특정 DB SQL에 의존하지 않는다.

- JPA Criteria: 문자가 아닌 자바 코드로 짜서 JPQL을 빌드해주는 제너레이터. 원래 쿼리문은 단순 String이기 때문에 동적 쿼리를 만들기 힘들다. 단점은 sql스럽지 않다. 사실 강사님은 실무에서 안쓴다고 한다. 자기도 못알아봐서 유지보수가 어려웠다. 그래서 `QureyDSL` 사용 권장

  ```java
  // 사용 준비
  CriteriaBuilder cb = em.getCriteriaBuilder();
  CriteriaQuery<Member> query = cb.createQuery(Member.class);
  
  // 루트 클래스 (조회 시작할 클래스)
  Root<Member> m = query.from(Member.class);
  
  // 쿼리 생성. where부터 분리해서 조건에 맞는 경우만 where을 넣을 수도 있다.
  CriteriaQuery<Member> cq = query.select(m).where(cb.equal(m.get("username"), "kim"));
  List<Member> resultList  = em.createQuery(cq).getResultList();
  ```

- QueryDSL: 자바 코드로 짜서 JPQL을 빌드해주는 제너레이터. 컴파일 시점에 문법 오류 찾을 수 있음. 동적쿼리 작성 편리하고 단순하고 쉽다.

  ```java
  JPAQueryFactory query = new JPAQueryFactory(em);
  QMember m = QMember.member;
  
  List<Member. list = query.selectFrom(m)
      				     .where(m.age.gt(18))
      				     .orderBy(m.name.desc())
      					 .fetch();
  ```

  

- 네이티브 SQL: 표준 SQL문법을 벗어나서 특정 DB에 종속적으로 사용해야 할 때

- JDBC API 직접 사용, MyBatis, SpringJdbcTemplate함께 사용: 영속성 컨텍스트를 적절한 시점에 강제 플러시 필요. createNativeQuery같은건 기본적으로 플러시가 동작. 

