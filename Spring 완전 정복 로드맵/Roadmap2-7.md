# 섹션 7. 의존관계 자동 주입

## 다양한 의존관계 주입 방법

### 생성자 주입

- `final` 같은 불변이고 필수적인 의존 관계에 사용함
- 생성자 호출 시점에 단 1번만 호출되는 것이 보장됨
- 생성자가 하나만 선언되었을 경우 `@Autowired`를 생략해도 자동 주입됨
    - 의존 관계 자동 주입은 spring bean으로 등록된 클래스에서 이루어지는 것임

### 수정자 주입

- 선택적이고 변경 가능성이 있는 의존 관계에 사용함
- setter에 `@Autowired`를 사용해서 의존 관계를 주입함
- `@Autowired`는 주입할 대상이 없으면 오류를 발생시킴
    - `@Autowired(required = false)`로 오류 무시 가능

### 필드 주입

- 필드에 `@Autowired`를 바로 추가함
- setter를 만들지 않기 때문에 DI 프레임워크를 사용하지 않으면 가용성이 너무 떨어짐
    - 필드는 보통 `private`이므로 테스트할 때 DI를 해주려면 setter를 추가해야 함
- 앱 구현과 상관 없는 테스트 코드나 config 코드에서는 사용해도 문제 없음
    - `@Autowired`를 사용하려면 `@SpringBootTest`와 같은 스프링이 제공하는 테스트를 사용해야 함

### 일반 메서드 주입

- 일반 메소드를 사용에도 `@Autowired`를 사용 가능하긴 함
    - 생성자, setter로 거의 다 커버 가능함

## 옵션 처리

- 주입할 spring bean이 없어도 동작해야 할 때 아래와 같은 방법으로 처리 가능
- `@Autowired(required = false)` : 주입할 bean이 없으면 수정자 메소드 자체가 호출이 안됨
- `org.springframework.lang.Nullable` : 주입할 bean이 없으면 `null`이 대신 주입됨
- `Optional<>` : 주입할 bean이 없으면 `Optional.empty`가 주입됨
- `test/../autowired/AutowiredTest.java` 생성
    
    ```java
    public class AutowiredTest {
    
        @Test
        void AutowiredOption(){
            ApplicationContext ac = new AnnotationConfigApplicationContext(TestBean.class);
        }
    
        static class TestBean {
            @Autowired(required = false)
            public void setNoBean1(Member noBean1) {
                System.out.println("noBean1 = " + noBean1);
            }
    
            @Autowired
            public void setNoBean2(@Nullable Member noBean2) {
                System.out.println("noBean2 = " + noBean2);
            }
    
            @Autowired
            public void setNoBean3(Optional<Member> noBean3) {
                System.out.println("noBean3 = " + noBean3);
            }
        }
    }
    ```
    
    - 결과
    
    ```
    noBean3 = Optional.empty
    noBean2 = null
    ```
    
    - `Member` 객체 타입인 빈이 컨테이너에 없는 상태임
    - `setNoBean1()`은 `required = false`이기 때문에 호출되지 않음

## 생성자 주입을 선택해라

- 대부분의 의존관계는 한번 설정된 후 종료 시점까지 불변해야 함
    - 수정자 주입에서는 setter 메소드가 public이기 때문에 임의로 실행해서 의존관계 변경 가능
- 테스트 코드를 작성할 때
    - 생성자 주입으로 구현할 경우 필드에 `final` 키워드를 붙일 수 있음
        
        ```java
        @Component
        public class OrderServiceImpl implements OrderService {
        
            private final MemberRepository memberRepository;
            private final DiscountPolicy discountPolicy;
        
            @Autowired
            public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
                this.memberRepository = memberRepository;
                this.discountPolicy = discountPolicy;
            }
        ...
        }
        ```
        
    - 생성자 호출 단계에서 의존관계 주입
    - 생성자 내부에서 의존 관계를 주입해주지 않을 경우 컴파일 오류로 명시됨
        - DI 프레임워크를 사용하지 않고 순수 자바 테스트 코드를 작성할 때도 편리함
    - 수정자 주입은 인스턴스 생성 후 따로 setter를 실행해야함
