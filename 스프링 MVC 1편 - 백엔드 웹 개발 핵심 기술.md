# 스프링 MVC 1편 - 백엔드 웹 개발 핵심 기술

<br>

## 웹 애플리케이션의 이해

### 웹 서버, 웹 애플리케이션 서버

- 웹: HTTP기반으로 통신. 웹 브라우저(클라이언트)에서 url을 치면 인터넷을 통해 서버에 접근. 서버는 html을 만들어 클라이언트에 보내준다. 전송, 응답할 때 모두 http 프로토콜을 통한다. 
- html, text, image, json 등 거의 모든 형태의 데이터 전송 가능. 서버간 데이터 주고 받을 때도 대부분 http 사용
- 웹 서버: http 기반동작. 정적 리소스(특정 폴더에 html, css, 이미지 등을 두면 서버가 그 파일들을 그냥 보내준다. ), 기타 부가기능 제공. 대표적으로 nginx, apache
- 웹 애플리케이션 서버(was): http 기반 동작. 웹 서버 기능 포함(특히 정적 리소스 제공), 프로그램 코드를 실행해 애플리케이션 로직을 수행한다. 동적 html, http api(json). 또 서블릿, jsp, 스프링 mvc가 동작한다. 사용자에 따라 동적으로 보여줄 수 있다.(프로그래밍을 할 수 있으니까). 대표적으로 tomcat jetty, undertow
- 웹 서버와 was차이: 웹 서버는 정적 리소스, was는 애플리케이셔 로직. 그러나 사실 경계가 모호함. 웹 서버도 프로그램 실행 기능을 포함하거나 was도 웹 서버 기능을 제공해서. 자바는 서블릿 컨테이너 기능을 제공하면 was다. 
- 웹 시스템 구성: was, db만으로 구성 가능. was는 정적 리소스와 애플리케이션 로직을 모두 제공 가능하다. 그런데 was가 너무 많은 역할을 하며 서버 과부하 우려가 있다. 가장 비싼 로직이 정적 리소스 때문에 수행이 어려울 수 있다. 또 was 장애 시 오류 화면도 노출이 불가능하다.
- 따라서 정적 리소스는 웹서버가 처리하고 동적인 처리가 필요하면 was에 위임한다. 이러면 효율적인 리소스가 관리가 가능하다. 정적 리소스가 많이 사용되면 웹 서버 증설, 애플리케이션 리소스가 많이 사용되면 was 증설. 또 웹 서버는 잘 죽지 않아 was, db장애 시 web 서버가 오류 화면 제공. api로 데이터만 제공받으면 웹 서버가 없어도 된다.



### 서블릿

- 소켓 연결, 메시지 파싱, 응답 메시지 생성 등 복잡한 과정은 자동화해주고 데이터베이스에 값을 저장하는 핵심 로직만 수행하게한다.
- 그래도 http 스펙은 어느정도 알아야한다.
- http 요청 흐름: was가 request response객체 새로 만들어 서블릿 호출. 개발자가 request, response 객체에서 http 정보를 사용. was가 response에 담긴 내용으로 응답 정보 생성에 웹 브라우저에 전송
- 서블릿 컨테이너: 서블릿 객체 자동 생성, 호출, 생명주기까지 관리. 톰캣처럼 서블릿을 지원하는 was를 서블릿 컨테이너라고 한다. 서블릿 객체는 싱글톤으로 관리한다. requiest, resoponse객체는 요청이 들어올 때마다 생성되지만 동일한 서블릿 객체 인스턴스에 접근한다. 최초 로딩 시점에 미리 만들어둔다. `공유 변수 사용 주의`해야한다. 잘못하면 다른 회원 이름이 보이는 등의 에러 발생할 수 있음. 서블릿 객체는 컨테이너가 종료될 때 함께 종료된다. jsp도 서블릿으로 변환되어 사용하며 `동시 요청을 위한 멀티 쓰레드 처리를 지원한다.`



### 동시 요청 - 멀티 쓰레드

