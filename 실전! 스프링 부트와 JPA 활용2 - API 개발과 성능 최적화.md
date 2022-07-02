# 실전! 스프링 부트와 JPA 활용2 - API 개발과 성능 최적화

- [강의 링크](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-API%EA%B0%9C%EB%B0%9C-%EC%84%B1%EB%8A%A5%EC%B5%9C%EC%A0%81%ED%99%94)



## API 개발 기본

### 회원 등록 API

- api 컨트롤러와 템플릿 엔진 컨트롤러를 다른 패키지에 작성한다. 에러가 있을 때 return 해야하는 값 같 등 작성해야 하는 내용이 다르다. 

api/MemberApiController

- @Controller + @ResponseBody = `@RestController`
- v1에서 @RequsetBody는 json으로 온 body를 member에 그대로 매핑해준다. json외도 설정으로 가능하다. api를 만들 때 엔티티를 파라미터에 넣고 외부에 노출하지 않는다.
- valid annotation이 스프링 2.3이상부터는 사라져서 따로 의존성을 추가해줘야 했다. valid덕분에 @notEmpty를 검증해준다.
- 그런데 v1의 api처럼 사용하려면 presentation의 검증 로직이 엔티티에 들어가있어 엔티티에서 @NotEmpty를 지정하면 어떤 api에선 필수 값이지만 다른 api에서는 아닐 수 있어 문제가 된다.
- 또 만약 엔티티 필드명을 바꾸면 v2는 메서드가 실행될 때 set부분에서 오류를 확인 가능하지만 v1은 엔티티 자체를 손대야 해서 api 스펙 자체가 바뀌게 된다. v1은 파라미터가 어떤 것이 넘어올지 api스펙 문서를 보지 않는 이상 모른다.
- 따라서 엔티티와 api스펙이 1:1 매핑된 상황이기 때문에 별도의 data transfer object, dto가 필요하다. 

```java
@RestController
@RequiredArgsConstructor
public class MemberApiController {

    private final MemberService memberService;
    @PostMapping("/api/v1/members")
    public CreateMemberResponse saveMemberV1(@RequestBody @Valid Member member) {
        Long id = memberService.join(member);
        return new CreateMemberResponse(id);
    }

    @PostMapping("/api/v2/members")
    public CreateMemberResponse saveMemberV2(@RequestBody @Valid CreateMemberRequest request) {
        Member member = new Member();
        member.setName(request.getName());

        Long id = memberService.join(member);
        return new CreateMemberResponse(id);
    }

    // DTO
    @Data
    static class CreateMemberRequest {
        @NotEmpty
        private String name;
    }

    @Data
    static class CreateMemberResponse {
        private Long id;

        public CreateMemberResponse(Long id) {
            this.id = id;
        }
    }
}
```



### 회원 수정 API

- 강사님은 DTO에서는 lombok annotation 많이 쓰시는 편. 엔티티에선 getter setter 정도만
- update에서 바로 member반환하면 영속상태가 끊긴(트랜잭션 범위를 넘어가기 때문에) member가 반환된다.
- 등록, 수정, 삭제는 커맨드로 id를 반환해주는 것이 좋지만 수정은 어떤 데이터를 반환하는 것이 좋지 않다?

```java
// MemberApiController

    @PutMapping("/api/v2/members/{id}")
    public UpdateMemberResponse updateMemberV2(@PathVariable Long id, @RequestBody @Valid UpdateMemberRequest request) {

        memberService.update(id, request.getName());
        Member findMember = memberService.findOne(id);
        return new UpdateMemberResponse(findMember.getId(), findMember.getName());
    }

    @Data
    static class UpdateMemberRequest {
        private String name;
    }

    @Data
    @AllArgsConstructor
    static class UpdateMemberResponse {
        private Long id;
        private String name;
    }

// MemberService.java

    @Transactional
    public void update(Long id, String name) {
        Member member = memberRepository.findOne(id);
        member.setName(name);
    }
```



### 회원 조회 API

- v1처럼 바로 return할 경우 엔티티 직접 노출되어 일대다 관계인 orders까지 전부 노출된다. Member orders 컬럼에서 @JsonIgnore를 사용하면 숨길 수 있지만 api를 요청하는 케이스마다 필요할 수도 있고 아닐 수도 있는데다가 엔티티에 화면(presentation)을 위한 로직이 들어가있으면 좋지 못하다. 또한 반환하는 멤버 수를 세서 함께 반환하는 등의 추가 확장을 할 수 없다.
- v2는 api스펙이 dto와 일대일로 엮여있고 반환을 위한 Return 클래스가 존재해 size()처럼 추가확장한 반환이 가능하다.

```java
    @GetMapping("/api/v1/members")
    public List<Member> membersV1() {
        return memberService.findMembers();
    }

    @GetMapping("/api/v2/members")
    public Result membersV2() {
        List<Member> findMembers = memberService.findMembers();
        List<MemberDTO> collect = findMembers.stream()
                .map(m -> new MemberDTO(m.getName()))
                .collect(Collectors.toList());

        return new Result(collect.size(), collect);
    }

    @Data
    @AllArgsConstructor
    static class Result<T> {
        private int count;
        private T data;
    }

    @Data
    @AllArgsConstructor
    static class MemberDTO {
        private String name;
    }
```



## API 개발 고급 - 준비

### API 개발 고급 소개

- 조회: 등록, 수정은 성능 문제가 거의 없다. 조회가 주로 문제가 된다. 샘플 데이터 입력해 테스트
- 지연 로딩과 조회 성능 최적화(N+1문제): 쿼리 1, 2개로 해결될 것이 수십개가 나가는 경우
- 컬렉션 조회 최적화: 조인했는데 데이터가 뻥튀기 되는 경우
- 페이징과 한계 돌파: 페이징 쿼리 작성해서 날릴 때 조인되어 어려워지는 경우
- OSIV와 성능 최적화: osiv로지연로딩이 편하게 되고 안하면 lazy loading exception이 자주 된다. 끄고 켤 때 알아보기



### 조회용 샘플 데이터 입력

InitDb.java

- 데이터 생성을 위한 자바
- @PostConstruct: 스프링 빈 올라오고 나서 호출된다. init에 그냥 넣으면 라이플 사이클 때문에 잘 안됨. 별도 빈으로 등록 필요

```java
package jpabook.jpashop;

import jpabook.jpashop.domain.*;
import jpabook.jpashop.domain.item.Book;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

import javax.annotation.PostConstruct;
import javax.persistence.EntityManager;

@Component
@RequiredArgsConstructor
public class InitDb {

    private final InitService initService;

    @PostConstruct
    public void init() {
        initService.dbInit1();
        initService.dbInit2();
    }

    @Component
    @Transactional
    @RequiredArgsConstructor
    static class InitService {

        private final EntityManager em;
        public void dbInit1() {
            Member member = createMember("userA", "서울", "스트리트", "코드");
            em.persist(member);

            Book book1 = createBook("jpa1", 10000, 100);
            em.persist(book1);

            Book book2 = createBook("jpa2", 20000, 100);
            em.persist(book2);

            OrderItem orderItem1 = OrderItem.createOrderItem(book1, 10000, 1);
            OrderItem orderItem2 = OrderItem.createOrderItem(book2, 20000, 2);

            Delivery delivery = createDelivery(member);
            Order order = Order.createOrder(member, delivery, orderItem1, orderItem2);
            em.persist(order);
        } 
		...
    }
}
```



## API 개발 고급 - 지연 로딩과 조회 성능 최적화

