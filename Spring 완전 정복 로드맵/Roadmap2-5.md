# 섹션 5. 싱글톤 컨테이너

## 웹 애플리케이션과 싱글톤

- 대부분의 스프링 앱은 동시에 여러 요청이 들어오는 웹 어플리케이션임
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-5%20(1).png)
    
- `test/../singleton/SingletonTest.java` 생성
    
    ```java
    public class SingletonTest {
        @Test
        @DisplayName("스프링 없는 순수 DI 컨테이너")
        void pureContainer() {
            AppConfig appConfig = new AppConfig();
            MemberService memberService1 = appConfig.memberService();
            MemberService memberService2 = appConfig.memberService();
    
            System.out.println("memberService1 = " + memberService1);
            System.out.println("memberService2 = " + memberService2);
            
            Assertions.assertThat(memberService1).isNotSameAs(memberService2);
        }
    }
    ```
    
    - `appConfig`에 요청할 때마다 새로운 객체가 생성됨
    - 해당 객체를 하나만 생성해서 공유하는 싱글톤 패턴을 사용할 수 있음

## 싱글톤 패턴

- Instance가 단 하나만 생성되는 것을 보장하는 디자인 패턴
    - 객체의 instance가 2개 이상 생성되지 못하도록 막아야 함
        - `private` 생성자를 사용해서 외부에서 `new`를 사용하지 못하도록 막음
    - spring container의 기본적인 빈 등록 방식은 싱글톤임
- `test/../singleton/SingletonService.java` 생성
    
    ```java
    public class SingletonService {
        private static final SingletonService instance = new SingletonService();
    
        public static SingletonService getInstance() {
            return instance;
        }
    
        private SingletonService() {
        }
        
        public void logic() {
            System.out.println("싱글톤 객체 로직 호출");
        }
    }
    ```
    
- `test/../singleton/SingletonTest.java`에 아래 테스트 추가
    
    ```java
    @Test
    @DisplayName("싱글톤 패턴을 적용한 객체 사용")
    void singletonServiceTest() {
        SingletonService singletonService1 = SingletonService.getInstance();
        SingletonService singletonService2 = SingletonService.getInstance();
    
        assertThat(singletonService1).isSameAs(singletonService2);
    }
    ```
    
    - same(`==`) : 인스턴스의 주소 비교
    - equal(`is equal to`) : 인스턴스의 내용 비교

### 싱글톤 패턴의 문제점

- 싱글톤 패턴 구현 코드가 김
- 의존관계상 클라이언트가 구체 클래스에 의존함(`SingletonService.getInstance()`로 인스턴스 호출)
    - DIP 위반
    - OCP 원칙을 위반할 가능성이 높음
    - 테스트 어려움
- 내부 속성 변경, 초기화가 어려움
- private 생성자를 사용해 유연성이 떨어짐
    - 자식 클래스 만들기 어려움
    - DI 적용이 어려움

## 싱글톤 컨테이너

- spring container는 싱글톤 패턴의 문제점을 해결하면서 spring bean의 인스턴스를 싱글톤으로 관리함
- `test/../singleton/SingletonTest.java`에 아래 테스트 추가
    
    ```java
    @Test
    @DisplayName("스프링 컨테이너와 싱글톤")
    void springContainer() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
        MemberService memberService1 = ac.getBean("memberService", MemberService.class);
        MemberService memberService2 = ac.getBean("memberService", MemberService.class);
    
        System.out.println("memberService1 = " + memberService1);
        System.out.println("memberService2 = " + memberService2);
    
        assertThat(memberService1).isSameAs(memberService2);
    }
    ```
    
    - `AppConfig.class`를 사용해서 `ApplicationContext` 생성함
    - 싱글톤 컨테이너 적용 후
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-5%20(2).png)
    

## 싱글톤 방식의 주의점

- 하나의 인스턴스를 여러 클라이언트가 공유하기 때문에 stateful이 아닌 stateless하게 설계해야 함
    - 특정 client에 의존적인 필드 X
    - 특정 client가 수정 가능한 필드 X
    - 가급적 읽기만 가능해야 함
    - 필드 대신 지역 변수, 파라미터, ThreadLocal 등을 사용해야 함
- `test/../singleton/StatefulService.java` 생성
    
    ```java
    public class StatefulService {
        private int price;
    
        public void order(String name, int price) {
            System.out.println("name = " + name + " price = " + price);
            this.price = price;
        }
    
        public int getPrice() {
            return price;
        }
    }
    ```
    
- `test/../singleton/StatefulServiceTest.java` 생성
    
    ```java
    class StatefulServiceTest {
        @Test
        void statefulServiceSingleton() {
            ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);
            StatefulService statefulService1 = ac.getBean(StatefulService.class);
            StatefulService statefulService2 = ac.getBean(StatefulService.class);
    
            statefulService1.order("userA", 10000);
            statefulService2.order("userB", 20000);
    
            int price = statefulService1.getPrice();
            System.out.println("price = " + price);
    
            Assertions.assertThat(statefulService1.getPrice()).isEqualTo(20000);
        }
    
        static class TestConfig{
            @Bean
            public StatefulService statefulService() {
                return new StatefulService();
            }
        }
    }
    ```
    

## `@Configuration`과 싱글톤

