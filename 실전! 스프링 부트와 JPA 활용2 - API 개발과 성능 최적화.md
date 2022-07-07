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



### 주문 조회 V3: 엔티티를 DTO로 변환 - 페치 조인 최적화

- order와 orderitems를 조인하면 2명이 각각 아이템a, 아이템b를 1개씩 구매했기 때문에 orderitems는 4개이므로 이게 기준이 되어 join결과도 4개가 된다.
- distinct를 사용하지 않았을 때 조회된 orders를 확인해보면 entity id까지 같게 나온다. 데이터를 조회하면 한 명의 주문결과 2건이 2번 보인다.
- 이 때문에 select 뒤에 distinct를 사용하게 되는데 이게 들어간 후 실행해서 나온 쿼리문을 직접 db 콘솔에서 실행해도 한 줄이 완전히 다 같지 않고 아이템명 등이 달라 똑같이 4개가 나온다.
- 하지만 jpa에서는 application에 결과를 다 가져온 후 자체적으로 order(entity)의 id값이 같으면 알아서 중복을 제거해주는 기능을 한다.
- 또 페치조인을 사용할 경우 앞 단계와 다르게 sql은 한 번만 실행되지만 **페이징이 불가능**하다. 하이버네이트가 메모리에서 페이징한다고 경고한다. 쿼리에서는 현재 4개가 있기 때문에 우리 기준과 다르다.
- 또 컬렉션 페치 조인은 1개만 사용할 수 있다. 둘 이상 사용하면 데이터가 완전히 뻥튀기 되어  1 * m * n의 상황이 되고 정확성도 떨어진다.

```java
// OrderApiController.java
    @GetMapping("/api/v3/orders")
    public List<OrderDto> ordersV3() {
        List<Order> orders = orderRepository.findAllWithItem();
        List<OrderDto> collect = orders.stream()
                .map(o -> new OrderDto(o))
                .collect(toList());
        return collect;
    }

// OrderRepository.java
    public List<Order> findAllWithItem() {
        return em.createQuery(
                "select distinct o from Order o" +
                " join fetch o.member m" +
                " join fetch o.delivery d" +
                " join fetch o.orderItems oi" +
                " join fetch oi.item i", Order.class)
                .getResultList();
    }
```



### 주문 조회 V3.1: 엔티티를 DTO로 변환 - 페이징과 한계 돌파

- 페이징 + 컬렉션 엔티티를 함께 조회하려면 `ToOne관계를 모두 페치조인`한다. 이는 row수를 증가시키지 않아 페이징 쿼리에 영향을 주지 않는다. 컬렉션은 지연 로딩으로 조회하고 최적화를 위해 hibernate.default_batch_fetch_size(켜두면 좋음), @BatchSize를 적용한다.
  - 쿼리 호출 수가 1 + N에서 1 + 1이 된다.
- hibernate.default_batch_fetch_size를 사용하면 한번에 in을 사용해 한 번에 가져온다. 처음 order가져오는데 주문아이템 2개라 id2개인데 알아서 orders와 관련된 거 한 번에 가져오게된다. 숫자에 따라 in 갯수 지정 가능. v3보단 쿼리수는 증가하지만 v3는 쿼리 결과가 application에 m*n으로 오는데 비해 v3.1은 쿼리에서부터 중복없이 전송량이 감소한다.
- hibernate.default_batch_fetch_size크기는 100~1000사이를 권장한다. DB에 따라 IN절 파라미터가 1000 제한하기도 한다. 1000으로 잡으면 1000개를 DB에서 애플리케이션에 불러오므로 DB에 순간 부하가 증가할 수 있지만 애플리케이션은 결국 전체 데이터를 로딩하므로 메모리 사용량이 같다. 1000이 가장 좋지만 순간 부하를 어디까지 견딜 수 있는지로 결정.

```java
// OrderApiController.java
    @GetMapping("/api/v3.1/orders")
    public List<OrderDto> ordersV3_page(
            @RequestParam(value = "offset", defaultValue = "0") int offset,
            @RequestParam(value = "limit", defaultValue = "100") int limit
            ) {
        List<Order> orders = orderRepository.findAllWithMemberDelivery();

        List<OrderDto> collect = orders.stream()
                .map(o -> new OrderDto(o))
                .collect(toList());
        return collect;
    }

// OrderRepository.java
    public List<Order> findAllWithMemberDelivery(int offset, int limit) {
        return em.createQuery(
                        "select o from Order o" +
                                " join fetch o.member m" +
                                " join fetch o.delivery d", Order.class)
                .setFirstResult(offset)
                .setMaxResults(limit)
                .getResultList();
    }

// application.yml
spring:
  jpa:
    properties:
      default_batch_fetch_size: 1000
          
// Order.java, 위의 default_batch_fetch_size로 글로벌하게 적용하는 대신 디테일하게 사용
@BatchSize(1000)
@OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> orderItems = new ArrayList<>();          
```



