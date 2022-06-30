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



## 상품 도메인 개발

### 상품 엔티티 개발(비지니스 로직 추가)

Item.java

- add, remove는 재고가 늘어나고 줄어드는 것을 관리하는 비지니스 로직이다. 엔티티 자체가 할 수 있는 비지니스 로직은  setter를 가지고 조작하는게 아니라 엔티티안에 작성해야 응집력이 높고 객체지향적이다.

```java
public abstract class Item {
    @Id
    @GeneratedValue
    @Column(name = "item_id")
    private Long id;
    ...
        
    public void addStock(int quantity) {
        this.stockQuantity += quantity;
    }

    public void removeStock(int quantity) {
        int restStock = this.stockQuantity - quantity;
        if (restStock < 0) {
            throw new NotEnoughStockException("need more stock");
        }
        this.stockQuantity = restStock;
    }
} 
```



exception/NotEnoughStockException

- override 메서드들 전부 그대로 두었다.

```java
package jpabook.jpashop.exception;

public class NotEnoughStockException extends RuntimeException{

    public NotEnoughStockException() {
        super();
    }

    public NotEnoughStockException(String message) {
        super(message);
    }

    public NotEnoughStockException(String message, Throwable cause) {
        super(message, cause);
    }

    public NotEnoughStockException(Throwable cause) {
        super(cause);
    }

    protected NotEnoughStockException(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
        super(message, cause, enableSuppression, writableStackTrace);
    }
}

```



## 주문 도메인 개발

### 주문, 주문상품 엔티티 개발

Order.java

```java
@Entity
@Table(name = "orders")
@Getter @Setter
public class Order {
    ...
    // 생성메서드
    public static Order createOrder(Member member, Delivery delivery, OrderItem... orderItems) {
        Order order = new Order();
        order.setMember(member);
        order.setDelivery(delivery);
        for (OrderItem orderItem : orderItems) {
            order.addOrderItem(orderItem);
        }
        order.setStatus(OrderStatus.ORDER);
        order.setOrderDate(LocalDateTime.now());
        return order;
    }

    // 주문 취소
    public void cancel() {
        if (delivery.getStatus() == DeliveryStatus.COMP) {
            throw new IllegalStateException("이미 배송완료된 상품은 취소가 불가능합니다.");
        }
        this.setStatus(OrderStatus.CANCEL);
        for (OrderItem orderItem : orderItems) {
            orderItem.cancel();
        }
    }

    // 조회 로직
    public int getTotalPrice() {
        int totalPrice = orderItems.stream().mapToInt(OrderItem::getTotalPrice).sum();
        return totalPrice;

//        int totalPrice = 0;
//        for (OrderItem orderItem : orderItems) {
//            totalPrice += orderItem.getTotalPrice();
//        }
//        return totalPrice;
    }
}    
```



OrderItem.java

```java
@Entity
@Getter @Setter
public class OrderItem {
    ...
    // 생성
    public static OrderItem createOrderItem(Item item, int orderPrice, int count) {
        OrderItem orderItem = new OrderItem();
        orderItem.setItem(item);
        orderItem.setOrderPrice(orderPrice);
        orderItem.setCount(count);

        item.removeStock(count);
        return orderItem;
    }

    // 비지니스 로직
    public void cancel() {
        getItem().addStock(count);
    }

    // 조회
    public int getTotalPrice() {
        return getOrderPrice() * getCount();
    }
}
```



### 주문 서비스 개발

OrderService.java

```java
package jpabook.jpashop.service;

@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class OrderService {

    private final OrderRepository orderRepository;
    private final MemberRepository memberRepository;
    private final ItemRepository itemRepository;

    // 주문
    @Transactional
    public Long order(Long memberId, Long itemId, int count) {

        Member member = memberRepository.findOne(memberId);
        Item item = itemRepository.findOne(itemId);

        Delivery delivery = new Delivery();
        delivery.setAddress(member.getAddress());

        OrderItem orderItem = OrderItem.createOrderItem(item, item.getPrice(), count);

        // 여기선 create로 생성하는데 다른 곳에서는 set을 사용하면 유지보수하기 힘들다.
        // 그래서 OrderItem의 기본생성자를 막기 위해 protected로 만들어둔다.
        Order order = Order.createOrder(member, delivery, orderItem);

        // CASCADE 설정이 되어있어 함께 persist. orderItem, delivery
        // cascade 범위는 private owner인 경우? 다른데서 가져다 쓰지 않을 때?
        orderRepository.save(order);

        return order.getId();
    }

    // 취소
    @Transactional
    public void cancelOrder(Long orderId) {
        Order order = orderRepository.findOne(orderId);
        order.cancel();
        // jpa라서 변경 후 update요청 필요x
    }
}
```

OrderItem.java

```java
// set하지 않도록 막는 방법 2개
// 1. lombok사용
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class OrderItem {
    // 2. 기본 생성자 미리 protected로 만들어둬서 밖에서 사용못하게 하기.
    //    protected OrderItem() {}
}
```



- 주문, 취소는 비지니스 로직 대부분이 엔티티에 있고 서비스 계층은 단순히 엔티티에 필요한 요청을 위임하는 역할이다. 이런 핵심 비지니스 로직이 엔티티에 있는 것어 객체 지향 특성을 적극적으로 활용하는 것을 `도메인 모델 패턴`이라고 한다.  반대로 엔티티에 비지니스 로직이 거의 없고 서비스 계층에서 대부분의 비지니스 로직을 처리하는 것을 `트랜잭션 스크립트 패턴`이라고 한다.
  - jpa, orm을 활용하면 전자로 많이 사용하게 된다. 어떤 것이 더 유지보수하기 쉬운지 생각해보고 선택하면 좋다.



