# 섹션 2. 스프링 핵심 원리 이해1 - 예제 만들기

## 프로젝트 생성

- [start.spring.io](http://start.spring.io)에서 설정
- Project : Gradle Project
- Language : Java
- Spring Boot : 2.7.3(stable 버전 중 최신 버전 선택)
- Group : hello
- Artifact : core

- `build.gradle` 파일 변경 후에는 돌고래 버튼 또는 오른쪽 Gradle 메뉴 확장해서 Reload 눌러줘야 적용됨
- Settings - Build, Execution, Deployment - Build Tools - Gradle에서 Build and run using, Run tests using을 모두 IntelliJ IDEA로 바꿔주면 더 빠르게 실행 가능
    - 실행시 출력되는 prompt도 달라짐

> 프로젝트 환경 설정을 쉽게 하기 위해 spring boot를 사용했고, spring과 관련된 코드는 사용하지 않음
> 

## 비즈니스 요구사항과 설계

- 회원
    - 회원 가입 및 조회 기능
    - 등급 : 일반, VIP
    - 회원 DB는 자체 DB 또는 외부 시스템과 연동 가능(미확정)
- 주문과 할인 정책
    - 회원은 상품 주문 가능
    - 회원 등급에 따라 할인 정책이 적용됨
    - 할인 정책 : VIP는 모두 1000원 고정 금액 할인
        - 기본 할인 정책은 오픈 직전까지 미확정(적용하지 않을 수도 있음)
- interface를 만들고 구현체를 바꿀 수 있도록 설계하면 미확정인 부분 해결 가능

## 회원 도메인 설계

- 회원 도메인 요구사항
    - 회원 가입 및 조회
    - 일반, VIP 등급
    - 회원 DB는 자체 DB 또는 외부 시스템 연동 가능(미확정)

**회원 도메인 협력 관계**

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-2%20(1).png)

- 기획자들도 볼 수 있음

**회원 클래스 다이어그램**

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-2%20(2).png)

- 구체화되어 interface, implement가 표현됨

**회원 객체 다이어그램**

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-2%20(3).png)

- instance 간에 참조가 어떻게 되는지 표현

## 회원 도메인 개발

`hello.core/member`에 아래 파일들 생성

### 회원 엔티티

- enum `Grade`
    - `BASIC, VIP` 넣어줌
- class `Member`
    - `Long id`  
    `String name`  
    `Grade grade` 정의
    - constructor, getter and setter 생성

### 회원 저장소

- interface `MemberRepository`
    - `void save(Member member)`, `Member findById(Long memberId)` 정의
- class `MemoryMemberRepository`

```java
public class MemoryMemberRepository implements MemberRepository {

    private static Map<Long, Member> store = new HashMap<>();

    @Override
    public void save(Member member) {
        store.put(member.getId(), member);
    }

    @Override
    public Member findById(Long memberId) {
        return store.get(memberId);
    }
}
```

- 동시성 이슈가 있을 수 있기 때문에 원래는 `ConcurrentHashMap`을 사용해야 함

### 회원 서비스

- interface `MemberService`
    - `void join(Member member)`, `Member findMember(Long memberId)` 정의
- class `MemberServiceImpl`

```java
public class MemberServiceImpl implements MemberService {
    private final MemberRepository memberRepository = new MemoryMemberRepository();

    @Override
    public void join(Member member) {
        memberRepository.save(member);
    }

    @Override
    public Member findMember(Long memberId) {
        return memberRepository.findById(memberId);
    }
}
```

- 구현체가 하나일 경우 interface 이름 뒤에 Impl 붙여서 명명

## 회원 도메인 실행과 테스트

- `hello.core`에 `MemberApp` 클래스 생성

```java
public class MemberApp {
    public static void main(String[] args) {
        MemberService memberService = new MemberServiceImpl();
        Member member = new Member(1L, "memberA", Grade.VIP);
        memberService.join(member);

        Member findMember = memberService.findMember(1L);
        System.out.println("member = " + member.getName());
        System.out.println("findMember = " + findMember.getName());
    }
}
```

- `psvm` : main 함수 단축키
- `soutv` : 변수 출력 단축키

- `test/../hello.core`에 `member` 패키지 생성 후 `MemberServiceTest` 클래스 생성

```java
public class MemberServiceTest {

    MemberService memberService = new MemberServiceImpl();

    @Test
    void join() {
        //given
        Member member = new Member(1L, "memberA", Grade.VIP);

        //when
        memberService.join(member);
        Member findMember=memberService.findMember(1L);

        //then
        Assertions.assertThat(member).isEqualTo(findMember);
    }
}
```

### 회원 도메인 설계의 문제점

