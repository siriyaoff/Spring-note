# 섹션 6. 컴포넌트 스캔

## 컴포넌트 스캔과 의존관계 자동 주입 시작하기

- `@Bean`을 사용해서 config 파일에서 직접 등록하는 대신, 컴포넌트 스캔을 사용하여 스프링 빈으로 자동 등록할 수 있음
- `@Autowired`을 사용해서 DI도 자동으로 가능함
- `../hello.core/AutoAppConfig.java` 생성
    
    ```java
    @Configuration
    @ComponentScan(
            excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Configuration.class)
    )
    public class AutoAppConfig {
    }
    ```
    
    - config 파일에 `@ComponentScan` 붙여서 컴포넌트 스캔 사용 가능
    - `@Component` 어노테이션이 붙은 클래스들을 스프링 빈으로 등록함
    - `excludeFilters...` : 기존의 config 파일들이 컴포넌트 스캔의 대상이 되어서 수동으로 빈을 등록하는 코드가 실행되지 않도록 하기 위해서 `Configuration.class`를 스캔 대상에서 제외함
- `MemoryMemberRepository`, `RateDiscountPolicy`에 `@Component` 추가
- `MemberServiceImpl`에 `@Component`, 생성자에 `@Autowired` 추가
- `OrderServiceImpl`에 `@Component`, 생성자에 `@Autowired` 추가
- `test/../scan/AutoAppConfigTest.java` 생성
    
    ```java
    public class AutoAppConfigTest {
        @Test
        void basicScan(){
            AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class);
    
            MemberService memberService = ac.getBean(MemberService.class);
            Assertions.assertThat(memberService).isInstanceOf(MemberService.class);
        }
    }
    ```
    
    - 로그에서 `ClassPathBeanDefinitionScanner`가 candidate를 등록하고 singleton bean이 등록되는 것과 DI를 확인할 수 있음

### `@ComponentScan`, `@Autowired`의 동작 과정

1. `@ComponentScan`
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-6%20(1).png)
    
    - `@ComponentScan`은 `@Component`가 붙은 모든 클래스를 spring bean으로 등록함
    - 이름은 맨 앞글자만 소문자로 바꾼 클래스 명이 기본으로 지정됨
        - `@Component("memberService2")`와 같이 직접 이름 지정 가능
2. `@Autowired` 자동 DI
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-6%20(2).png)
    
    - 기본적으로 타입이 같은 빈을 찾아서 주입함

## 탐색 위치와 기본 스캔 대상

**패키지 탐색 시작 위치 지정**

- `/../hello.core/AutoAppConfig.java` 수정
    
    ```java
    @Configuration
    @ComponentScan(
            basePackages = "hello.core",
            excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Configuration.class)
    )
    public class AutoAppConfig {
    }
    ```
    
- `basePackages` : 패키지 탐색의 시작 위치를 지정함
    - `basePackages = {"hello.core", "hello.service"}`와 같이 여러 개 지정 가능
- `basePackageClasses`로 시작 위치를 지정한 클래스의 패키지로 지정 가능
- 지정하지 않으면 `@ComponentScan`이 붙은 config 클래스의 패키지가 시작 위치로 설정됨
- config 클래스의 위치를 프로젝트 최상단에 두는 것이 관례임
    - 예제에서는 `hello.core`에 config 파일을 두고 `@ComponentScan` 어노테이션을 붙이면 `hello.core`를 포함한 하위 패키지들이 스캔 대상이 됨
    - SpringBoot를 사용할 경우 `@SpringBootApplication`을 루트 위치에 두면 됨
        - `@SpringBootApplication`에도 `@ComponentScan`이 포함되어 있음

**컴포넌트 스캔 기본 대상**

- `@Configuration`, `@Service`, `@Repository`, `@Controller` 등의 어노테이션들은 모두 `@Component` 어노테이션을 포함함
    - 어노테이션끼리 포함 관계는 상속을 사용하는 것이 아니라 스프링에서 지원하는 기능임
    - `@Repository` : 스프링 데이터 접근 계층으로 인식하고 데이터 계층의 예외를 스프링 예외로 변환해줌
    - `@Configuration` : 스프링 config 정보로 인식하고, spring bean이 singleton을 유지하도록 처리함
    - `@Service` : 특별한 기능은 없고 비즈니스 계층을 인식하는데 도움을 줌

## 필터

- `includeFilters` : component scan 대상 추가
- `excludeFilters` : component scan 제외 대상 추가
- `test/../scan/filter/MyIncludeComponent.java` 생성
    
    ```java
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    public @interface MyincludeComponent {
    }
    ```
    
- `test/../scan/filter/MyExcludeComponent.java` 생성
    
    ```java
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    public @interface MyExcludeComponent {
    }
    ```
    
- `test/../scan/filter/BeanA.java` 생성
    
    ```java
    @MyIncludeComponent
    public class BeanA {
    }
    ```
    
- `test/../scan/filter/BeanB.java` 생성
    
    ```java
    @MyExcludeComponent
    public class BeanB {
    }
    ```
    
- `test/../scan/filter/ComponentFilterAppConfigTest.java` 생성
    
    ```java
    class ComponentFilterAppConfigTest {
        @Test
        void filterScan(){
            ApplicationContext ac = new AnnotationConfigApplicationContext(ComponentFilterAppConfig.class);
            BeanA beanA = ac.getBean("beanA", BeanA.class);
            Assertions.assertThat(beanA).isNotNull();
            assertThrows(
                    NoSuchBeanDefinitionException.class,
                    () -> ac.getBean("beanB", BeanB.class)
            );
        }
    
        @Configuration
        @ComponentScan(
                includeFilters = @Filter(type= FilterType.ANNOTATION, classes = MyIncludeComponent.class),
                excludeFilters = @Filter(type= FilterType.ANNOTATION, classes = MyExcludeComponent.class)
        )
        static class ComponentFilterAppConfig{
        }
    }
    ```
    

### `FilterType` 옵션

- `ANNOTATION` : 기본값
- `ASSIGNABLE_TYPE` : 지정한 타입과 자식 타입을 인식함
- `ASPECTJ` : AspectJ 패턴 사용(`org.example..*Service+`)
- `REGEX` : 정규 표현식
- `CUSTOM` : `TypeFilter` interface를 구현해서 처리

## 중복 등록과 충돌

### 자동 빈 등록 vs 자동 빈 등록

- `ConflictingBeanDefinitionException` 예외 발생

### 수동 빈 등록 vs 자동 빈 등록

- `../AutoAppConfig.java` 수정
    
    ```java
    @Configuration
    @ComponentScan(
            basePackages = "hello.core",
            excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Configuration.class)
    )
    public class AutoAppConfig {
        @Bean(name="memoryMemberRepository")
        MemberRepository memberRepository(){
            return new MemoryMemberRepository();
        }
    }
    ```
    
    - `AutoAppConfigTest` 실행하면 오류 없이 `Overriding bean definition for ...` 로그만 나옴
        - 수동 빈이 자동 빈을 overriding 함
- 최근에는 스프링 부트에서는 수동 빈 등록, 자동 빈 등록이 충돌하면 오류가 발생하도록 기본 값이 바뀌었음
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-6%20(3).png)
    
    - `CoreApplication` 실행 결과
    - `src/main/resources/application.properties`에 `spring.main.allow-bean-definition-overriding=true` 옵션 추가하면 overriding 하도록 설정 가능
    - `AutoAppConfigTest`는 spring boot로 실행한게 아니기 때문에 오버라이딩이 자동으로 됨