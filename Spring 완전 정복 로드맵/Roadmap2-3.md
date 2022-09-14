# 섹션 3. 스프링 핵심 원리 이해2 - 객체 지향 원리 적용

## 새로운 할인 정책 개발

- 고정 금액 할인보다 주문 금액당 할인하는 정률 % 할인으로 변경(e.g. 10%)
- RateDiscountPolicy 추가
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-3%20(1).png)
    
- `hello.core.discount`에 `RateDiscountPolicy` 클래스 추가

```java
public class RateDiscountPolicy implements DiscountPolicy {

    private int discountPercent = 10;

    @Override
    public int discount(Member member, int price) {
        if (member.getGrade() == Grade.VIP) {
            return price * discountPercent / 100;
        } else {
            return 0;
        }
    }
}
```

- `Ctrl + Shift + T` : 테스트 생성
    - `discount` 메소드에 커서두고 테스트 생성

```java
class RateDiscountPolicyTest {

    RateDiscountPolicy discountPolicy = new RateDiscountPolicy();

    @Test
    @DisplayName("VIP는 10% 할인이 적용되어야 한다")
    void vip_o() {
        //given
        Member member = new Member(1L, "memberVIP", Grade.VIP);
        //when
        int discount = discountPolicy.discount(member, 10000);
        //then
        assertThat(discount).isEqualTo(1000);
    }

    @Test
    @DisplayName("VIP가 아니면 할인이 적용되지 않아야 한다")
    void vip_x() {
        //given
        Member member = new Member(2L, "memberVIP", Grade.BASIC);
        //when
        int discount = discountPolicy.discount(member, 10000);
        //then
        assertThat(discount).isEqualTo(0);
    }
}
```

- `Assertions`는 `assertj`꺼 써야함
    - `alt + Enter`해서 on-demand static import 해놓으면 좋음
- JUnit5부터 `@DisplayName()` 지원됨(테스트 이름 설정 가능)

## 새로운 할인 정책 적용과 문제점

- `OrderServiceImpl`의 discountPolicy의 객체를 `RateDiscountPolicy`로 생성하면 적용됨

```java
public class OrderServiceImpl implements OrderService {

    private final MemberRepository memberRepository = new MemoryMemberRepository();
//    private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
    private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
...
}
```

- 역할과 구현을 분리해서 설계하고 구현함
- OCP, DIP를 준수한 것 같지만, 아님
    - DIP : 클래스 의존관계를 보면 interface 뿐만 아니라 implements(`FixDiscountPolicy`, `RateDiscountPolicy`)에도 의존함
    - OCP : 현재 코드는 기능 확장 후 변경할 때 client 코드에도 영향을 줌(OCP 위반)
- 기대했던 의존 관계
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-3%20(2).png)
    
    - `OrderServiceImpl`이 `DiscountPolicy` 인터페이스만 의존한다고 생각하먀
- 실제 의존 관계
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-3%20(3).png)
    
    - `OrderServiceImpl`은 `FixDiscountPolicy` implements에도 의존하고 있음(DIP 위반)
- 정책 변경할 때
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-3%20(4).png)
    
    - `OrderServiceImpl`의 코드도 함께 변경해야 함(OCP 위반)

### 문제 해결

- interface에만 의존하도록 의존관계를 변경해야 함(DIP를 위반하지 않도록)

```java
public class OrderServiceImpl implements OrderService {

    private final MemberRepository memberRepository = new MemoryMemberRepository();
    private DiscountPolicy discountPolicy;
...
}
```

- 구현체가 없는데 어떻게 실행될까?
    - NPE(Null Pointer Exception)이 발생함
    - 누군가 client인 `OrderServiceImpl`에 `DiscountPolicy`의 구현 객체를 대신 생성하고 주입해줘야 함

## 관심사의 분리

- 이전 코드는 `OrderServiceImpl`이라는 구현체가 `DiscountPolicy` 인터페이스의 구현체를 직접 선택하는 것임 → 다양한 책임을 가지게 됨

### AppConfig 등장