- 백엔드 개발자에게 중요한 개념
- 서블릿을 호출하는건 **쓰레드**
- 쓰레드 애플리케이션 코드를 하나하나 순차적으로 실행함. 메인 메서드를 실행하면 main이란 이름의 쓰레드가 실행된다. 쓰레드는 한 번에 하나의 코드 라인만 수행하므로 동시 처리가 필요하면 쓰레드를 추가 생성한다. 
- 요청이 올 때마다 쓰레드를 생성한다면
  - 장점: 동시 요청 처리, 리소스(cpu, 메모리) 허용할 때까지 처리가능, 하나 지연되어도 나머지 동작
  - 단점: 생성 비용이 비싸다(메모리 등), 응답 속도가 늦어진다, 컨텍스트 스위칭 비용이 발생한다(코어 수), 생성에 제한이 없어 cpu, 메모리 임계점을 넘어 서버가 죽는다.
- 쓰레드 풀: 쓰레드들이 있다가 요청이 오면 풀에 요청 -> 사용-> 사용후 죽이지 않고 반납. 약 200개를 만들어뒀는데 그 이상의 요청이 온다면? 나머지 요청은 대기, 거절
  - 장점: 미리 생성되어 있어 생성 종료 비용(cpu) 절약, 응답 시간 빠름. 너무 많은 요청이 와도 기존 요청 안전하게 처리 가능.
- was의 주요 튜닝 포인트는 `최대 쓰레드 수`다. 너무 낮으면 서버 리소스가 여유롭지만 클라이언트는 응답이 지연된다. 너무 높으면 cpu, 메모리 임계점을 초과해 서버가 다운된다. 클라우드라면 장애 발생 시 서버를 늘리고 튜닝하지만 아니면 그냥 튜닝.
- 적정 숫자는?  로직 복잡도, cpu, 메모리, io 리소스 상황에 따라 다르다. 최대한 실제 서비스와 유사하게 성능 테스트를 시도 해야란다. 아파치 ab, 제이미터, nGrider등
- 멀티 쓰레드에 대한 부분은 was가 지원한다. 개발자가 관련 코드를 신경쓰지 않아도 되지만 서블릿이나 스프링 빈같은 싱글톤 객체는 주의해야 한다.



### HTML, HTTP API, CSR, SSR

- 정적 리소스, 동적인 html(템플릿)
- http api: html이 아닌 데이터 전달. 주로 JSON형식. 다양한 시스템에서 호출한다. 예를 들면 서버 to 서버(주문과 결제, 기업간), 앱, 웹 클라이언트 to 서버(ui클라이언트. 자바스크립트를 통해 호출한다. react, vue.js등도 웹 클라이언트다.) 등. UI화면이 필요하면 클라이언트가 별도 처리한다.
- SSR(서버 사이드 렌더링): 서버에서 최종 HTML을 생성해 클라이언트에 전달. 주로 정적인 화면에 사용. JSP, 타임리프
- CSR(클라이언트 사이드 렌더링): HTML 결과를 자바스크립트를 사용해 브라우저에서 동적으로 생성해서 적용. 주로 동적인 화면에 사용. 웹 환경을 마치 앱처럼 부분부분 변경 가능하다. 
  - 웹 브라우저가 서버에 요청하면 빈 HTML과 자바스크립트 링크를 보낸다. 웹 브라우저가 서버에 자바스크립트에 요청을 보내면 클라이언트 로직, HTML 렌더링 코드를 반환한다. 그 다음 웹 브라우저가 서버에 HTTP API로 데이터 요청을 보내면 서버가 DB에서 조회해서 데이터를 보내준다. 
- CSR + SSR을 동시에 지원하는 프레임 워크도 있다.



### 자바 백엔드 웹 기술 역사

- SERVLET, JSP나 여러 MVC 프레임워크가 존재했지만 `애노테이션 기반`인 스프링 MVC가 등장하며 MVC 프레임워크계를 평정했다. 유연, 편리, 깔끔한 코드 작성 가능. 
- 스프링 부트가 등장. 서버 내장 / 과거에는 서버에 WAS직접 설치하고 소스는 WAR파일을 만들어 설치한 WAS에 배포. -> 빌드 결과(Jar)에 WAS 서버 포함해 빌드 배포 단순화
- 스프링 웹 기술 분화: Web Servlet(Spring MVC) / Web Reactive(Spring WebFlux, 완전 최신 기술)
  - Spring WebFlux: 비동기 넌 블러킹 처리. 최소 쓰레드로 최대 성능(쓰레드 컨텍스트 스위칭 비용 효율화. 코어가 4개라면 쓰레드도 4~5개 정도라서 계속 작업 가능) / 함수형 스타일 개발(동시처리 코드 효율화) / 서블릿 기술 X
  - 그러나 난이도 매우 높고 관계형 데이터베이스 지원 부족하며 일반 MVC 쓰레드도 빠른편이고 실무에서 거의 사용하지 않고 있음.