### 주문 조회 V4: JPA에서 DTO 직접 조회

- orderRepository는 order entity조회와 관련된 핵심 비지니스
- repository/order/query 쪽에서는 화면, api와 의존관계가 있는 특정 화면에 fit한 내용이 들어있다.
- 코드가 단순하다. 특정 주문 한 건만 조회하면 이 방식을 사용해도 성능이 잘나온다.

OrderApiController.java

```java
    // query는 루트 1번, 컬렉션 n번 실행.  toMany는 별도로 처리
    // toMany는 조인하면 데이터가 증가(최적화하기 어려우므로 findOrderItems 별도 메서드로 조회)

    @GetMapping("/api/v4/orders")
    public List<OrderQueryDto> ordersV4() {
        return orderQueryRepository.findOrderQueryDtos();
    }
```



repository/order/query/OrderQueryRepository.java

- 저번에 만들었던 OrderDto는 orderRepository에 존재하기 때문에 controller에서 참조하면 의존관계 순환이 일어나므로 새로 만들었다.
- findOrders에서 jpql을 짰지만 orderItems같은 컬렉션 데이터를 select해오는건 불가능하기에 빠졌다. 이 때 member, delivery는 toOne관계로 조인해도 데이터가 증가하지 않아 최적화하기 쉬우므로 한 번에 먼저 조회했다.
- 그 다음 orderItems는 그냥 forEach를 통해 result에 채워넣었다.

```java
@Repository
@RequiredArgsConstructor
public class OrderQueryRepository {

    private final EntityManager em;
    
    public List<OrderQueryDto> findOrderQueryDtos() {
        List<OrderQueryDto> result = findOrders();  // 1번 -> n개
        result.forEach(o -> {
            List<OrderItemQueryDto> orderItems = findOrderItems(o.getOrderId());
            o.setOrderItems(orderItems);
        });
        return result;
    }

    private List<OrderItemQueryDto> findOrderItems(Long orderId) {
        return em.createQuery(
                "select new jpabook.jpashop.repository.order.query.OrderItemQueryDto(oi.order.id, i.name, oi.orderPrice, oi.count)" +
                        " from OrderItem oi" +
                        " join oi.item i" +
                        " where oi.order.id = :orderId", OrderItemQueryDto.class)
                .setParameter("orderId", orderId)
                .getResultList();
    }

    private List<OrderQueryDto> findOrders() {
        return em.createQuery(
                        "select new jpabook.jpashop.repository.order.query.OrderQueryDto(o.id, m.name, o.orderDate, o.status, d.address)" +
                                " from Order o" +
                                " join o.member m" +
                                " join o.delivery d", OrderQueryDto.class)
                .getResultList();
    }
}
```



```java
// OrderQueryDto.java, orderItems제외
@Data
public class OrderQueryDto {
    
	...
        
    public OrderQueryDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
    }
}


// OrderItemQueryDto.java
@Data
public class OrderItemQueryDto {

    @JsonIgnore
    private Long orderId;
    private String itemName;
    private int orderPrice;
    private int count;

    public OrderItemQueryDto(Long orderId, String itemName, int orderPrice, int count) {
        this.orderId = orderId;
        this.itemName = itemName;
        this.orderPrice = orderPrice;
        this.count = count;
    }
}

```



### 주문 조회 V5: JPA에서 DTO 직접 조회 - 컬렉션 조회 최적화

- findOrders를 통해 toOne 관계 먼저 조회
- toOrderIds에서 findOrders를 통해 얻은 값에서 orderId를 stream을 통해 가져옴
- findOrderItemMap에서 where의 orderId 조건을 in으로 변경해서 toMany관계인 orderItem 한꺼번에 가져온다.
- Collectors.groupingBy를 사용해 orderId 기준으로 map으로 매칭
- 메모리에 올려둔 map에서 result로 가져오기 때문에 쿼리 2번으로 최적화되고 매칭 성능이 향상.
- v4에 비해 많이 최적화됐지만 코드는 더 복잡하다. 