### 주문 기능 테스트

OrderServiceTest

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@Transactional
class OrderServiceTest {

    @Autowired EntityManager em;
    @Autowired OrderService orderService;
    @Autowired OrderRepository orderRepository;

    @Test
    public void 상품주문() throws Exception {
        Member member = createMember("member1", new Address("seoutl", "river", "123"));
        Item book = createBook("book title", 10000, 10);
        int orderCount = 2;
        Long orderId = orderService.order(member.getId(), book.getId(), orderCount);
        Order getOrder = orderRepository.findOne(orderId);

        assertEquals(OrderStatus.ORDER, getOrder.getStatus());
        assertEquals(1, getOrder.getOrderItems().size());
        assertEquals(10000 * orderCount, getOrder.getTotalPrice());
        assertEquals(8, book.getStockQuantity());
    }

    @Test
    public void 주문취소() throws Exception {
        Member member = createMember("member1", new Address("seoutl", "river", "123"));
        Item book = createBook("book title", 10000, 10);
        int orderCount = 2;
        Long orderId = orderService.order(member.getId(), book.getId(), orderCount);
        orderService.cancelOrder(orderId);
        Order getOrder = orderRepository.findOne(orderId);

        assertEquals(OrderStatus.CANCEL, getOrder.getStatus());
        assertEquals(10, book.getStockQuantity());
    }

    @Test
    public void 재고수량초과() throws Exception {
        Member member = createMember("member1", new Address("seoutl", "river", "123"));
        Item book = createBook("book title", 10000, 10);
        int orderCount = 12;

        orderService.order(member.getId(), book.getId(), orderCount);
        fail("재고 수량 예외가 발생해야 한다.");

    }

    private Item createBook(String title, int price, int stockQuantity) {
        Item book = new Book();
        book.setName(title);
        book.setPrice(price);
        book.setStockQuantity(stockQuantity);
        em.persist(book);
        return book;
    }

    private Member createMember(String name, Address address) {
        Member member = new Member();
        member.setUsername(name);
        member.setAddress(address);
        em.persist(member);
        return member;
    }
}
```



### 주문 검색 기능 개발

- 회원 이름, 주문 상태 필드를 가진 OrderStatus 클래스를 만든다. OrderRepository에서 findAll로 검색해야 하는데 두 가지 조건을 가지고 있을 때 이 조건들의 null 여부에 따라 동적으로 변하는 쿼리가 필요하다. 쿼리를 스트링으로 만들어서 if 조건에 따라 where를 추가해주거나 JPA Criteria로 해결하는 방법은 지나치게 복잡해 두 가지 다 추천하지 않고 querydsl을 활용해야 한다.



## 웹 계층 개발

### 회원 등록

MemberController.java

- @Valid: 필수값일 때
- BindingResult: 원래는 오류가 있으면 에러 메세지가 보이지만 BindingResult가 있으면 에러를 담고 실행되게 할 수 있다.

```java
@Controller
@RequiredArgsConstructor
public class MemberController {
    private final MemberService memberService;

    @GetMapping("/members/new")
    public String createForm(Model model) {
        model.addAttribute("memberForm", new MemberForm());
        return "members/createMemberForm";
    }

    @PostMapping("/members/new")
    public String createForm(@Valid MemberForm form, BindingResult result) {
        if (result.hasErrors()) {
            return "members/createMemberForm";
        }

        Address address = new Address(form.getCity(), form.getStreet(), form.getZipcode());
        Member member = new Member();
        member.setUsername(form.getName());
        member.setAddress(address);

        memberService.join(member);
        return "redirect:/";
    }
}
```



### 상품 수정

- 수정이 중요

ItemController.java

```java
 @GetMapping("items/{itemId}/edit")
    public String updateItemForm(@PathVariable("itemId") Long itemId, Model model) {
        Book item = (Book) itemService.findOne(itemId);
        BookForm form = new BookForm();
        form.setId(item.getId());
        form.setName(form.getName());
        model.addAttribute("form", form);
        return "items/updateItemForm";
    }

    @PostMapping("items/{itemId}/edit")
    public String updateItem(@ModelAttribute("form") BookForm form) {

        Book book = new Book();
        book.setIsbn(form.getIsbn());
        book.setAuthor(form.getAuthor());

        itemService.saveItem(book);
        return "redirect:items";
    }
```



### 변경감지와 병합

- `매우 중요!!`
- 영속성 컨텍스트에서 엔티티를 다시 조회한 후 데이터를 수정해서 `트랜잭션 커밋 시점에 변경 감지가 동작`해 update 실행
- 병합: 준영속 상태 엔티티를 영속 상태로 변경할 때 사용하는 기능. merge를 호출하면 파라미터로 넘어온 준영속 엔티티 식별자 값으로 1차 캐시에서 엔티티 조회 -> 없으면 db에서 조회해 1차 캐시에 저장. 조회한 영속성 엔티티에 바꾼 값을 채워넣고 반환. 
  - 주의: 변경감지 기능을 사용하면 원하는 속성만 선택해 변경하지만 `병합은 모든 속성이 변경된다.` 병합 시 값이 없으면 null로 업데이트 된다.
- `가급적이면 병합(merge)를 사용하지 말자.`
- 컨트롤러에서 엔티티 생성하지 말기.
- 트랜잭션이 있는 서비스 계층에서 식별자와 변경할 데이터 전달하고 영속 상테 엔티티 조회하고 데이터 직접 변경.