- `test/../order/OrderServiceImplTest.java` 생성
    
    ```java
    class OrderServiceImplTest {
        @Test
        void createOrder() {
            MemoryMemberRepository memberRepository = new MemoryMemberRepository();
            memberRepository.save(new Member(1L, "name", Grade.VIP));
    
            OrderServiceImpl orderService = new OrderServiceImpl(memberRepository, new FixDiscountPolicy());
            Order order = orderService.createOrder(1L, "itemA", 10000);
            Assertions.assertThat(order.getDiscountPrice()).isEqualTo(1000);
        }
    }
    ```
    

## 롬복과 최신 트렌드

- `build.gradle`에 설정 추가
    
    ```java
    ...
    configurations {
    	compileOnly {
    		extendsFrom annotationProcessor
    	}
    }
    
    dependencies {
    	compileOnly 'org.projectlombok:lombok'
    	annotationProcessor 'org.projectlombok:lombok'
    	testCompileOnly 'org.projectlombok:lombok'
    	testAnnotationProcessor 'org.projectlombok:lombok'
    
    	implementation 'org.springframework.boot:spring-boot-starter'
    	testImplementation 'org.springframework.boot:spring-boot-starter-test'
    }
    ...
    ```
    
    - 이후 annotation processing enable 설정
- `OrderServiceImpl.java`에 `@RequiredArgsConstructor` 추가 후 기존 생성자 삭제
    - `@RequiredArgsConstructor` : `final`이 붙은 필드들만 파라미터로 받는 생성자 추가
    - 위 어노테이션이 기존 생성자와 동일한 생성자를 만들어주기 때문에 이렇게 만들어진 생성자만 남겨서 `@Autowired` 없이 의존 관계가 자동 주입되도록 설정

## 조회 빈이 2개 이상일 경우

- `@Autowired`는 `ac.getBean(DiscountPolicy.class)`와 비슷하게 타입으로 조회함
- `FixDiscountPolicy.java`도 `@Component`로 등록
    - `basicScan()` 테스트가 `NoUniqueBeanDefinitionException` 예외가 발생하며 실패하게 됨
    - spring bean을 수동 등록해도 되지만, DI 자동 주입에서 해결하는 방법들도 존재함

## `@Autowired` 필드 명, `@Qualifier`, `@Primary`

### `@Autowired` 필드 명 매칭

- `@Autowired`의 동작 과정
    1. 타입 매칭
    2. 필드 이름, 파라미터 이름 매칭
    3. `NoSuchBeanDefinitionException` 예외 발생

### `@Qualifier` 사용

- `@Qualifier`의 동작 과정
    1. `@Qualifier` 끼리 매칭
    2. 빈 이름과 매칭
    3. `NoSuchBeanDefinitionException` 예외 발생
- DI할 때 구분하는 옵션을 제공하는 것으로, 빈 이름을 변경하는 것이 아님
- `RateDiscountPolicy` 클래스에 `@Qualifier("mainDiscountPolicy")` 추가
- `FixDiscountPolicy` 클래스에 `@Qualifier("fixDiscountPolicy")` 추가
- 아래와 같이 생성자에서 사용 가능
    
    ```java
    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
    		this.memberRepository = memberRepository;
    		this.discountPolicy = discountPolicy;
    }
    ```
    
- `@Qualifier`로도 빈 이름을 검색할 수 있지만, `@Qualifier`끼리만 매칭하는게 좋음

### `@Primary` 사용

- `@Autowired` 매칭할 때 빈이 여러 개 매칭되면, `@Primary`를 가진 빈이 우선권을 가짐
- `RateDiscountPolicy`, `FixDiscountPolicy` 모두 `@Qualifier`를 제거
- `RateDiscountPolicy`에 `@Primary` 추가