- app의 전체 동작 방식을 구성(config)하기 위해 **구현 객체를 생성하고 연결하는 책임**을 가지는 별도의 설정 클래스를 만듦
- `hello.core`에 `AppConfig` 클래스 생성

```java
public class AppConfig {

    public MemberService memberService() {
        return new MemberServiceImpl(new MemoryMemberRepository());
    }

    public OrderService orderService() {
        return new OrderServiceImpl(new MemoryMemberRepository(), new FixDiscountPolicy());
    }
}
```

- `AppConfig`는 앱 실행에 필요한 구현 객체를 생성
    - `MemberServiceImpl`, `MemoryMemberRepository`, `OrderServiceImpl`, `FixDiscountPolicy`
- 생성한 인스턴스의 reference는 생성자를 통해서 주입함
    - `MemberServiceImpl` → `MemoryMemberRepository`
    - `OrderServiceImpl` → `MemoryMemberRepository`, `FixDiscountPolicy`

> Q. 해시맵이 각각 따로 생성되는거 아닌가? → 맞음, spring bean으로 만들면 이런 중복이 없어지는 듯
> 
- `MemberServiceImpl`, `OrderServiceImpl` 둘 다 생성자를 만들어줘야 함
    
    ```java
    // MemberServiceImpl
    public class MemberServiceImpl implements MemberService {
        private final MemberRepository memberRepository;
    
        public MemberServiceImpl(MemberRepository memberRepository) {
            this.memberRepository = memberRepository;
        }
    ...
    }
    
    // OrderServiceImpl
    public class OrderServiceImpl implements OrderService {
    
        private final MemberRepository memberRepository;
        private final DiscountPolicy discountPolicy;
    
        public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
            this.memberRepository = memberRepository;
            this.discountPolicy = discountPolicy;
        }
    ...
    }
    ```
    

### 클래스 다이어그램

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-3%20(5).png)

- `MemberServiceImpl`은 `MemberRepository`에만 의존함 → DIP 준수
- `OrderServiceImpl`은 `MemberRepository`, `DiscountPolicy`에만 의존함 → DIP 준수
- 관심사를 분리함(객체 생성/연결, 실행이 분리됨)

### 회원 객체 instance 다이어그램

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-3%20(6).png)

- client인 `memberServiceImpl`의 의존관계를 외부에서 주입해줌(Dependency Injection)

### `AppConfig` 실행

- `MemberApp`, `OrderApp`에 적용시켜야 함

```java
// MemberApp
public class MemberApp {
    public static void main(String[] args) {
        AppConfig appConfig = new AppConfig();
        MemberService memberService = appConfig.memberService();
...
}}

// OrderApp
public class OrderApp {
    public static void main(String[] args) {
        AppConfig appConfig = new AppConfig();
        MemberService memberService = appConfig.memberService();
        OrderService orderService = appConfig.orderService();
...
}}
```

- 테스트코드들도 수정해야 함

```java
// MemberServiceTest
public class MemberServiceTest {

    MemberService memberService;

    @BeforeEach
    public void beforeEach() {
        AppConfig appConfig = new AppConfig();
        memberService = appConfig.memberService();
    }

// OrderServiceTest
public class OrderServiceTest {

    MemberService memberService;
    OrderService orderService;

    @BeforeEach
    public void beforeEach() {
        AppConfig appConfig = new AppConfig();
        memberService= appConfig.memberService();
        orderService= appConfig.orderService();
    }
```

- `@BeforeEach`는 각 테스트를 실행하기 전에 매번 호출됨

## AppConfig 리팩터링

- 현재 `AppConfig`에는 중복이 존재하고, 역할에 따른 구현이 잘 보이지 않음
- 기대하는 그림
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-3%20(7).png)
    
- `AppConfig` 리팩터링 후

```java
public class AppConfig {

    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    private MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    public DiscountPolicy discountPolicy() {
        return new FixDiscountPolicy();
    }
}
```

- interface, implements가 다 드러남
- `new MemoryMemberRepository()` 부분의 중복이 제거됨 → 변경할 때 편리함

## 새로운 구조와 할인 정책 적용