- 자바 뷰 템플릿 역사
  - JSP: 속도 느리고 기능 부족
  - 프리마커,  벨로시티: 성능 속도 좋지만 발전을 잘 안함
  - 타임리프: 내추럴 템플릿(HTML 유지하면서 뷰 템플릿 적용) / 스프링 MVC와 강력한 기능 통합 / 성능은 프리마커, 속도는 벨로시티가 낫다.



## 서블릿

### 프로젝트 생성

- spring initializer: gradle, packaing(war - jsp를 사용할 수 있도록), dependencies(web, lombok)
- lombok사용할 때 settings-annotation에서 enable annotation processing체크하기



### Hello 서블릿

- was서버 설치 -> 서블릿 코드 클래스 파일로 빌드해서 올리고 -> 톰캣 실행이지만 스프링 부트가 톰캣 서버 내장하고 있어 편리하다.

src/main/java/hello.servlet/servletApplication.java

- @ServletComponentScan: 패키지 안에서 서블릿 찾아서 등록

```java
@ServletComponentScan 
@SpringBootApplication
public class ServletApplication {

	public static void main(String[] args) {
		SpringApplication.run(ServletApplication.class, args);
	}

}
```



src/main/java/hello.servlet/basic/HelloServlet.java

- ctrl o: select methods to Override/implement를 열어서 service가져오기(열쇠 달린거)
- soutm: print문에 클래스명과 메서드명 출력
- soutv: print문에 변수
- sout: print문

```java
package hello.servlet.basic;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet(name = "helloServlet", urlPatterns = "/hello")
public class HelloServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // soutm
        System.out.println("HelloServlet.service");
        //soutv
        System.out.println("request = " + request);
        System.out.println("response = " + response);

        // query parameter를 넣으면
        String parameter = request.getParameter("username");
        System.out.println("parameter = " + parameter);

        response.setContentType("text/plain");
        response.setCharacterEncoding("utf-8");
        // http body에 입력하기
        response.getWriter().write("hello " + parameter);
    }
}
```

- application.properties에 logging.level.org.apache.coyote.http11=debug를 적어서 실행 시 설정 내용 콘솔창에 보이도록 하기



### HttpServletRequest - 기본 사용법

src/main/java/hello.servlet/basic/request/RequestHeaderServlet.java

- host, 언어, cookie, content 정보 등도 출력 가능하다.

```java
package hello.servlet.basic.request;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Enumeration;

@WebServlet(name="requestHeaderServlet", urlPatterns = "/request-header")
public class RequestHeaderServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        headerPrintOld(request);
        headerPrintNew(request);
    }
    private void headerPrintNew(HttpServletRequest request) {
        System.out.println("요즘 방식 시작");
        // 요즘 방식
        request.getHeaderNames().asIterator()
                .forEachRemaining(headerName -> System.out.println(headerName + ": " + headerName));
        System.out.println("요즘 방식 끝");
    }

    private void headerPrintOld(HttpServletRequest request) {
        System.out.println("예전 방식 시작");
        // 예전 방식.
        Enumeration<String> headerNames = request.getHeaderNames();
        while (headerNames.hasMoreElements()) {
            String headerName = headerNames.nextElement();
            System.out.println(headerName + ": " + headerName);
        }
        System.out.println("예전 방식 끝");
    }
}
```



### HTTP 요청 데이터 - 개요

- HTTP 요청 메시지를 통해 클라이언트에서 서버로 데이터 전달
- GET - 쿼리 파라미터 url?username=hello&age=20 / 메세지 바디없이 url 쿼리파라미터에 데이터 포함해 전달. 검색, 필터, 페이징에서 사용
- POST - HTML Form / content-type: application/x-www-form-urlencoded / 메세지 바디에 쿼리 파라미터 형식으로 전달 / 가입, 주문 등
- HTTP message body - 데이터 직접 담아서 요청. http api에서 주로 사용(json, xml,  text) / POST, PUT, PATCH



