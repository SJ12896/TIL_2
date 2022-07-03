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

- **완전 짱 중요한 내용이라고 함**

### 간단한 주문 조회 V1: 엔티티 직접 노출

OrderSimpleApiController.java

- order 조회하면서 멤버(다대일), 배달(일대일)인 컬렉션 아닌 데이터 가져오는 것 성능 최적화
- 그냥 전체 주문 리스트를 가져와서 return 하면 order에 member필드가 있고 member에도 order 필드가 있어 무한 루프가 생긴다. 둘 중 하나에서 @JsonIgnore가 필요하다.
- 수정해도 에러가 발생하는데 order에서 member는 lazy loading상태기 때문에 db에서 가져오지 않고 byteBuddy라는 proxy라이브러리가 들어있는 상태다. proxy는 나중에 member객체를 db에서 가져올 때 채워진다.(초기화 된다.)
- 이를 막기 위해 지연로딩일 경우는 가져오지 않도록 hibernate 모듈([Jackson Datatype Hibernate5](https://mvnrepository.com/artifact/com.fasterxml.jackson.datatype/jackson-datatype-hibernate5))을 설치해 빈 등록하면 지연 로딩은 null이 된다. 옵션을 통해 강제로 가져오는 것도 가능하다.
- 하지만 다 가져오면 api 스펙 노출 + 필요 없는 데이터를 가져오며 성능 문제가 생긴다.
- 그래서 hibernate모듈로 강제로 가져오지 않고 null로 가져오게 둔 다음 아래 for문을 통해 proxy 객체 대신 진짜 데이터를 가져오도록 아무거나 get을 사용한다.
- 이 과정이 귀찮아 EAGER로 변경하면 결과 같고 쿼리 수도 같지만 (이 경우에만, find같은 경우엔 성능 최적화 안됨?) 성능 튜닝이 매우 어려워지므로 하지 말고 lazy & fetch join을 사용하자.

```java
@RestController
@RequiredArgsConstructor
public class OrderSimpleApiController {
    private final OrderRepository orderRepository;

    @GetMapping("/api/v1/simple-orders")
    public List<Order> ordersV1() {
        List<Order> all = orderRepository.findAllByString(new OrderSearch());
        for (Order order : all) {
            order.getMember().getName(); 
            order.getDelivery().getAddress();
        }
        return all;
    }
}
```



### 간단한 주문 조회 V2: 엔티티를 DTO로 변환

- v1, v2 둘 다 lazy loading으로 db 쿼리 너무 많이 호출.
- dto가 엔티티를 파라미터로 받는 것은 괜찮. 중요하지 않은 곳이니까.
- 처음 order 조회로 sql 1번 -> 결과 주문수 2개 -> 첫 루프 돌 때 order의 member, delivery 조회 -> 두번째 simpeOrderDto에서도 member, delivery가져옴. -> 결국 쿼리 5번 나감. `1 + N + N 번 실행`

```java
import static java.util.stream.Collectors.toList;

@RestController
@RequiredArgsConstructor
public class OrderSimpleApiController {
    private final OrderRepository orderRepository;

    @GetMapping("/api/v2/simple-orders")
    public List<SimpleOrderDto> ordersV2() {
        List<SimpleOrderDto> result =  orderRepository.findAllByString(new OrderSearch()).stream()
                .map(SimpleOrderDto::new)
                .collect(toList());

        return result;
    }

    @Data
    static class SimpleOrderDto {
        private Long orderId;
        private String name;
        private LocalDateTime orderDate;
        private OrderStatus orderStatus;
        private Address address;

        public SimpleOrderDto (Order order) {
            orderId = order.getId();
            name = order.getMember().getName(); // lazy 초기화
            orderDate = order.getOrderDate();
            orderStatus = order.getStatus();
            address = order.getDelivery().getAddress();  // lazy 초기화
        }
    }
}
```



### 간단한 주문 조회 V3: 엔티티를 DTO로 변환 - 페치 조인 최적화

- fetch join으로 가져오기 때문에 쿼리 1번 실행됨.
- join fetch를 사용해서 lazy 무시하고 proxy 아닌 진짜 값 채워서 가져옴. 

```java
// OrderSimpleApiController.java
    @GetMapping("/api/v3/simple-orders")
    public List<SimpleOrderDto> ordersV3() {
        List<Order> orders = orderRepository.findAllWithMemberDelivery();
        List<SimpleOrderDto> result = orders.stream()
                .map(o -> new SimpleOrderDto(o))
                .collect(toList());

        return result;
    }

// OrderRepository.java
    public List<Order> findAllWithMemberDelivery() {
        return em.createQuery(
                "select o from Order o" +
                        " join fetch o.member m" +
                        " join fetch o.delivery d", Order.class
        ).getResultList();
    }
```



### 간단한 주문 조회 V4: JPA에서 DTO로 바로 조회

- v3, v4는 우열을 가릴 수 없다. v3는 외부에서 건드리지 않고 내부에서 튜닝할 수 있고 여러 군데서 활용 가능하다. v4는 sql 짤 때 원하는 것만 select했기 때문에 재사용성이 떨어지고 해당 dto에서만 사용가능하며 작성하는 쿼리가 다소 지저분하다 . 하지만 원하는 데이터만 선택하므로 애플리케이션 네트워크 용량 최적화면에서 더 낫다. 그런데 select를 몇 십개 하는거 아닌 이상 티는 별로 안난다.

- v4실행 결과 쿼리는 from 부터는 같지만 select는 직접 쿼리를 짰기 때문에 줄어들어 select했다.
- OrderRepository.java
  - 원래 controller에 dto가 있었는데 repository쪽에서 사용해야 하기 때문에 controller와 의존 관계가 생기지 않도록 분리. 한 방향으로 흘러가게 한다.
  - JPA는 entity, value object정도만 반환할 수 있고 dto는 안되기 때문에 new를 사용해야하는데 원래 생성자에서 Order를 매개변수로 사용했으나 이제 원하는 필드를 각각 넣도록 변경한다.
  - api 스펙에 맞춘 코드가 repository에 들어가기 때문에 repository 재사용성이 떨어진다.
  - jpql결과를 dto로 즉시 변환
- OrderSimpleQueryRepository.java
  - repository는 순수하게 엔티티를 조회할 때 써야 하는데 OrderRepository에서 위와 같이 dto를 조회하므로 api 스펙이 repository에 들어간다.
  - 따라서 repository에 order.simplequery로 패키지를 새로 만들고 dto를 조회하는 메서드를 분리했다.

```java
    @GetMapping("/api/v4/simple-orders")
    public List<OrderSimpleQueryDto> ordersV4() {
        return orderSimpleQueryRepository.findOrderDtos();
    }


// OrderSimpleQueryDto.java
@Data
public class OrderSimpleQueryDto {
       ...
     public OrderSimpleQueryDto (Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address) {
            this.orderId = orderId;
            this.name = name;
            this.orderDate = orderDate;
            this.orderStatus = orderStatus;
            this.address = address;
        }
}

// OrderRepository.java
    public List<OrderSimpleQueryDto> findOrderDtos() {
        return em.createQuery(
                "select new jpabook.jpashop.repository.OrderSimpleQueryDto(o.id, m.name, o.orderDate, o.status, d.address)" +
                        " from Order o" +
                        " join o.member m" +
                        " join o.delivery d", OrderSimpleQueryDto.class)
                        .getResultList();
    }


// OrderSimpleQueryRepository.java
public class OrderSimpleQueryRepository {
    private final EntityManager em;
    public List<OrderSimpleQueryDto> findOrderDtos() {
        return ...
    }
}
```



- 쿼리 방식 선택 권장 순서
  1. 우선 엔티티를 dto로 변환하는 방법 선택
  2. 필요하면 페치 조인으로 최적화
  3. 안되면 dto로 직접 조회
  4. 최후로 네이티브 sql이나 spring jdbc template을 사용해 sql 직접 사용



## API 개발 고급 - 컬렉션 조회 최적화

- 일대다 관계에서 조인해서 조회하면 데이터가 늘어나게 된다. 이전에는 fetch join으로 다 가져와도 성능에 크게 크게 문제가 없었다. 

### 주문 조회 V1: 엔티티 직접 노출

OrderApiController.java

- 모듈을 통해 지연로딩은 null로 가져올 때

```java
@RestController
@RequiredArgsConstructor
public class OrderApiController {

    private final OrderRepository orderRepository;

    @GetMapping("/api/v1/orders")
    public List<Order> ordersV1() {
        List<Order> all = orderRepository.findAllByString(new OrderSearch());
        for (Order order : all) {
            order.getMember().getName();
            order.getDelivery().getAddress();
            List<OrderItem> orderItems = order.getOrderItems();
            // orderItem 내부의 item들도 초기화
            orderItems.stream().forEach(o -> o.getItem().getName());
        }
        return all;
    }
}
```



### 주문 조회 V2: 엔티티를 DTO로 변환

- Dto에서 @Data를 사용하거나 Data가 만들어주는게 너무 많으면 @Getter를 사용한다.

- dto 안에 entity가 있으면 안된다. entity가 노출된다. 의존을 완전히 끊는게 필요하다. 따라서 private List<OrderItem> orderItem도 전부 dto로 바꿔야 한다. 따라서 OrderItemDto를 만들고 원하는 필드만 가져오도록 한다.
- addresss는 value Object라서 괜찮다.
- 이 방법은 쿼리가 상당히 많이 실행된다.

```java
@GetMapping("/api/v2/orders")
    public List<OrderDto> ordersV2() {

        List<Order> orders = orderRepository.findAllByString(new OrderSearch());
        List<OrderDto> result = orders.stream()
                .map(o -> new OrderDto(o))
                .collect(Collectors.toList());
        return result;
    }

    static class OrderDto {

        private Long orderId;
        private String name;
        private LocalDateTime orderDate;
        private OrderStatus orderStatus;
        private Address address; 
        private List<OrderItemDto> orderItems;

        public OrderDto(Order order) {
            orderId = order.getId();
            name = order.getMember().getName();
            orderDate = order.getOrderDate();
            orderStatus = order.getStatus();
            address = order.getDelivery().getAddress();
            order.getOrderItems().stream().forEach(o -> o.getItem().getName());
            orderItems = order.getOrderItems().stream()
                    .map(orderItem -> new OrderItemDto(orderItem))
                            .collect(Collectors.toList());
        }
    }

    @Getter
    static class OrderItemDto {

        private String itemName;
        private int orderPrice;
        private int count;

        public OrderItemDto(OrderItem orderItem) {
            itemName = orderItem.getItem().getName();
            orderPrice = orderItem.getOrderPrice();
            count = orderItem.getCount();
        }
   
```