- `FixDiscountPolicy` → `RateDiscountPolicy`로 바꿔보자
- `AppConfig`만 변경하면 됨
    - `AppConfig`의 등장으로 앱이 사용 영역과 구성 영역으로 분리됨

### 사용, 구성 영역

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-3%20(8).png)

- 할인 정책의 변경

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-3%20(9).png)

- `AppConfig`에서 `discountPolicy()`의 생성자만 `RateDiscountPolicy()`로 바꿔주면 됨

```java
public DiscountPolicy discountPolicy() {
    return new RateDiscountPolicy();
}
```

- client 코드인 `OrderServiceImpl`과 같은 사용 영역의 어떠한 코드도 변경할 필요가 없음
- `AppConfig`와 같은 구성 영역은 당연히 변경됨
- 추상화에 의존 → DIP 만족
- 기능 확장해도 client의 코드를 변경할 필요가 없음 → OCP 만족

## 전체 흐름 정리

- 정률 할인 적용을 위해서 `OrderServiceImpl`의 구현체를 변경해야 했음
    - DIP 위반
- 관심사 분리
    - `AppConfig`를 통해 구현 객체를 생성하고 interface와 연결함 → DIP 준수
    - client 객체는 로직을 실행하는 것에만 집중, 권한이 줄어듦
- `AppConfig` 리팩터링
    - interface와 implements를 분리
    - 중복 제거
- 새로운 구조와 할인 정책 적용할 때 `AppConfig`가 있는 구성 영역만 변경하면 됨 → OCP 준수

## 좋은 객체 지향 설계의 5가지 원칙의 적용

- 이 코드에서는 SRP, DIP, OCP 적용

### SRP(단일 책임 원칙)

- 한 클래스는 하나의 책임만 가져야 한다.
- 관심사 분리
    - 구현 객체 생성 및 연결 : `AppConfig`가 담당
    - 실행 : client 객체가 담당

### DIP(의존관계 역전 원칙)

- 프로그래머는 “추상화에 의존해야지, 구체화에 의존하면 안된다.”
    - 의존성 주입은 이 원칙을 따르는 방법 중 하나임
- 기존에는 해로운 할인 정책을 적용할 때 client 코드도 변경해야 함
- client 코드가 `DiscountPolicy` interface에만 의존하도록 코드를 변경함
    - interface만으로는 아무것도 실행할 수 없음(NPE 발생)
    - `AppConfig`가 `FixDiscountPolicy` 객체 인스턴스를 client 코드 대신 생성해서 의존관계 주입 → DIP 준수하면서 문제 해결

### OCP

- 소프트웨어 요소는 확장에는 열려 있으나 변경에는 닫혀 있어야 한다.
- 앱을 사용 영역과 구성 영역으로 나눔
    - `AppConfig`가 의존관계를 `RateDiscountPolicy`로 변경해서 client 코드에 주입하기 때문에 client 코드는 변경하지 않아도 됨
    - 새롭게 확장해도 사용 영역의 변경은 닫혀 있음

## IoC, DI, 그리고 컨테이너

### IoC(Inversion of Control, 제어의 역전)

- client 구현 객체가 스스로 필요한 구현 객체를 생성, 연결, 실행함
    - 구현 객체가 프로그램의 제어 흐름을 스스로 조종함  
    개발자 입장에서는 자연스러운 흐름
- `AppConfig`가 등장하면서 구현 객체는 자신의 로직을 실행하는 역할만 담당
    - 구현 객체가 뭔지 모름
    - 프로그램 제어 흐름에 대한 권한은 `AppConfig`가 가지고 있음
    - IoC : 프로그램의 제어 흐름을 외부(`AppConfig`)에서 관리하는 것을 의미함

### 프레임워크 vs 라이브러리

- 프레임워크 : 내가 작성한 코드를 제어하고 대신 실행함(e.g. JUnit)
    - `@Test`, `@BeforeEach`와 같은 annotation을 붙여서 로직만 설계하면 알아서 제어해서 실행
- 라이브러리 : 내가 작성한 코드가 직접 제어 흐름을 담당함
    - 객체를 xml이나 json으로 바꾸는 라이브러리를 직접 호출해서 원하는 타이밍에 바꿈