- `AppConfig.java` 코드
    
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
    
    - `memberService()`, `orderService()` 둘 다 `memberRepository()`를 호출하고 `MemoryMemberRepository()`가 두 번 호출됨
- `MemberServiceImpl`, `OrderServiceImpl` 둘 다 `getMemberRepository()` 메소드를 만들고 `test/../singletonConfigurationSingletonTest.java` 생성
    
    ```java
    // getMemberRepository()
    public MemberRepository getMemberRepository() {
        return memberRepository;
    }
    
    public class ConfigurationSingletonTest {
        @Test
        void configurationTest() {
            ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
    
            MemberServiceImpl memberService = ac.getBean("memberService", MemberServiceImpl.class);
            OrderServiceImpl orderService = ac.getBean("orderService", OrderServiceImpl.class);
            MemberRepository memberRepository = ac.getBean("memberRepository", MemberRepository.class);
    
            MemberRepository memberRepository1 = memberService.getMemberRepository();
            MemberRepository memberRepository2 = orderService.getMemberRepository();
    
            System.out.println("memberService -> memberRepository = " + memberRepository1);
            System.out.println("orderService -> memberRepository = " + memberRepository2);
            System.out.println("memberRepository = " + memberRepository);
    
            Assertions.assertThat(memberService.getMemberRepository()).isSameAs(memberRepository);
            Assertions.assertThat(orderService.getMemberRepository()).isSameAs(memberRepository);
        }
    }
    ```
    
    - 모두 같은 MMR 인스턴스임
- `AppConfig.java` 코드 수정
    
    ```java
    ...
    @Bean
    public MemberService memberService() {
        System.out.println("call AppConfig.memberService");
        return new MemberServiceImpl(memberRepository());
    }
    
    @Bean
    public MemberRepository memberRepository() {
        System.out.println("call AppConfig.memberRepository");
        return new MemoryMemberRepository();
    }
    
    @Bean
    public OrderService orderService() {
        System.out.println("call AppConfig.orderService");
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }
    ...
    ```
    
    - 예상 결과
        
        ```
        call AppConfig.memberService
        call AppConfig.memberRepository
        call AppConfig.memberRepository
        call AppConfig.orderService
        call AppConfig.memberRepository
        ```
        
    - `ConfigurationSingletonTest`를 돌리면 아래와 같이 호출됨
        
        ```
        call AppConfig.memberService
        call AppConfig.memberRepository
        call AppConfig.orderService
        memberService -> memberRepository = hello.core.member.MemoryMemberRepository@2c7b5824
        orderService -> memberRepository = hello.core.member.MemoryMemberRepository@2c7b5824
        memberRepository = hello.core.member.MemoryMemberRepository@2c7b5824
        ```
        

## `@Configuration`과 바이트코드 조작의 마법

- `ConfigurationSingletonTest.java`에 테스트 추가
    
    ```java
    @Test
    void configurationDeep() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
        AppConfig bean = ac.getBean(AppConfig.class);
    
        System.out.println("bean = " + bean.getClass());
    }
    ```
    
    - 결과 : `bean = class hello.core.AppConfig$$EnhancerBySpringCGLIB$$ee4ff54b`
        - `class hello.core.AppConfig`가 나와야 함
- 스프링이 `CGLIB`이라는 바이트코드 조작 라이브러리를 사용해서 `AppConfig` 클래스를 상속받은 다른 클래스를 만들고, 그 클래스를 spring bean으로 등록한 것임
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-5%20(3).png)
    
    - 만들어진 클래스는 아래 슈도코드처럼 싱글톤이 보장되도록 구현되어 있음
        
        ```java
        // memberRepository() in AppConfig@CGLIB
        @Bean
        public MemberRepository memberRepository() {
        		if (memoryMemberRepository in Spring Container?) {
        				return the memoryMemberRepository in Spring Container;
        		} else {
        				invoke original memberRepository() in AppConfig.java
        				Register the instance into Spring Container
        				return the instance
        		}
        }
        ```
        
    - `AppConfig@CGLIB` 클래스는 `AppConfig`의 자식 클래스이기 때문에 `AppConfig`로 조회해도 검색됨

### `@Configuration`을 적용하지 않고 `@Bean`만 적용할 경우

- `AppConfig.java`에서 `@Configuration` 삭제 후 `ConfigurationSingletonTest.java` 실행
    - `configurationDeep()` 결과
        
        ```java
        call AppConfig.memberService
        call AppConfig.memberRepository
        call AppConfig.memberRepository
        call AppConfig.orderService
        call AppConfig.memberRepository
        bean = class hello.core.AppConfig
        ```
        
    - `configurationTest()` 결과
        
        ```java
        call AppConfig.memberService
        call AppConfig.memberRepository
        call AppConfig.memberRepository
        call AppConfig.orderService
        call AppConfig.memberRepository
        memberService -> memberRepository = hello.core.member.MemoryMemberRepository@2631f68c
        orderService -> memberRepository = hello.core.member.MemoryMemberRepository@6ed3f258
        memberRepository = hello.core.member.MemoryMemberRepository@8ad6665
        ```
        
    - `@Bean`이 있기 때문에 spring bean으로 등록은 되지만 singleton이 보장되지는 않음