### `@Primary`, `@Qualifier`의 활용

- `@Qualifier`는 주입받는 모든 코드에 `@Qualifier`를 붙여야 함
- `@Qualifier`가 `@Primary`보다 우선 순위가 높음

## 애노테이션 직접 만들기

- `@Qualifier`를 사용하면 내부가 문자열이기 때문에 컴파일 에러를 잡을 수 없음
    - 애노테이션을 만들어서 해결 가능
- `../annotation/MainDiscountPolicy.java` 생성
    
    ```java
    @Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER,
            ElementType.TYPE, ElementType.ANNOTATION_TYPE})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Qualifier("mainDiscountPolicy")
    public @interface MainDiscountPolicy {
    }
    ```
    
- `OrderServiceImpl`의 생성자에도 적용
    
    ```java
    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, @MainDiscountPolicy DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
    ```
    
    - `@MainDiscountPolicy`에 `@Qualifier("mainDiscountPolicy")`가 포함되어 있기 때문에 적용됨
        - 어노테이션 상속이 아니라, 어노테이션을 모아서 사용하는, 스프링에서 지원하는 기능임

## 조회한 빈이 모두 필요할 때, `List`, `Map`

- `test/../autowired/AllBeanTest.java` 생성
    
    ```java
    public class AllBeanTest {
        
        @Test
        void findAllBean() {
            ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class, DiscountService.class);
            DiscountService discountService = ac.getBean(DiscountService.class);
            Member member = new Member(1L, "userA", Grade.VIP);
            int discountPrice = discountService.discount(member, 10000, "fixDiscountPolicy");
            
            assertThat(discountService).isInstanceOf(DiscountService.class);
            assertThat(discountPrice).isEqualTo(1000);
        }
    
        static class DiscountService {
            
            private final Map<String, DiscountPolicy> policyMap;
            private final List<DiscountPolicy> policies;
            
            public DiscountService(Map<String, DiscountPolicy> policyMap, List<DiscountPolicy> policies) {
                this.policyMap = policyMap;
                this.policies = policies;
                
                System.out.println("policyMap = " + policyMap);
                System.out.println("policies = " + policies);
            }
            
            public int discount(Member member, int price, String discountCode) {
                DiscountPolicy discountPolicy = policyMap.get(discountCode);
                
                System.out.println("discountCode = " + discountCode);
                System.out.println("discountPolicy = " + discountPolicy);
                
                return discountPolicy.discount(member, price);
            }
        }
    }
    ```
    
    - config 파일은 한 번에 여러 개 등록 가능
        - 여기선 `DiscountPolicy` 끌어오기 위해 `AutoAppConfig`도 등록함
        - 빈에 등록된 `DiscountPolicy`들을 `Map`, `List` 타입으로 한꺼번에 받을 수 있음

## 자동, 수동의 올바른 실무 운영 기준

- 기본적으로는 자동 등록을 사용하는게 좋음
    - 빈이 많아지면 config 관리하는 것 자체에 공수가 듦
    - 자동 빈 등록도 OCP, DIP 지킴
- 직접 구현한 기술 지원 관련 빈은 수동 등록하는게 좋음
    - 업무 로직 빈 : MVC 관련 빈들과 같이 비즈니스 요구사항을 개발할 때 사용하는 빈
    - 기술 지원 빈 : DB 연결, 공통 로그 등의 기술적인 문제를 해결할 때 사용하는 빈
        - 수동 등록해서 config에 명확하게 드러내는게 좋음
        - 스브링에서 제공하는 기술 지원 빈들은 자동 등록됨
- 비즈니스 로직에서도 다형성을 활용할 경우에는 수동 등록이나 같은 패키지에 묶어두는게 좋음
    - `DiscountPolicy`의 경우 `discount`라는 패키지로 묶어서 가독성 증가