### DI(Dependency Injection, 의존관계 주입)

- `OrderServiceImpl`은 `DiscountPolicy` interface에 의존하는 것이 확실하지만, 실제 어떤 implements가 사용될지는 모름
- 정적인 **클래스 의존 관계**와 동적인 **인스턴스(객체) 의존 관계**를 분리해서 생각해야 함

**클래스 의존 관계**

- 실행하지 않고 코드만 보고 의존관계를 쉽게 판단 가능
- 클래스 다이어그램
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-3%20(10).png)
    
    - `OrderServiceImpl`은 `MemberRepository`, `DiscountPolicy`에 의존함
    - `OrderServiceImpl`에 어떤 객체가 주입될 지는 알 수 없음

**객체 의존 관계**

- 런타임에 실제 생성된 객체의 reference가 연결된 의존 관계
- 객체 다이어그램
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-3%20(11).png)
    
    - 의존관계 주입 : 런타임에 외부에서 구현 객체를 생성하고 client에 전달해서 서버와의 실제 의존 관계가 연결되는 것
        - client 코드(클래스 의존관계)를 변경하지 않고 client가 호출하는 대상의 instance를 변경할 수 있음

### IoC 컨테이너, DI 컨테이너

- DI 컨테이너(IoC 컨테이너) : `AppConfig`와 같이 객체를 생성, 관리하면서 의존관계를 연결해주는 객체
    - IoC 컨테이너는 범용적이고, *의존관계 주입*에 초점을 맞춰 DI 컨테이너가 주로 사용됨
    - 어셈블러, 오브젝트 팩토리 등으로도 불림

## 스프링으로 전환하기

- 현재까지는 순수한 자바 코드로 DI를 적용했음
- `AppConfig` 스프링 기반으로 변경
    
    ```java
    @Configuration
    public class AppConfig {
    
        @Bean
        public MemberService memberService() {
            return new MemberServiceImpl(memberRepository());
        }
    
        @Bean
        public MemberRepository memberRepository() {
            return new MemoryMemberRepository();
        }
    
        @Bean
        public OrderService orderService() {
            return new OrderServiceImpl(memberRepository(), discountPolicy());
        }
    
        @Bean
        public DiscountPolicy discountPolicy() {
            return new RateDiscountPolicy();
        }
    }
    ```
    
    - `@Configuration`을 붙임
    - 각 메소드에 `@Bean`을 붙이면 spring container에서 spring bean으로 등록해줌
- `MemberApp` 수정
    
    ```java
    public class MemberApp {
        public static void main(String[] args) {
    //        AppConfig appConfig = new AppConfig();
    //        MemberService memberService = appConfig.memberService();
            ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
            MemberService memberService = applicationContext.getBean("memberService", MemberService.class);
    ...
    }}
    ```
    
- `OrderApp` 수정
    
    ```java
    public class OrderApp {
        public static void main(String[] args) {
    //        AppConfig appConfig = new AppConfig();
    //        MemberService memberService = appConfig.memberService();
    //        OrderService orderService = appConfig.orderService();
    
            ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
            MemberService memberService = applicationContext.getBean("memberService", MemberService.class);
            OrderService orderService = applicationContext.getBean("orderService", OrderService.class);
    ...
    }}
    ```
    

### 스프링 컨테이너

- `ApplicationContext`를 Spring Container라고 함
    - 기존에는 `AppConfig`를 사용해서 직접 DI를 했지만, 이제는 spring container를 사용함
- spring container는 `@Configuration`이 붙은 `AppConfig`를 설정 정보로 사용함
    - `@Bean`이 붙은 메소드를 모두 호출해서 반환된 객체들을 spring container에 등록함  
    → 등록된 객체들이 spring bean이라고 불림
    - spring bean의 이름은 `@Bean`이 붙은 메소드 이름으로 설정됨
        - `@Bean(name = "sss")`로 변경 가능
    - `appConfig.memberService` 대신 `applicataionContext.getBean("memberService", memberService.class)`으로 객체를 찾을 수 있음