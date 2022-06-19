# **스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술**

[링크](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/dashboard)



## 프로젝트 환경설정

### 프로젝트 생성

- https://start.spring.io: 스프링 부트 기반 프로젝트를 만들어주는 사이트
- Maven / Gradle: 라이브러리를 가져오고 빌드하는 라이프사이클을 관리해주는 툴. 요즘은 거의 Gradle
- Artifact: 빌드되어 나올 때 결과물
- Dependencies: Spring Web, Thymeleaf(Template engines)
- 다운로드 받은 스프링부트 압축을 풀고 프로젝트 안의 build.gradle을 open
- 프로젝트 내부
  - .idea: 인텔리제이 설정파일
  - gradle/wrapper: gradle 관련
  - src/main, test: main/java밑에 실제 패키지와 소스파일. test는 테스트 관련 소스 코드(요즘 중요)
  - resources: xml, properties, html등 자바 제외한 파일들
  - build.gradle: 요즘은  직접 쓰지않고 start.spring.io를 통해서 설정 파일이 제공된다.
    - sourceCompatibility: java version
    - dependencies: 전에 선택한 2개와 기본 테스트 관련 dependency
    - repositories: mavenCentral()에서 dependencies들 다운
  - gradlew, gradlew.bat: build할 때 사용
  - settings.gradle: 일단 패스
- 스프링이 톰캣 웹서버를 내장하고 있어 자체적으로 띄우면서 웹서버를 시작한다.
- Settings - Gradle에서 Build and run using, Run test using을 Gradle이 아닌 Intellij IDEA로 변경하면 실행 시 gradle을 거치지않아 빨라진다.



### 라이브러리 살펴보기

- gradle에서 의존관계를 관리하는데 spring-boot-starter-web 라이브러리를 가져올 때 여기서 필요로하는 tomcat, webmvc같은 의존관계가 있다. gradle이 이런 걸 전부 가져온다.
- spring-boot-starter에 스프링 부트 + 코어 + 로깅
  - system.out.println대신 log를 사용해서 확인해야한다. 기본적으로 가져오는 log중 slf4j보다 logback을 사용한다.
- spring-boot-starter-test: junit(요즘 거의5), mockito, assertj, spring-test 등



### View 환경설정