```java
// OrderApiController.java
    @GetMapping("/api/v5/orders")
    public List<OrderQueryDto> ordersV5() {
        return orderQueryRepository.findAllByDto_optimizaiion();
    }


// orderQueryRepository.java
    public List<OrderQueryDto> findAllByDto_optimizaiion() {
        List<OrderQueryDto> result = findOrders();

        Map<Long, List<OrderItemQueryDto>> orderItemMap = findOrderItemMap(toOrderIds(result));

       result.forEach(o -> o.setOrderItems(orderItemMap.get(o.getOrderId())));
        return result;
    }


    private Map<Long, List<OrderItemQueryDto>> findOrderItemMap(List<Long> orderIds) {
        List<OrderItemQueryDto> orderItems = em.createQuery(
                        "select new jpabook.jpashop.repository.order.query.OrderItemQueryDto(oi.order.id, i.name, oi.orderPrice, oi.count)" +
                                " from OrderItem oi" +
                                " join oi.item i" +
                                " where oi.order.id in :orderIds", OrderItemQueryDto.class)
                .setParameter("orderIds", orderIds)
                .getResultList();

        Map<Long, List<OrderItemQueryDto>> orderItemMap = orderItems.stream().collect(Collectors.groupingBy(OrderItemQueryDto::getOrderId));
        return orderItemMap;
    }


    private List<Long> toOrderIds(List<OrderQueryDto> result) {
        List<Long> orderIds = result.stream()
                .map(o -> o.getOrderId())
                .collect(Collectors.toList());
        return orderIds;
    }

```



### 주문 조회 V6: JPA에서 DTO로 직접 조회, 플랫 데이터 최적화

- OrderFlatDto: OrderQueryDto에서 orderItems를 제외하고 OrderItemQueryDto에서 개별 필드로 가져온다.
- 쿼리 1번으로 데이터를 가져올 수 있지만 데이터가orderItems기준인 m * n으로 온다. 따라서 페이징도 불가능하다. 경우에 따라 쿼리 1번이지만 v5보다 느릴 수 있다.
- 또 return을 QueryDto로 해야한다면 가져와서 루프에서 직접 변경해야 한다. 따라서 애플리케이션에서 추가 작업이 크다.
- collect에서 groupingBy로 묶어줄 객체를 지정할 때 해당 DTO클래스에서 @EqualsAndHashCode(of = "orderId") 애노테이션을 통해 묶을 기준을 지정해야 한다.

```java
// OrderApiController.java

    @GetMapping("/api/v6/orders")
    public List<OrderFlatDto> ordersV6() {
        List<OrderFlatDto> flats = orderQueryRepository.findAllByDto_flat();

        return flats;
    }


// OrderQueryRepository.java
    public List<OrderFlatDto> findAllByDto_flat() {
        return em.createQuery(
                "select new jpabook.jpashop.repository.order.query.OrderFlatDto(o.id, m.name, o.orderDate, o.status, d.address, i.name, oi.orderPrice, oi.count)" +
                        " from Order o" +
                        " join o.member m" +
                        " join o.delivery d" +
                        " join o.orderItems oi" +
                        " join oi.item i", OrderFlatDto.class)
                .getResultList();
    }
```



### API 개발 고급 정리

- 권장순서
  1. 엔티티 조회 방식으로 우선 접근: 페치 조인으로 쿼리 수 최적화 / 컬렉션 최적화
  2. 엔티티 조회 방식으로 해결 안되면 dto조회 방식 사용
  3. nativeSQL, 스프링 JdbcTemplate
- 엔티티 조회는 페치 조인, @batchSize등 코드 수정하지 않고 옵션으로 최적화를 시도할 수 있지만 dto 직접 조회는 성능 최적화하거나 최적화 방식 변경할 때 코드를 변경해야 한다.
- 옵션으로도 최적화 안돼서 캐시하려면 엔티티가 아닌 dto를 캐시해야한다. redis나 local memory cache
- 성능 최적화와 코드 복잡도 사이에서 줄타기. 엔티티는 jpa가 많이 최적화하므로 단순한 코드 + 성능 최적화. dto조회는 sql직접 다루는 것과 유사
- dto방식인 v4, 5, 6도 엔티티 조회 방식으로 하면 훨씬 단순하다. 5번은 batchSize같은걸 쓰면 된다. 쿼리가 1번 실행된다고 좋은 방법인 것은 아니다. 



## API 개발 고급 - 실무 필수 최적화

### OSIV와 성능 최적화

- Open Session In View: 하이버네이트
- Open EntityManager In View: JPA
- JPA에서 em이 하이버네이트에서 session