- OCP, DIP를 잘 지키고 있을까?
    
    ```java
    private final MemberRepository memberRepository = new MemoryMemberRepository();
    ```
    
    - `MemberServiceImpl`의 변수 선언은 추상화, 구체화 모두 의존 → DIP 위반

## 주문과 할인 도메인 설계

- 주문과 할인 정책
    - 회원이 상품을 주문
    - 할인 정책 : VIP에 대해 고정 금액 할인(미확정, 추후 변경 가능)

### 주문 도메인 협력, 역할, 책임

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-2%20(4).png)

1. 주문 생성 : client가 주문 서비스에 주문 생성 요청
2. 회원 조회 : `findById` 메소드로 회원 정보 조회
3. 할인 적용 : 주문 서비스가 할인 여부를 할인 정책 역할에 위임함
4. 주문 결과 반환 : 주문 서비스가 할인 결과를 포함한 주문 결과를 반환
    - 실제로는 주문 데이터를 DB에 저장해야 하지만, 예제이기 때문에 반환만 함

### 주문 도메인 설계

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-2%20(5).png)

- 역할과 구현을 분리해서 설계함
    - 구현 객체를 자유롭게 조립할 수 있음
    - 회원 저장소, 할인 정책을 유연하게 변경할 수 있음

### 주문 도메인 클래스 다이어그램

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-2%20(6).png)

### 주문 도메인 객체 다이어그램 예시

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-2%20(7).png)
![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-2%20(8).png)

- interface들의 협력 관계를 그댇로 재사용할 수 있음

## 주문과 할인 도메인 개발

`hello.core`에 `discount` 패키지 생성 후 아래 파일들 구현

- interface `DiscountPolicy`

```java
public interface DiscountPolicy {

    /**
     * @return 할인 대상 금액
     */
    int discount(Member member, int price);
}
```

- class `FixDiscountPolicy`

```java
public class FixDiscountPolicy implements DiscountPolicy {

    private int discountFixAmount = 1000;

    @Override
    public int discount(Member member, int price) {
        if (member.getGrade() == Grade.VIP) {
            return discountFixAmount;
        } else {
            return 0;
        }
    }
}
```

`hello.core`에 `order` 패키지 생성 후 아래 파일들 구현

- class `Order`
    - constructor, getter and setter 정의

```java
public class Order {

    private Long memberId;
    private String itemName;
    private int itemPrice;
    private int discountPrice;

		@Override
    public String toString() {
        return "Order{" +
                "memberId=" + memberId +
                ", itemName='" + itemName + '\'' +
                ", itemPrice=" + itemPrice +
                ", discountPrice=" + discountPrice +
                '}';
    }
}
```

- interface `OrderService`

```java
public interface OrderService {
    Order createOrder(Long memberId, String itemName, int itemPrice);
}
```

- class `OrderServiceImpl`

```java
public class OrderServiceImpl implements OrderService {

    private final MemberRepository memberRepository = new MemoryMemberRepository();
    private final DiscountPolicy discountPolicy = new FixDiscountPolicy();

    @Override
    public Order createOrder(Long memberId, String itemName, int itemPrice) {
        Member member = memberRepository.findById(memberId);
        int discountPrice = discountPolicy.discount(member, itemPrice);

        return new Order(memberId, itemName, itemPrice, discountPrice);
    }
}
```

- 할인에 관한 내용은 `discountPolicy`에서 알아서 해줌
    - SRP(단일 책임 원칙)을 잘 지킨 예시

## 주문과 할인 도메인 실행과 테스트

- `hello.core`에 class `OrderApp` 생성

```java
public class OrderApp {

    public static void main(String[] args) {
        MemberService memberService = new MemberServiceImpl();
        OrderService orderService = new OrderServiceImpl();

        Long memberId = 1L;
        Member member = new Member(memberId, "memberA", Grade.VIP);
        memberService.join(member);

        Order order = orderService.createOrder(memberId, "itemA", 10000);

        System.out.println("order = " + order);
    }
}
```

- 메인 코드에서 테스트

- `test/../hello.core`에 `order` 패키지 생성 후 `OrderServiceTest` 클래스 생성

```java
public class OrderServiceTest {

    MemberService memberService = new MemberServiceImpl();
    OrderService orderService = new OrderServiceImpl();

    @Test
    void createOrder() {
        Long memberId = 1L;
        Member member = new Member(memberId, "memberA", Grade.VIP);
        memberService.join(member);

        Order order = orderService.createOrder(memberId, "itemA", 10000);
        Assertions.assertThat(order.getDiscountPrice()).isEqualTo(1000);
    }
}
```

- `memberId`를 primitive로 정의하면 `null`을 넣을 수 없음