- src/main/resources/static에 index.html을 만들어 실행 시 바로 나오는 웰컴페이지를 만든다.
- [spring boot reference documentation](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#web.servlet.spring-mvc.welcome-page) : welcome page를 위해 index.html(정적 페이지)을 찾고 없으면 index template을 찾아 welcome page로 사용한다.
- thymeleaf 템플릿을 적용한 페이지를 위해 resources/templates에 hello.html를 생성한다. p태그의 th는 thymeleaf를 의미한다.

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
  <meta http-equiv="Content-Type" content="text/html"; charset="UTF-8">
  <title>Hello</title>
</head>
<body>
<p th:text="'안녕하세요. ' + ${data}"> 안녕하세요. 손님</p> 
</body>
</html>
```

- hello페이지를 위한 controller를 위해 main/java/hello.hellospring에 controller를 만들고 HelloController.java를 생성한다.

```java
package hello.hellospring.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HelloController {

    @GetMapping("hello")  // hello라는 get요청이 들어올 때 실행
    public String hello(Model model) {
        model.addAttribute("data", "hello!!");  // attributeName, attributeValue
        return "hello";   // resources에 존재하는 hello를 찾아서 렌더링해라
    }
}
```



### 빌드하고 실행하기

- hello-spring에서 windows는 ./gradlew.bat build를 통해 빌드한다. 생성된 build폴더의 libs로 이동해  java -jar hello-spring-0.0.1-SNAPSHOT.jar을 실행하면 서버가 시작된다. 반드시 intellij에서 실행했던 서버는 종료하고 실행해야하며 ctrl+c로 종료가능하다.
- 잘안될경우 clean하면 build 폴더 사라지므로 ./gradlew.bat clean build하면 지우고 다시 빌드된다.



## 스프링 웹 개발 기초

### 정적 컨텐츠

- 서버에서 파일을 그대로 브라우저로 보내는 것
- localhost:8080/hello.html 요청 -> 내장 톰캣 서버를 거쳐 hello-static 관련 컨트롤러를 찾고 없으면 resouces에서 찾는다.



### MVC와 템플릿 엔진

- 서버에서 그대로가 아니라 동적으로 작동하도록 한 것
- view는 화면을 보여주는데만 집중. 

src/main/resources/templates/hello-template.html

- 이 파일 자체를 absolute path로 열어봤을 때(파일 위치) 서버 없어도 서버를 켠 것처럼 보여주는게 thymeleaf의 장점으로 p태그 뒤에 쓰인 default 내용이 보인다. 실제 서버를 타면 text에 적힌대로 name값이 적용된 내용이 보인다.

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta http-equiv="Content-Type" content="text/html"; charset="UTF-8">
    <title>Hello</title>
</head>
<body>
<p th:text="'hello. ' + ${name}" > hello! empty</p> 
</body>
</html>
```



src/main/java/hello.spring/controller

- name이라는 param을 받아서 template에 넘겨줄 수 있도록한다. required를 쓰지않으면 default가 true라서 name값이 없으면 에러가 발생한다. (?name=spring)
- requestParam과 연관된 parameter(required처럼)을 보고싶다면 ctrl + p
- viewResolver가 templates에서 hello-template을 찾아서 template engine을 연결한다.

```java
    @GetMapping("hello-mvc")
    public String helloMvc(@RequestParam(name = "name", required = false) String name, Model model) { 
        // viewResolver가 templates에서 view 찾고 tempate engine연결
        model.addAttribute("name", name);
        return "hello-template";
    }
```



### API

- 정적컨텐츠가 아닐경우 view를 찾아 템플릿 엔진을 통해 html을 브라우저로 넘기던가 api방식으로 데이터를 바로 내리는가 2가지가 있다. 

src/main/java/hello.spring/controller

- @ResponseBody: http 응답하는 body에 내용을 넣는다. 어노테이션이 없으면 viewResolver에게 요청이 간다. 하지만 있으면 HttpMessageConverter가 동작해서 stringConvertor로 html없이 그대로 데이터만 넘긴다. 첫번째는 그래서 문자가 그대로 넘겨졌고 두 번째는 객체인 경우라서 jsonConverter가 동작해 spring이 name을 json으로 만들어서 http응답에 반환했다.

```java
@GetMapping("hello-string")
    @ResponseBody 
    public String helloString(@RequestParam("name") String name) {
        return "hello " + name; 
    }

    @GetMapping("hello-api") 
    @ResponseBody 
    public Hello helloApi(@RequestParam("name") String name) {
        Hello hello = new Hello();
        hello.setName(name);
        return hello;
    }

    static class Hello {
        private String name;

        public String getName() {
            return name;
        }
        public void setName(String name) {
            this.name = name;
        }
    }
```



## 회원 관리 예제 - 백엔드 개발

### 비즈니스 요구사항 정리

- 데이터: id, 이름 / 기능: 등록, 조회
- 저장소: 데이터베이스에 접근, 도메인 객체를 DB에 저장하고 관리. 메모리기반 데이터 저장소(추후 바꾸기 위해 interface로 구현 클래스 변경할 수 있도록 설계)
- 도메인: 비즈니스 도메인 객체. 회원, 주문, 쿠폰 등 DB에 저장하고 관리됨.

### 회원 도메인과 리포지토리 만들기

domain/Member

```java
package hello.hellospring.domain;

public class Member {

    private Long id;
    private String name;

    // getters & setters
    ...
}
```



repository/MemberRepository(interface)

- Optional: 가져오는데 null일 때 처리하기 위해 optinal로 감싸서 반환

```java
package hello.hellospring.repository;

import hello.hellospring.domain.Member;

import java.util.List;
import java.util.Optional;

public interface MemberRepository {
    Member save(Member member);
    Optional<Member> findById(Long id);
    Optional<Member> findByName(String name);
    List<Member> findAll();
}
```



repository/MemoryMemberRepository

```java
package hello.hellospring.repository;

import hello.hellospring.domain.Member;

import java.util.*;

public class MemoryMemberRepository implements MemberRepository{

    private static Map<Long, Member> store = new HashMap<>();
    private static long sequence = 0L;

    @Override
    public Member save(Member member) {
        member.setId(++sequence);
        store.put(member.getId(), member);
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) { // Optional을 써서 null인 경우 대처
        return Optional.ofNullable(store.get(id));
    }

    @Override
    public Optional<Member> findByName(String name) {
        return store.values().stream().filter(member -> member.getName().equals(name)).findAny();
    }

    @Override
    public List<Member> findAll() {
        return new ArrayList<>(store.values());
    }
}
```



### 회원 리포지토리 테스트 케이스 작성

- 개발 기능 테스트할 때 main method실행하거나 컨트롤러를 실행하지만 오래 걸리고 반복 실행이 어렵고 여러 테스트를 한 번에 실행하기 어렵다. 그래서 jUnit 프레임워크로 테스트를 실행한다. 여럿이서 함께 개발할 때 테스트 작성은 필수적이며 테스트를 먼저 작성 후 개발하는 것을 TDD라고 한다.

src/test/java/hello.hellospring/repository/MemoryMemberRepositoryTest

- main/java에서 repository package에 작성한 코드를 테스트하기 위해 똑같이 repository package를 test에 만들었다.
- 한꺼번에 테스트를 실행할 때 순서에 따라 repository에 저장되는 객체가 중복되어 오류가 생길 수 있다. @AfterEach를 적용해 테스트를 실행 후 repository를 clear하도록 만든다.
- Assertions.assertEquals(member, result)는 org.junit.jupiter.api.Assertions를 import해서 사용한다.
- assertThat(member).isEqualTo(result)는 org.assertj.core.api.Assertions에 존재한다. assertThat이 위보다 편리해서 사용했다.

```java
package hello.hellospring.repository;

import hello.hellospring.domain.Member;
// import org.junit.jupiter.api.Assertions;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Test;

import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

class MemoryMemberRepositoryTest {
	
    // 원래 interface인 MemberRepository로 지정되어있었으나 MemoryMemberRepository만 테스트하기 때문에 변경했다.
    MemoryMemberRepository repository = new MemoryMemberRepository();

    @AfterEach
    public void afterEach() {
        repository.clearStore();
    }

    @Test
    public void save() {
        Member member = new Member();
        member.setName("spring");

        repository.save(member);
        // Optional이라서 get으로 값 꺼내기
        Member result = repository.findById(member.getId()).get(); 
        
        assertThat(member).isEqualTo(result);
    }

    @Test
    public void findByName() {
        Member member1 = new Member();
        member1.setName("spring1");
        repository.save(member1);

        Member member2 = new Member();
        member2.setName("spring2");
        repository.save(member2);

        Member result = repository.findByName("spring1").get();

        assertThat(result).isEqualTo(member1);
    }

    @Test
    public void findAll() {
        Member member1 = new Member();
        member1.setName("spring1");
        repository.save(member1);

        Member member2 = new Member();
        member2.setName("spring2");
        repository.save(member2);

       List<Member> result = repository.findAll();

       assertThat(result.size()).isEqualTo(2);
    }
}
```



repository/MemoryMemberRepository

- 테스트 한 번 실행 후 저장소를 비울 수 있도록 메서드를 추가했다.

```java
    public void clearStore() {
        store.clear();
    }
```



### 회원 서비스 개발

- 변수 추출하기: Ctrl + Alt + V
- 메서드 추출하기: Ctrl + Alt + M

main/java/hello.hellospring/service

```java
package hello.hellospring.service;

import hello.hellospring.domain.Member;
import hello.hellospring.repository.MemberRepository;
import hello.hellospring.repository.MemoryMemberRepository;

import java.util.List;
import java.util.Optional;

public class MemberService {

    private final MemberRepository memberRepository = new MemoryMemberRepository();

    /*
    * 회원가입
    */
    public Long join(Member member) {
        // 같은 이름이 있는 중복 회원x
        validateDuplicateMember(member);
        memberRepository.save(member);
        return member.getId();
    }

    private void validateDuplicateMember(Member member) {
        // Optional로 감싸서 ifPresent같은 optional 메서드를 활용할 수 있다. 요즘은 ifNull보단 null가능성이 있으면 Optional사용. 또는 orElseGet 사용
        memberRepository.findByName(member.getName()).ifPresent(m -> {
            throw new IllegalStateException("이미 존재하는 회원입니다.");
        });
    }

    /*
     * 전체 회원 조회. repository이름과 다르게 서비스클래스는 비지니스적인 이름 사용.
     */
    public List<Member> findMembers() {
        return memberRepository.findAll();
    }

    public Optional<Member> findOne(Long memberId) {
        return memberRepository.findById(memberId);
    }
}
```



### 회원 서비스 테스트

- 테스트를 만들고 싶은 클래스에 커서를 두고 alt + enter로 자동 테스트 생성
- 테스트는 빌드될 때 실제 서비스에 포함되지 않아 메서드를 한글로 만들어도 상관없다.
- 이전 run 다시하기 : shift + f10

MemberService.java

```java
public class MemberService {

    // 원래 memberRepository를 바로 생성했으나 test에서 같은 repository로 검증하기 위해 constructor를 이용하는 방식으로 변경한다.
    private final MemberRepository memberRepository;

    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
    
    ...
}
```



MemberServiceTest.java

```java
package hello.hellospring.service;

import hello.hellospring.domain.Member;
import hello.hellospring.repository.MemoryMemberRepository;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.api.Assertions.*;

class MemberServiceTest {

    MemberService memberService;

    // 원래 테스트에서 new memberRepository, memberService를 생성했다. 이럴 경우 서비스에서의 repository와 다른 객체가 다시 생성된 상황이기 때문에 같은 걸 써서 검증하도록 수정했다.
    MemoryMemberRepository memberRepository;

    // 외부에서 repository를 memberService에 넣어준다. dependency injection
    @BeforeEach
    public void beforeEach() {
        memberRepository = new MemoryMemberRepository();
        memberService = new MemberService(memberRepository);
    }

    @AfterEach
    public void afterEach() {
        memberRepository.clearStore();
    }

    @Test
    void 회원가입() {
        // given
        Member member = new Member();
        member.setName("hello");

        // when
        Long saveId = memberService.join(member);

        // then(결과)
        Member findMember = memberService.findOne(saveId).get();
        assertThat(member.getName()).isEqualTo(findMember.getName());
    }

    @Test
    public void 중복_회원_예외() {
        // given
        Member member1 = new Member();
        member1.setName("spring");

        Member member2 = new Member();
        member2.setName("spring");

        // when
        memberService.join(member1);
        IllegalStateException e = assertThrows(IllegalStateException.class, () -> memberService.join(member2));
        assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");
        
        // 중복회원이 있을 때 생기는 IllegalStateException이 아니라서 테스트 실패.
        // assertThrows(NullPointerException.class, () -> memberService.join(member2));

        // try catch문을 사용해도 되지만 좀 어색
        //        try {
//            memberService.join(member2);
//            fail();
//        } catch(IllegalStateException e) {
//            assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.ㄴㄴ");
//        }

        // then
    }

    @Test
    void findMembers() {
    }

    @Test
    void findOne() {
    }

```



## 스프링 빈과 의존관계

### 컴포넌트 스캔과 자동 의존관계 설정

- 스프링 빈 등록
  - 컴포넌트 스캔과 자동의존관계 설정: @Component가 있으면 자동으로 스프링 빈으로 등록된다. 원래 @Service, Controller, Repository가 아닌 @Component지만 Service등에 들어가보면 Component가 이미 붙어 있다. 
  - 자바 코드로 직접 스프링 빈 등록
- 현재 메인 메서드인 HelloSpringApplication에서 시작해서 컨테이너가 이 패키지에서 찾아 스프링 빈을 등록하기 때문에 그 외 패키지에 @Component가 있어도 등록하지 않는다.
- 스프링은 컨테이너에 빈을 등록할 때 `기본으로 싱글톤으로 등록한다.` 같은 스프링 빈이면 모두 같은 인스턴스
- 컨트롤러 -> 서비스 -> 레포지토리로 연결 (의존관계 주입. dependency injection)

MemberController.java

```java
package hello.hellospring.controller;

import hello.hellospring.service.MemberService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;

// 스프링 컨테이너가 생성할 때 컨트롤러(빈)를 객체로 가져와서 관리(빈 관리)
@Controller
public class MemberController {

    // new로 생성해서 쓸 수 있지만 스프링이 관리할 수 있도록 컨테이너에 등록해 받아쓰는 방식이 되어야 한다. memberController말고 여기저기서 memberService를 쓸 수도 있는데 new로 여러개의 인스턴스를 생성할 필요가 없다.
    private final MemberService memberService;

    // 스프링 컨테이너가 멤버 서비스를 가져와서 스프링이 Autowired를 통해 알아서 연결시켜줘야 하는데 memberService는 순수한 자바 클래스로 작성되어 등록되어 있지 않다.
    // 생성자를 통해 주입
    @Autowired
    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }
}

```



MemberService.java

```java
 // Service로 등록
@Service
public class MemberService {
    ...
}
```



MemoryMemberRepository.java

- interface가 아닌 구현체에서 Repository로 등록

```java
@Repository
public class MemoryMemberRepository implements MemberRepository{
    ...
}
```



### 자바 코드로 직접 스프링 빈 등록하기

- @Service, Repository를 제거하고 직접 등록하기(Controller는 그대로)
- 과거에는 xml이라는 문서로 설정했으나 현재 거의 사용하지 않는다. 
- DI
  - 필드주입: private MemberService memberService앞에 바로 @Autowired 입력하는 방식이 있지만 별로 추천하지 않는다.
  - 생성자 주입: 우리가 한 방식. 처음 어플리케이션 조립 시점에 생성하고 변경을 못하게 한다.
  - setter주입:  setMemberService를 통해 여기에 @Autowired를 넣는다. 단점은 controller를 호출할 때 public이라 누구나 호출할 수 있어 중간에 잘못 바꾸면 문제가 생긴다.
- 정형화된 컨트롤러, 서비스, 리포지토리는 컴포넌트 스캔을 사용하지만 정형화되지 않은 `상황에 따라 구현 클래스를 변경`하는 경우에는 설정을 통해 스프링 빈으로 등록



src/main/java/hello.hellospring/SpringConfig.java

```java
package hello.hellospring;

import hello.hellospring.repository.MemberRepository;
import hello.hellospring.repository.MemoryMemberRepository;
import hello.hellospring.service.MemberService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SpringConfig {

    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
}
```





## 회원 관리 예제 - 웹 MVC 개발

### 회원 웹 기능  - 홈 화면 추가

HomeController.java

```java
package hello.hellospring.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HomeController {

    @GetMapping("/")
    public String home() {
        return "home";
    }
}
```



templates/home.html

```html
    <p>회원 기능</p>
    <p>
      <a href="/members/new">회원 가입</a>
      <a href="/members">회원 목록</a>
    </p>
```

- static에 index.html이 있지만 우선순위에 따라 스프링 컨테이너에서 관련 컨트롤러를 먼저 찾기 때문에 HomeController에 mapping된 게 있어 호출된다.



### 회원 웹 기능 - 등록, 조회



## 스프링 DB 접근 기술

### H2 데이터베이스 설치

- 용량이 작고 가벼운 교육용 데이터베이스
- 설치 후 윈도우는 ./h2.bat을 실행한다.
- JDBC URL이 파일로 접근하게 되어있으면 애플리케이션과 웹 콘솔이 동시에 접근되지 않아 jdbc:h2:tcp://localhost/~/test로 변경한다. 여러군데서 접근할 수 있게 한다.
- 테이블 클릭하면 자동으로 SELECT문이 나온다.

```sql
drop table if exists member CASCADE;
CREATE TABLE member
(
    id bigint generated by default as identity, /* 빈 값 일 때 id값 채워줌 */
    name varchar(255),
    primary key(id)
);
```

- 필요하면 src와 같은 위치에 sql디렉토리를 만들어 sql문을 적어둔다.



### 순수 JDBC

- build.gradle에 jdbc, h2 라이브러리를 추가해야 한다. 
- main아래 application.propertis에 datasource.url과 driver-class-name을 추가한다. build.gradle에서 refresh가 필요하다.
- repository package에 jdbcMemberRepository를 MemberRepository를 구현해 만든다. memoryMemberRepository에서 jdbcMemberRepository로 변경해서 사용. 

```java
private final DataSource dataSource;

public JdbcMemberRepository(DataSource dataSource) {
    this.dataSource = dataSource;
}

public Member save(Member member) {
    ...
}
```

- 개방-폐쇄 원칙(OCP, Open-Closed Principle): 확장에 열리고 수정, 변경에 닫혀있다.
- 스프링 DI를 사용하면 `기존 코드를 전혀 손대지 않고 설정만으로 구현 클래스를 변경`할 수 있다.



SpringConfig.java

```java
@Configuration
public class SpringConfig {

    private final DataSource dataSource;
	
    // dataSource에 에러가 뜨지만 작동에 문제가 없다. 단순히 잘못 표시된거같다.
    public SpringConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }
    ...
}
```





### 스프링 통합 테스트

- 이전에 한 테스트는 스프링과 관련 없는 자바 테스트(**단위 테스트**). 단위 테스트가 사실 더 좋은 테스트다. 단위로 쪼개서 하고 스프링 없이 할 수 있도록 훈련해야 한다.
- 테스트 전용 DB에서 따로 실행한다. 한 번 실험을 하고나면 데이터가 db에 남아있어 오류가 발생한다. AfterEach를 전에 사용했지만 db는 transaction 개념이 있어 commit을 해야 반영이 된다.
- @Transactional이 테스트 케이스에 붙어있으면 테스트가 끝나고 rollback해준다. service같은데 붙어 있으면 rollback하지 않는다. @Commit같은 옵션도 존재함.
- 통합 테스트 전에 같은 데이터가 있으면 안되기 때문에 미리 비우고 시작한다. Transactional과는 별개로

```java
@SpringBootTest // 스프링 컨테이너와 테스트를 함께 실행한다.
@Transactional
class MemberServiceIntegrationTest {
    
    // 테스트는 제일 편한 방식으로 한다.
    @Autowired MemberService memberService;
    @Autowired MemberRepository memberRepository;
    
}
```



### 스프링 Jdbc Template

- 순수 jdbc와 동일한 환경설정. JDBC API에서 본 반복 코드를 대부분 제거하지만 sql은 직접 작성 필요
- jdbc template은 아직 사용하는 곳이 많다.



JdbcTemplateMemberRepository

- alt enter로 private RowMapper<Member> memberRowMapper()를 람다로 변경
- 생성자가 딱 하나면 @Autowired 생략 가능. 스프링이 dataSource 자동으로 injection.

```java
public class JdbcTemplateMemberRepository implements MemberRepository{
    
    private final JdbcTemplate jdbcTemplate;

    public JdbcTemplateMemberRepository(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }

    @Override
    public Member save(Member member) {
        // jdbcInsert를 통해 쿼리 짤 필요 없이 테이블 명, pk, column명만 있으면 알아서 만들어준다.
        SimpleJdbcInsert jdbcInsert = new SimpleJdbcInsert(jdbcTemplate);
        jdbcInsert.withTableName("member").usingGeneratedKeyColumns("id");

        Map<String, Object> parameters = new HashMap<>();
        parameters.put("name", member.getName());

        Number key = jdbcInsert.executeAndReturnKey(new MapSqlParameterSource(parameters));
        member.setId(key.longValue());
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) {
        // 디자인 패턴중 템플릿을 적용됨.
        List<Member> result = jdbcTemplate.query("select * from member where id = ?", memberRowMapper(), id);
        return result.stream().findAny();
    }

    @Override
    public Optional<Member> findByName(String name) {
        List<Member> result = jdbcTemplate.query("select * from member where name = ?", memberRowMapper(), name);
        return result.stream().findAny();
    }

    @Override
    public List<Member> findAll() {
        return jdbcTemplate.query("select * from member", memberRowMapper());
    }
    
/*    private RowMapper<Member> memberRowMapper() {
        return new RowMapper<Member>() {
            @Override
            public Member mapRow(ResultSet rs, int rowNum) throws SQLException {
                Member member = new Member();
                member.setId(rs.getLong("id"));
                member.setName(rs.getString("name"));
                return member;
            }
        };
    } */
    
    private RowMapper<Member> memberRowMapper() {
        // alt enter로 람다로 변경
        return (rs, rowNum) -> {
            Member member = new Member();
            member.setId(rs.getLong("id"));
            member.setName(rs.getString("name"));
            return member;
        };
    }
}
```



### JPA(Java Persistance API)

- JdbcTemplate에서 반복은 줄었어도 sql은 여전히 개발자가 작성해야 했다. JPA를 사용해 JPA가 만들어주도록 한다. SQL, 데이터 중심 설계 -> `객체 중심 설계`로 패러다임 전환. `개발 생산성 크게 향상`
- build.gralde에 jpa관련 라이브러리 추가해준다.
- application.properties에도 추가해준다. jpa를 사용하고 spring.jpa.hibernate.ddl-auto=create를 해주면 table까지 자동으로 만들어주지만 실습에서는 none으로 둔다.
- JPA는 자바의 표준 인터페이스라 구현은 여러개인데 hibernate를 사용해서 한다.
- JPA는 ORM(Object Relational Mapper) 기술이다. 



Member.java

- JPA를 사용하기 위해 추가해준다. 
- 만약 coulmn name이 데이터베이스와 다르다면 @Column(name="username")처럼 추가해 mapping해준다.

```java
@Entity
public class Member {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)  // db가 알아서 생성해주는건 identity
    private Long id;
    
    ...
}
```



JpaMemberRepository.java

- jpa는 entityManager로 모든게 동작한다. spring boot가 자동으로 em을 생성해서 그 만들어진걸 injection만 해주면 된다.

```java
package hello.hellospring.repository;

import hello.hellospring.domain.Member;

import javax.persistence.EntityManager;
import java.util.List;
import java.util.Optional;

public class JpaMemberRepository implements MemberRepository {

    private final EntityManager em;

    public JpaMemberRepository(EntityManager em) {
        this.em = em;
    }

    @Override
    public Member save(Member member) {
        em.persist(member);
        return member;
    }
	
    // id는 pk값이라 바로 찾아 사용 가능하다.
    @Override
    public Optional<Member> findById(Long id) {
        Member member = em.find(Member.class, id);
        return Optional.ofNullable(member);
    }

    @Override
    public Optional<Member> findByName(String name) {
        List<Member> result = em.createQuery("select m from Member m where m.name = :name", Member.class).setParameter("name", name).getResultList();
        return result.stream().findAny();
    }

    @Override
    public List<Member> findAll() {
        // jpql이라는 쿼리 언어. table이 아닌 entity를 대상으로 쿼리를 날리면 sql로 번역된다. member entity자체를 select해서 조회. pk기반이 아닌걸 찾을 때 사용
        return em.createQuery("select m from Member m", Member.class).getResultList();
    }
}
```



MemberService.java

```java
// JPA를 쓰려면 항상 @Transactional이 필요하다. 데이터 저장하거나 변경할 때. 회원 가입 때만 필요하니까 join에 붙여도 된다.
@Transactional
public class MemberService {
	...
}
```



SpringConfig.java

```java
@Configuration
public class SpringConfig {
    
    private EntityManager em;

    @Autowired
    public SpringConfig(EntityManager em) {
        this.em = em;
    }
    
    ...
        
    @Bean
    public MemberRepository memberRepository() {
        return new JpaMemberRepository(em);
    }
}
```



### 스프링 데이터 JPA

- 인터페이스만으로 개발 완료 가능. crud도 스프링 데이터 JPA가 모두 제공한다. 스프링 데이터 JPA는 반드시 JPA를 학습한 후 학습하자.
- 복잡한 동적 쿼리는 Querydsl이라는 라이브러리를 사용한다. 쿼리를 자바 코드로 작성할 수 있다. 이 조합으로도 해결하기 어려우면 JPA가 제공하는 네이티브 쿼리나 JdbcTemplate를 사용한다.

SpringDataJpaMemberRepository.java

- 인터페이스를 만들면 구현체를 자동으로 만들어 스프링 빈에 등록해준다. findAll(), findById()처럼 많이 쓰는게 이미 만들어져있다.
- paging기능도 자동 제공
- email이나 username등 공통할 수 없는 걸로 메서드를 만들어야 한다면 findByName은 select m from Member m where m.name=? 처럼 이름에 따라 쿼리를 만들어주기 때문에 findByNameAndId, findByEmail처럼 이름 규칙을 잘 지켜서 만들면된다.

```java
package hello.hellospring.repository;

import hello.hellospring.domain.Member;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface SpringDataJpaMemberRepository extends JpaRepository<Member, Long>, MemberRepository {

    @Override
    Optional<Member> findByName(String name);
}

```



SpringConfig.java

- 현재 등록한 memberRepository가 없지만 interface만 만들어둔 SpringDataJpaMemberRepository가 알아서 interface구현체를 직접 만들어 등록한상태다.

```java
@Configuration
public class SpringConfig {
    private final MemberRepository memberRepository;

    public SpringConfig(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
    
    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository);
    }

}
```



## AOP

### AOP가 필요한 상황

- 모든 메서드 호출 시간을 측정하고 싶으면 각 메서드마다 시간을 측정하도록 코드를 추가해야 한다. 그런데 이건 핵심 기능도 아닌데다가 핵심 기능과 섞여있어 유지보수를 힘들게 만든다. 
- 공통 관심 사항(cross-cutting concern), 핵심 관심 사항(core concern)



### AOP 적용

- Aspect Oriented Programming
- 시간 측정 로직(공통 관심 사항)을 한 군데 모아 원하는 곳에 적용
- 원래 controller에서 서비스를 호출하고 있었는데 AOP적용 후 컨트롤러에서 `프록시`(가짜 멤버 서비스)를 만들어서 컨테이너가 올라와 스프링 빈을 등록할 때 가짜 스프링 빈을 먼저 등록하고 이후에  joinPoint.proceed()를 호출해 내부적으로 여기저기 태운 후 실제 서비스를 호출한다.
  - 이를 확인하기 위해 MemberController의 생성자에서 System.out.println("memberService= " + memberService.getClass());로 확인하면 콘솔창에는 memberService= class hello.hellospring.service.MemberService$$EnhancerBySpringCGLIB$$5b8ac77b라고 출력된다. 진짜 서비스가 아니라 CGLIB로 인한 proxy객체가 출력되는걸 볼 수 있다. 
- controller, service, repository 순으로 진행되던게 전부 앞에 각각 프록시를 호출한 다음 진짜 controller, service, repository를 호출하도록 바뀌는 것이다.
- DI덕분에 이런 기술 가능



src/main/java/hello.spring/aop/TimeTraceAop.java

- @Around: 타겟팅. 패키지명, 클래스명, 파라미터 타입 등 원하는 조건 넣기. 여기서는 해당 패키지에 전부 적용하도록 했다. 호출 될 때마다 joinPoint에서 원하는 걸 조작할 수 있다.
- AOP를 @Component로 등록해서 찾게 하거나 SpringConfig에서 등록해서 사용하면 된다. @Service, @Repository처럼 정형화되어 등록하는 경우는 바로 가능하지만 AOP같은 경우는 SpringConfig에서 빈 등록을 권장한다고 했다. 그런데 막상 해보니 오류가 났는데 [순환참조 문제](https://www.inflearn.com/questions/48156)였다. SpringConfig 등록으로 사용하려면 Around 내용을 바꿔줘야 한다.

```
    @Bean
    public TimeTraceAop timeTraceAop() {
        return new TimeTraceAop();
    }
```

```java
package hello.hellospring.aop;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class TimeTraceAop {

    @Around("execution(* hello.hellospring..*(..))")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        System.out.println("START: " + joinPoint.toString());
        try {
            return joinPoint.proceed();
        } finally {
            long finish = System.currentTimeMillis();
            long timeMs = finish - start;
            System.out.println("END: " + joinPoint.toString() + " " + timeMs + "ms");
        }
    }
}
```

- 콘솔창 예시

```
START: execution(String hello.hellospring.controller.HomeController.home())
END: execution(String hello.hellospring.controller.HomeController.home()) 5ms
```





## 다음으로