- 이걸 모르면 장애가 발생할 수 있다. 
- spring boot 애플리케이션을 실행하면 spring.jpa.open-in-view가 기본적으로 enable 어쩌구 하면서 경고가 보인다. OSIV 전략은 최초 DB 커넥션 시작 시점부터 API 응답 끝날 때까지 영속성 컨텍스트, DB 커넥션을 유지한다. 지연 로딩은 영속성 컨텍스트가 살아 있어야하고 이는 DB 커넥션을 유지한다. 이것 자체는 큰 장점이다.
  - 서비스 계층에서 @Transactional 애노테이션을 갖는 메소드는 이 작업이 끝났을 때도 커넥션을 유지하고 있다. OSIV가 이런 역할을 한다.
  - API의 경우엔 유저에게 반환이 될 때 까지, 화면인 경우엔 뷰 템플릿 가지고 렌더링할 때 까지. 유저에게 완전히 response가 나가서 완전히 끝날 때까지.
- 그런데 치명적인 단점이 있는데 너무 오래 db커넥션 리소스를 사용해 실시간 트래픽이 중요한 애플리케이션에서 커넥션이 모잘라 장애가 발생한다. 외부 api를 호출하면 대기 시간만큼 커넥션 리소스를 반환하지 못하고 유지해야 한다.
- 그래서` OSIV를 off`한다면 트랙잭션 종료 시 영속성 컨텍스트를 닫고 db 커넥션도 반환. 그러면 모든 `지연로딩을 트랜잭션 안에서 처리`해야 한다. 지금까지 작성한 지연로딩 코드가 트랜잭션 안으로 들어가야 한다. 지연로딩은 view template에서 동작하지 않아 강제로 호출해 둬야 한다.
  - 끄고 전에 작성했던 코드를 실행해보면 could not initialize proxy 에러가 발생한다.
  - 커맨드와 쿼리 분리: 해결하기 위해 OrderQueryService를 생성해 별도로 제공하는 방법이 있다. OrderService에서 핵심 비지니스 로직을 작성하고 querySvc에서 화면, api에 맞춘 서비스(주로 읽기 전용 트랜잭션)을 작성하게 된다.
  - 보통 서비스 계층에서 트랜잭션을 유지하므로 이를 유지하며 지연 로딩을 사용할 수 있다.
- 강사님은 고객 서비스 실시간 api는 끄고 admin처럼 커넥션 많이 사용x에선 켠다고 한다.



### 스프링 데이터 JPA 소개

- JPA에서 반복하는 코드 자동화. save, find, findAll등은 여기저기서 비슷하다. 
- jpaRepository에 기본 crud가 모두 제공된다. 일반화하기 어려운 기능(findById같은게 아니라 findByName같은 것)도 메서드 이름으로 실행한다. 인터페이스만 만들면 실행시점에 구현체 주입.
- 기존 memberRepository의 파일명을 바꾸고 똑같은 memberRepository파일을 JpaRepository를 상속받아 만들면 거의 대부분이 그대로 동작한다. 다만 findById같은 곳에서 Optional객체로 반환하기 때문에 get을 사용한다. get은 null일 때 에러가 발생하므로 orElseThrow(), orElse()등을 사용할 수도 있다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    // select m from Member m where m.name = ? 으로 알아서 짜준다.
    List<Member> findByName(String name);
}


// MemberService.java
    public Member findOne(Long memberId) {
        return memberRepository.findById(memberId).get();
    }
```



### QueryDSL 소개

- build.gradle에서 plugin, dependencies등 여러개 추가 후 q파일을 만들기 위해 tasks/other에 compileQuerydsl실행하고 나면 src/main/generated에 q파일들이 생성되어 있다. 이 파일들은 git에 커밋하면 안된다.
- 100퍼센트 자바 코드라 오타 내도 컴파일 시점에 바로 알려주기 때문에 좋다.
- where에 있는 메서드에서 null을 반환하면 사용하지 않는다.

```java
// OrderRepository.java
    public List<Order> findAll(OrderSearch orderSearch) {
        
        // static import로 하고 지워도 됨.
        Qorder order = Qorder.order;
        QMember member = QMember.member;

        JPAQueryFactory query = new JpaQueryFactory(em);
        return query.select(order)
                .from(order)
                .join(order.member, member)
                .where(statusEq(orderSearch.getOrderStatus()), nameLike(orderSearch.getMemberName()))
                .limit(1000)
                .fetch();
    }

    private BooleanExpression statusEq(OrderStatus statusCond) {
        if (statusCond == null) {
            return null;
        }
        return Qorder.order.status.eq(statusCond);
    }

    private BooleanExpression nameLike(String memberName) {
        if (!StringUtils.hasText(memberName)) {
            return null;
        }
        return QMember.member.name.like(orderSearch.getMemberName());
    }
```





