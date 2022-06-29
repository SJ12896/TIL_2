# 실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발

## 프로젝트 환경설정

- lombok을 사용하면 꼭 settings - Annotation Processors에서 enable해주자.

- 의존관계가 궁금하면 ./gradlew dependencies
- 스프링 데이터 JPA는 스프링, JPA를 먼저 이해하고 사용하는 응용 기술이다.
- thymeleaf: natural template이라 마크업을 깨지않고 사용. 웹 브라우저에서도 열림. 그런데 버전2에서는 br도 꼭 태그닫아줬어야 함.
- html바꾸고 계속 서버 껐다 켜기 귀찮으니까 dependencies에 추가 implementation 'org.springframework.boot:spring-boot-devtools' 추가후 build-recompile하면 된다.
- application.properties대신 복잡할 때 더 나은 application.yml을 생성해 사용한다. jpa부분 띄어쓰기가 잘못되어 테이블이 생성되지 않는 오류가 있었다. 주의하자.

```yaml
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/jpashop;
    username: sa
    password:
    driver-class-name: org.h2.Driver
    
  jpa:
    hibernate:
    # 애플리케이션 실행에서 내가 가진 엔티티 지우고 다시 만들기
      ddl-auto: create
    properties:
      hibernate:
  #      show_sql: true
        format_sql: true
      
logging:
  level:
  # debug모드라 모든 sql이 보인다. shoq_sql은 system.out으로 출력하는거고 이건 log
    org.hibernate.SQL: debug
    # 쿼리문 실행 후 아래 bindingParameter의 순서, 타입, 값이 나온다. 더 편하게 보려면 외부 라이브러리 spring-boot-data-source-decorator를 build.gradle에 추가. 그런데 이건 운영에선 성능을 고려한 후 사용해야 한다.
    org.hibernate.type: trace
```

- 테스트시에 자동 롤백을 막으려면 @Rollback(false) 지정
- 스프링 부트를 통해 설정이 자동화되어 persistence.xml없고 LocalContainerEntityManagerFactoryBean도 없다.
- 값 타입은 변경 불가능하게 설계해야 한다. @Setter제거하고 값을 모두 초기화해 변경 불가능한 클래스를 만들자. JPA 스펙상 엔티티, 임베디드 타입은 자바 기본 생상자를 public, protected로 해야하는데 protected가 그나마 안전하다. 제약 이유는 JPA 구현 라이브러리가 객체 생성할 때 리플렉션같은 기술을 사용할 수 있도록 지원해야 하기 때문이다.
  - 아무것도 없는 기본 생성자 protected, 모든 필드 들어가는 생성자를 public으로 해서 public을 사용하도록 한다.



### 엔티티 설계시 주의점

- 엔티티에는 가급적 Setter를 사용하지 말자. 변경 포인트가 너무 많아 유지보수가 어렵다.
- `모든 연관관계는 지연로딩으로 설정`: 매우 중요. 즉시 로딩(한 엔티티 조회할 때 연관된 것들 한 번에 같이 조회)은 예측이 어렵고 어떤 sql이 실행될지 추적하기 어렵다. JPQL실행시 N+1문제 있을 수 있다. 함께 조회해야하는 경우에는 `fetch join`이나 엔티티 그래프 기능 사용. 특히 `XToOne`(OneToOne, ManyToOne)관계는 기본이 즉시로딩이므로 변경 필요
- 컬렉션은 필드에서 바로 초기화하는 것이 안전. -> NULL에서 안전. 하이버네이트는 엔티티 영속화 할 때 컬렉션을 감싸 하이버네이트 내장 컬렉션으로 변경한다. 따라서 필드 레벨에서 하는 것이 안전, 간결.
- 테이블, 컬럼명 생성: 스프링 부트에서 하이버네이트 기본 매핑 전략을 변경해 실제 테이블 명 다름. 기존에는 엔티티 필드명 그대로 테이블 명으로 사용했으나 
  - 스프링 부트 신규 설정(엔티티(필드) -> 테이블(컬럼)): 카멜케이스는 언더스코어 / 점도 언더스코어 / 대문자는 소문자
- cascade를 설정하면 각자해줄 필요없이 연쇄식으로 저장할 수 있다.
- 연관관계 메서드 설정
- 애플리케이션 아키텍처
  - controller, web: 웹 계층
  - service: 비지니스 로직, 트랜잭션 처리
  - repository: JPA 직접 사용, 엔티티 매니저 사용
  - domain: 엔티티 모여있는 계층, 모든 계층에서 사용
- service에서 조회할 때 성능 향상시키기

```java
    @Transactional(readOnly = true)
    public List<Member> findMembers() {
        return memberRepository.findAll();
    }
```

- service에서 repository 사용하는법

```java
// 1. repository를 테스트할 때 접근할 수 없어 바꿀 수 없다.
private MemberRepository memberRepository;

// 2. 테스트코드 작성할 때 좋지만 조립 이후에 set할 일은 없어서
    @Autowired
    public void setMemberRepository(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

// 3. 요즘 많이 쓰는 방식 생성자 injection 원래 써야하지만 최신 버전에서는 @Autowired 안써도 생성자 하나면 알아서 해준다.
    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

// 4. 모든 필드로 생성자 만들어주기
@AllArgsConstructor 
public class MemberService {
    private MemberRepository memberRepository;
    ...
}

// 5. final 있는 애들로만 생성자. 위의 더 나은 버전
@RequiredArgsConstructor 
public class MemberService {
    private final MemberRepository memberRepository;
    ...
}
```

- repository에서도 RequiredArgsConstructor사용

```java
@Repository
@RequiredArgsConstructor
public class MemberRepository {

    // @PersistenceContext 대신 autowired도 지원 스프링 부트 데이터 jpa가 해줌. 기본 라이브러리는x. 하지만 RequiredArgsConstructor 쓸 수 있으니까 final로 바꾸고 애노테이션
    private final EntityManager em;
}
```



- 가입 메서드를 테스트할 때 em.persist를 하지만 insert문이 실행되지 않는다. commit될 때 flush가 되면서 실행되기 때문. 스프링 트랜잭션은 기본이 rollback이라 쿼리 자체가 안보인 것이다. @Rollback(false)를 하거나  em.flush()를 한다.
- test폴더에 resources를 만들고 여기에 application.yml을 만들면 이게 우선권을 가지게 된다. 그래서 여기서 datasource를 바꾸면 h데이터베이스를 in-meory로 동작하게 바꿀 수 있다. 그런데 스프링 부트에서는 기본적으로 별도 설정 없으면 메모리 모드로 실행한다. 