### HTTP 요청 데이터 - GET 쿼리 파라미터

- 서버에서 HttpServletRequest가 제공하는 다음 메서드를 통해 편하게 조회: getParameter("name"), getParameterNames(), 
- name이 같은 복수 파라미터가 있다면? getParameterValues의 첫번째 값이 나온다. getParameterNames도 name은 하나고 값이 여러개인 상황이기때문에 하나만 출력된다. getParameterValues("name")을 사용해 전체 가져오기 가능.
- getParmeter는 항상 String이기 때문에 숫자형이 필요하면 Integer.parseInt로 감싸줘야한다.



### HTTP 요청 데이터 - POST HTML Form

- Form에서 데이터를 전송해도 쿼리 파라미터 출력하는 로직을 사용하면 파라미터가 출력된다. `쿼리 파라미터 조회 메서드를 그대로 사용하면 된다. `서버 입장에서는 두 개가 동일하다. 다만 http 메세지 바디에 해당 데이터를 포함해 보내기 때문에 content-type 지정이 꼭 필요하다.



### HTTP 요청 데이터 - API 메세지 바디 - 단순 텍스트

src/main/java/hello.servlet/basic/request/RequestBodyStringServlet.java

- getInputStream()을 통해 메세지 body 내용을 바이트코드로 바로 얻을 수 있다. 

```java
package hello.servlet.basic.request;

import org.springframework.util.StreamUtils;

import javax.servlet.ServletException;
import javax.servlet.ServletInputStream;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.nio.charset.StandardCharsets;

@WebServlet(name = "requestBodyStringServlet", urlPatterns = "/request-body-string")
public class RequestBodyStringServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        
        ServletInputStream inputStream = request.getInputStream();
        
        // 바이트->문자, 문자->바이트 할 때어떤 인코딩인지 알려줘야한다.
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
        
        System.out.println("messageBody = " + messageBody);
        response.getWriter().write("ok");
    }
}
```



### HTTP 요청 데이터 - API 메세지 바디 - JSON

src/main/java/hello.servlet/basic/HelloData.java

- lombok을 사용해 getter, setter 메서드 작성할 필요 없어짐.

```java
package hello.servlet.basic;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class HelloData {

    private String username;
    private int age;

}
```



src/main/java/hello.servlet/basic/request/RequestBodyJsonServlet.java

- 받은 json데이터를 파싱해서 자바객체로 변환하기 위해 라이브러리 사용. 스프링 부트로 스프링 mvc를 선택하면 기본으로 제공(jackson, objectMapper)
- formdata전송 방식으로 할 때도 StreamUtils.copyToString()을 사용가능하지만 getParameter()가 더 편리해서 굳이 사용하지 않는다.

```java
package hello.servlet.basic.request;

import com.fasterxml.jackson.databind.ObjectMapper;
import hello.servlet.basic.HelloData;
import org.springframework.util.StreamUtils;

import javax.servlet.ServletException;
import javax.servlet.ServletInputStream;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.nio.charset.StandardCharsets;

@WebServlet(name = "requestBodyJsonServlet", urlPatterns = "/request-body-json")
public class RequestBodyJsonServlet extends HttpServlet {

    private ObjectMapper objectMapper = new ObjectMapper();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        ServletInputStream inputStream = request.getInputStream();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
        System.out.println("messageBody = " + messageBody);

        HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);

        System.out.println("helloData = " + helloData.getUsername());
        System.out.println("helloData = " + helloData.getAge());

        response.getWriter().write("ok");
    }
}
```



### HttpServletResponse - 기본 사용법

- HTTP 응답 메세지 생성. 응답코드 지정, 헤더와 바디 생성. Content-Type, 쿠키, Redirect같은 편의 기능 제공

- response.setStatus(HttpServletResponse.SC_OK);  / 200이라 써도 되지만 의미가 보여지면 좋다.
- response.setHeader("Content-Type", "text/plain"); / 이렇게 header를 직접 설정하면 되는데 바로 setContet-Type을 쓰는 것도 가능하다. header를 설정할 때 원래 없는 옵션의 헤더를 만들어서 지정하는 것도 가능하다.
- cookie또한 setHeader의 Set-Cookie에 Max-Age=600이 가능하지만 cookie객체를 만들어 setMaxAge등을 적용한 후 response.addCookie를 하면 편하다.
- response.sendRedirect("주소")를 통해 redirect가능



### HTTP 응답 데이터 - 단순 텍스트, HTML

- 단순 텍스트 응답, HTML응답, HTTP API - JSON응답
- setContentType("text/html"), setCharacterEncoding을 한 후 reponse.getWriter()를 통해 가져온 PrintWriter를 사용해 writer.println("<html>")처럼 html코드로 응답할 수 있다. 웹 브라우저에서는 html이 적용된 화면이 보인다.



### HTTP 응답 데이터 - API JSON

- setContentType("text/html"), setCharacterEncoding(무조건 utf-8을 사용해야 해서 굳이 추가안해도 됨 / 다만 response.getWriter()는 추가 파라미터를 자동으로 추가하므로 response.getOutputStream()으로 출력하면 그런 문제가 없다. )을 한 후 name, age를 가지고있는 HelloData객체를 생성해 값을 담는다. 그리고 objectMapper.writeValueAsString(helloData); response.getWriter().write(result)를 사용해 json으로 변환된 값을 반환한다.



## 서블릿, JSP, MVC 패턴

### 회원 관리 웹 애플리케이션의 요구사항



src/main/java/hello.servlet/basic/domain/member/MemberRepository.java

- Member 클래스도 만들어두기

```java
package hello.servlet.domain.member;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class MemberRepository {

    // 동시성 문제가 고려되어 있지 않아 실무에서 ConcurrentHashMap, AtomicLong 사용 고려
    private static Map<Long, Member> store = new HashMap<>();
    private static long sequence = 0L;
	
    // 스프링을 사용하지 않고 싱글턴을 적용하기 위해서
    private static final MemberRepository instance = new MemberRepository();

    public static MemberRepository getInstance() {
        return instance;
    }

    private MemberRepository() {
    }

    public Member save(Member member) {
        member.setId(++sequence);
        store.put(member.getId(), member);
        return member;
    }

    public Member findById(Long id) {
        return store.get(id);
    }

    public List<Member> findAll() {
        // store.values()를 보호
        return new ArrayList<>(store.values());
    }

    public void clearStore() {
        store.clear();
    }
}
```



### 서블릿으로 회원 관리 웹 애플리케이션 만들기

- 서블릿과 자바 코드로 HTML을 만드는 작업이 너무 비효율적이라 HTML문서에 동적으로 변경할 부분을 자바 코드로 넣는 템플릿 엔진이 나타났다. JSP는 성능, 기능 면에서 밀려나 사장되어 가는 추세다. 요즘 주로 사용하는건 Thymeleaf



### JSP로 회원 관리 웹 애플리케이션 만들기

- jsp를 사용하기 위해 dependencies에 추가해야한다.
- jsp 파일 맨 위에는 <%@ page contentType="text/html;charset=UTF-8" language="java" %>가 필요하다.
- jsp 파일 안에 class를 import하기 위해서 <%@ page import="파일위치" %>가 필요하며 <% %> 안에 들어가는 문장은 java코드다.
- JSP의 문제는 JSP가 너무 많은 역할을 해 코드가 너무 길어지는 것이다. 비지니스 로직과 뷰 로직이 같이 있기 때문이다.



### MVC 패턴 - 개요

- html을 수정해야 할 때도 자바코드가 함께 있고 반대의 경우도 마찬가지다. 또한 이 두 일은 변경 라이프 사이클이 다를 확률이 높고 서로 영향을 주지 않는다. 그런데 이런 두 작업을 하나의 코드로 관리하고 있어 유지보수면에서 좋지 않다.
- MVC 패턴
  - 컨트롤러: HTTP 요청을 받아 파라미터 검증, 비즈니스 로직 실행(service에 들어이씩 때문에 service를 호출). 뷰에 전달할 결과 조회해 모델에 담기 / 컨트롤러에 비즈니스 로직이 있으면 컨트롤러의 일이 너무 과중해진다.
  - 모델: 뷰에 출력할 데이터 담기.
  - 뷰: 모델에 담긴 데이터를 사용해 화면을 그림.

### MVC 패턴 - 적용



### MVC 패턴 - 한계
