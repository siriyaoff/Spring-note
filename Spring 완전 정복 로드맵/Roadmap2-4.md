# 섹션 4. 스프링 컨테이너와 스프링 빈

## 스프링 컨테이너 생성

```java
ApplicationContext applicationContext = 
		new AnnotationConfigApplicationContext(AppConfig.class);
```

- `ApplicationContext`
    - Spring container라고 부름
    - Interface임
    - 어노테이션 기반의 `AppConfig` 클래스로 만들거나 XML로 만들 수 있음
- `AnnotationConfigApplicationContext` : `ApplicationContext` Interface의 구현체
- spring container는 `BeanFactory`, `ApplicationContext` 등이 존재함
    - `BeanFactory`가 최상위 객체이고 `ApplicationContext` 등이 거기에 유틸성이 추가된 형태

### 스프링 컨테이너의 생성 과정

1. 스프링 컨테이너 생성
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-4%20(1).png)
    
- `new AnnotationConfigApplicationContext(AppConfig.class)`
    - `AppConfig.class`를 spring container의 구성 정보로 지정함
1. 스프링 빈 등록
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-4%20(2).png)
    
- spring container가 `AppConfig.class`에 `@Bean` 어노테이션이 붙은 메소드들을 실행하고 스프링 빈 저장소에 등록함
    - Bean 이름 : 메소드 이름
        - Bean 이름은 `@Bean(name="memberSvc2")`와 같이 직접 부여할 수도 있음
        - 이름이 중복되면 안됨
    - Bean 객체 : 반환되는 객체
1. 스프링 빈 의존관계 설정 - 준비
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-4%20(3).png)
    
2. 스프링 빈 의존관계 설정 - 완료
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-4%20(4).png)
    
- config를 참고해서 의존관계를 주입(DI)함
- 자바 코드로 Bean을 등록하면 생성자를 호출하면서 DI도 한번에 처리됨
    - spring life cycle에서는 Bean을 생성하고 DI하는 단계가 나누어짐
    - 실제로도 의존관계 자동 주입이 될 때도 있음

## 컨테이너에 등록된 모든 빈 조회

- `test/.../core/beanfind/ApplicationContextInfoTest.java` 생성
    
    ```java
    public class ApplicationContextInfoTest {
        AnnotationConfigApplicationContext ac=new AnnotationConfigApplicationContext(AppConfig.class);
    
        @Test
        @DisplayName("모든 빈 출력하기")
        void findAllBean(){
            String[] beanDefinitionNames=ac.getBeanDefinitionNames();
            for(String beanDefinitionName:beanDefinitionNames){
                Object bean = ac.getBean(beanDefinitionName);
                System.out.println("name = "+beanDefinitionName+" object = "+bean);
            }
        }
    
        @Test
        @DisplayName("애플리케이션 빈 출력하기")
        void findApplicationBean() {
            String[] beanDefinitionNames=ac.getBeanDefinitionNames();
            for (String beanDefinitionName : beanDefinitionNames) {
                BeanDefinition beanDefinition = ac.getBeanDefinition(beanDefinitionName);
                
                // Role ROLE_APPLICATION: 직접 등록한 애플리케이션 빈
                // Role ROLE_INFRASTRUCTURE: 스프링이 내부에서 사용하는 빈
                if (beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
                    Object bean = ac.getBean(beanDefinitionName);
                    System.out.println("name = " + beanDefinitionName + " object = " + bean);
                }
            }
        }
    }
    ```
    
    - `ac.getBeanDefinitionNames()` : spring에 등록된 모든 빈 정보 출력
    - `ac.getBean(name)` : 빈 이름이 `name`인 인스턴스 조회
    - `beanDefinition.getRole()` : 인스턴스의 `ROLE` 반환
        - `BeanDefinition.ROLE_APPLICATION` : 사용자가 정의한 빈
        - `BeanDefinition.ROLE_INFRASTRUCTURE` : 스프링 내부에서 사용하는 빈
    
    ```
    name = org.springframework.context.annotation.internalConfigurationAnnotationProcessor object = org.springframework.context.annotation.ConfigurationClassPostProcessor@20b12f8a
    name = org.springframework.context.annotation.internalAutowiredAnnotationProcessor object = org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor@e84a8e1
    name = org.springframework.context.annotation.internalCommonAnnotationProcessor object = org.springframework.context.annotation.CommonAnnotationBeanPostProcessor@2e554a3b
    name = org.springframework.context.event.internalEventListenerProcessor object = org.springframework.context.event.EventListenerMethodProcessor@54a67a45
    name = org.springframework.context.event.internalEventListenerFactory object = org.springframework.context.event.DefaultEventListenerFactory@7d42c224
    name = appConfig object = hello.core.AppConfig$$EnhancerBySpringCGLIB$$d6ae189e@56aaaecd
    name = memberService object = hello.core.member.MemberServiceImpl@522a32b1
    name = memberRepository object = hello.core.member.MemoryMemberRepository@35390ee3
    name = orderService object = hello.core.order.OrderServiceImpl@5e01a982
    name = discountPolicy object = hello.core.discount.RateDiscountPolicy@5ddea849
    ```
    
    - 아래 5개가 직접 등록한 빈(`ROLE_APPLICATION`)임

## 스프링 빈 조회 - 기본

- `test/.../core/beanfind/ApplicationContextBasicFindTest.java` 생성
    
    ```java
    public class ApplicationContextBasicFindTest {
        AnnotationConfigApplicationContext ac=new AnnotationConfigApplicationContext(AppConfig.class);
    
        @Test
        @DisplayName("빈 이름으로 조회")
        void findBeanByName(){
            MemberService memberService = ac.getBean("memberService", MemberService.class);
            assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
        }
    
        @Test
        @DisplayName("이름 없이 타입으로만 조회")
        void findBeanByType(){
            MemberService memberService = ac.getBean(MemberService.class);
            assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
        }
    
        @Test
        @DisplayName("구현체 타입으로 조회")
        void findBeanByName2(){
            MemberServiceImpl memberService = ac.getBean("memberService", MemberServiceImpl.class);
            assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
        }
    
        @Test
        @DisplayName("빈 이름으로 조회X")
        void findBeanByNameX(){
            Assertions.assertThrows(NoSuchBeanDefinitionException.class,
                    () -> ac.getBean("xxxxx", MemberService.class));
        }
    }
    ```
    
    - `getBean`은 스프링 빈에 등록된 instance의 타입으로 검색하기 때문에 `Impl` 객체로 검색해도 됨
        - 하지만 구현체에 의존하기 때문에 좋은 코드는 아닐 수 있음
    - `assertThrows`는 `junit` 패키지에 있음
    - 실패 테스트는 일단 돌려서 exception 알아내고 작성해도 될 듯

## 스프링 빈 조회 - 동일한 타입이 둘 이상

- 타입으로 조회할 때 동일한 타입이 둘 이상 들어있으면 오류가 발생함
- `test/.../core/beanfind/ApplicationContextSameBeanFindTest.java` 생성
    
    ```java
    public class ApplicationContextSameBeanFindTest {
        AnnotationConfigApplicationContext ac=new AnnotationConfigApplicationContext(SameBeanConfig.class);
    
        @Test
        @DisplayName("같은 타입이 둘 이상 있으면 오류 발생")
        void findBeanByTypeDuplicate(){
            assertThrows(NoUniqueBeanDefinitionException.class,
                    () -> ac.getBean(MemberRepository.class));
        }
    
        @Test
        @DisplayName("같은 타입이 둘 이상 있으면 빈 이름을 지정")
        void findBeanByName(){
            MemberRepository memberRepository = ac.getBean("memberRepository1", MemberRepository.class);
            assertThat(memberRepository).isInstanceOf(MemberRepository.class);
        }
    
        @Test
        @DisplayName("특정 타입을 모두 조회")
        void findAllBeanByType(){
            Map<String, MemberRepository> beansOfType = ac.getBeansOfType(MemberRepository.class);
            for (String key : beansOfType.keySet()) {
                System.out.println("key = " + key + " value = " + beansOfType.get(key));
            }
            System.out.println("beansOfType = " + beansOfType);
            assertThat(beansOfType.size()).isEqualTo(2);
        }
    
        @Configuration
        static class SameBeanConfig {
            @Bean
            public MemberRepository memberRepository1() {
                return new MemoryMemberRepository();
            }
    
            @Bean
            public MemberRepository memberRepository2() {
                return new MemoryMemberRepository();
            }
        }
    }
    ```
    
    - `ac.getBeansOfType()`을 사용하면 해당 타입의 모든 빈 조회 가능
    - 테스트 클래스 내부에서 config 만들어서 사용해도 됨

## 스프링 빈 조회 - 상속 관계

- 부모 타입으로 조회하면 자식 타입들도 검색됨
    - `Object` 타입으로 조회하면 모든 스프링 빈을 조회함
- `test/.../core/beanfind/ApplicationContextExtendsFindTest.java` 생성
    
    ```java
    public class ApplicationContextExtendsFindTest {
        AnnotationConfigApplicationContext ac=new AnnotationConfigApplicationContext(TestConfig.class);
    
        @Test
        @DisplayName("부모 타입 조회시 자식이 둘 이상 있으면 중복 오류 발생")
        void findBeanByParentTypeDuplicate() {
            assertThrows(NoUniqueBeanDefinitionException.class,
                    () -> ac.getBean(DiscountPolicy.class));
        }
    
        @Test
        @DisplayName("부모 타입 조회시 자식이 둘 이상 있으면 빈 이름을 지정해야 함")
        void findBeanByParentTypeBeanName() {
            DiscountPolicy rateDiscountPolicy = ac.getBean("rateDiscountPolicy", DiscountPolicy.class);
            assertThat(rateDiscountPolicy).isInstanceOf(RateDiscountPolicy.class);
        }
    
        @Test
        @DisplayName("특정 하위 타입으로 조회")
        void fidnBeanBySubType() {
            RateDiscountPolicy bean = ac.getBean(RateDiscountPolicy.class);
            assertThat(bean).isInstanceOf(RateDiscountPolicy.class);
        }
    
        @Test
        @DisplayName("부모 타입으로 모두 조회")
        void findAllBeanByParentType() {
            Map<String, DiscountPolicy> beansOfType = ac.getBeansOfType(DiscountPolicy.class);
            assertThat(beansOfType.size()).isEqualTo(2);
            for (String key : beansOfType.keySet()) {
                System.out.println("key = " + key + " value = " + beansOfType.get(key));
            }
        }
    
        @Test
        @DisplayName("부모 타입으로 모두 조회 - Object")
        void findAllBeanByObjectType() {
            Map<String, Object> beansOfType = ac.getBeansOfType(Object.class);
            for (String key : beansOfType.keySet()) {
                System.out.println("key = " + key + " value = " + beansOfType.get(key));
            }
        }
    
        @Configuration
        static class TestConfig {
            @Bean
            public DiscountPolicy rateDiscountPolicy() {
                return new RateDiscountPolicy();
            }
    
            @Bean
            public DiscountPolicy fixDiscountPolicy() {
                return new FixDiscountPolicy();
            }
        }
    }
    ```
    

## BeanFactory와 ApplicationContext

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-4%20(5).png)

- BeanFactory
    - 스프링 컨테이너의 최상위 인터페이스
    - 스프링 빈을 관리하고 조회하는 역할(`getBean`, …)
- ApplicationContext
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-4%20(6).png)
    
    - `MessageSource` : 국제화 기능(언어 자동 제공)
    - `EnvironmentCapable` : 환경 변수 처리
    - `ApplicationEventPublisher` : 이벤트 생성, 구독하는 모델 지원
    - `ResourceLoader` : 파일, 클래스 패스, 외부에서 리소스를 편리하게 조회
- BeanFactory를 직접적으로 사용할 일은 거의 없고 부가 기능이 포함된 ApplicationContext를 주로 사용함
- BeanFactory, ApplicationContext를 Spring container라고 부름

> 클래스 - 클래스의 다중 상속은 불가능하지만, 인터페이스-인터페이스 다중 상속은 가능, 다중 구현도 가능
> 

## 다양한 설정 형식 지원 - 자바 코드, XML

- 자바 코드, XMl, Groovy 등의 형식으로 configuration 설정 가능
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-4%20(7).png)
    
    - Annotation 기반 : `AnnotationConfigApplicationContext` 클래스를 사용해 config 정보 전달
    - XML 기반
        - 컴파일 없이 config 정보 변경 가능
        - 최근에는 거의 사용 X

### XmlAppConfig 사용

- `main/resources/appConfig.xml` 생성
    
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
        <bean id="memberService" class="hello.core.member.MemberServiceImpl">
            <constructor-arg name="memberRepository" ref="memberRepository" />
        </bean>
        <bean id="memberRepository"
              class="hello.core.member.MemoryMemberRepository" />
        <bean id="orderService" class="hello.core.order.OrderServiceImpl">
            <constructor-arg name="memberRepository" ref="memberRepository" />
            <constructor-arg name="discountPolicy" ref="discountPolicy" />
        </bean>
        <bean id="discountPolicy" class="hello.core.discount.RateDiscountPolicy" />
    </beans>
    ```
    
    - `constructor-arg`로 생성자를 지정
    - `AppConfig.java`와 형식만 다르고 내용은 동일함
- `test/java/hello.core/xml/XmlAppContext.java` 생성
    
    ```java
    public class XmlAppContext {
        @Test
        void xmlAppContext(){
            ApplicationContext ac = new GenericXmlApplicationContext("appConfig.xml");
            MemberService memberService = ac.getBean("memberService", MemberService.class);
            Assertions.assertThat(memberService).isInstanceOf(MemberService.class);
        }
    }
    ```
    

## 스프링 빈 설정 메타 정보 - BeanDefinition

- 스프링은 `BeanDefinition` interface를 사용해서 다양한 config 설정 방법을 지원함
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-4%20(8).png)
    
    - `BeanDefinition` : bean metadata
        - `@Bean`, `<bean>` 각각 하나의 metadata가 생성됨
- spring container는 이 metadata를 기반으로 spring bean을 생성함
- `ApplicationContext` 클래스 다이어그램
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-4%20(9).png)
    
    - `AnnotationConfigApplicationContext`는 `AnnotatedBeanDefinitionReader`를 사용해서 `AppConfig.class`를 읽고 `BeanDefinition`을 생성함
    - config를 추가하려면 마찬가지로 `XxxBeanDefinitionReader`를 만들어서 `BeanDefinition`을 생성하면 됨

### BeanDefinition 살펴보기

- `test/../beandefinition/BeanDefinitionTest.java` 생성
    
    ```java
    public class BeanDefinitionTest {
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
    
        @Test
        @DisplayName("빈 설정 메타정보 확인")
        void findApplicationBean() {
            String[] beanDefinitionNames = ac.getBeanDefinitionNames();
            for (String beanDefinitionName : beanDefinitionNames) {
                BeanDefinition beanDefinition = ac.getBeanDefinition(beanDefinitionName);
    
                if (beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
                    System.out.println("beanDefinitionName = " + beanDefinitionName + 
                            " beanDefinition = " + beanDefinition);
                }
            }
        }
    }
    ```
    
    - `ac.getBeanDefinition()`을 사용하기 위해서 `ac`를 `AnnotationConfigApplicationContext`로 선언함(`ApplicationContext`로 선언하면 저 메소드가 없음)
    
    ```
    beanDefinitionName = appConfig beanDefinition = Generic bean: class [hello.core.AppConfig$$EnhancerBySpringCGLIB$$521c7d39]; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null
    beanDefinitionName = memberService beanDefinition = Root bean: class [null]; scope=; abstract=false; lazyInit=null; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=appConfig; factoryMethodName=memberService; initMethodName=null; destroyMethodName=(inferred); defined in hello.core.AppConfig
    beanDefinitionName = memberRepository beanDefinition = Root bean: class [null]; scope=; abstract=false; lazyInit=null; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=appConfig; factoryMethodName=memberRepository; initMethodName=null; destroyMethodName=(inferred); defined in hello.core.AppConfig
    beanDefinitionName = orderService beanDefinition = Root bean: class [null]; scope=; abstract=false; lazyInit=null; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=appConfig; factoryMethodName=orderService; initMethodName=null; destroyMethodName=(inferred); defined in hello.core.AppConfig
    beanDefinitionName = discountPolicy beanDefinition = Root bean: class [null]; scope=; abstract=false; lazyInit=null; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=appConfig; factoryMethodName=discountPolicy; initMethodName=null; destroyMethodName=(inferred); defined in hello.core.AppConfig
    /*
    beanDefinitionName = memberService 
    beanDefinition = Root bean: class [null]; 
    scope=; abstract=false; lazyInit=null; autowireMode=3; 
    dependencyCheck=0; autowireCandidate=true; primary=false; 
    factoryBeanName=appConfig; factoryMethodName=memberService; 
    initMethodName=null; destroyMethodName=(inferred); 
    defined in hello.core.AppConfig
    */
    ```
    
    - `BeanClassName` : 생성할 빈의 클래스 이름(팩토리빈을 사용하면 없음)
    - `factoryBeanName` : 팩토리빈 이름
    - `factoryMethodName` : 빈을 생성할 팩토리 메서드 지정
    - `Scope` : 싱글톤(기본값)
    - `lazyInit` : 실제 빈을 사용할 때 까지 최대한 생성을 지연
    - `InitMethodName` : 빈을 생성하고, 의존관계를 적용한 뒤에 호출되는 초기화 메서드 이름
    - `DestroyMethodName` : 빈의 생명주기가 끝나서 제거할 때 호출되는 메서드 명
    - `Constructor arguments, Properties`: 의존관계 주입할 때 사용(팩토리빈을 사용하면 없음)
    - xml을 사용하면 클래스가 명확하게 드러나고 팩토리 관련 내용이 비어있음
    - Annotationconfig를 사용하면 팩토리빈을 통해서 등록하기 때문에 클래스는 나오지 않고 팩토리빈 관련 속성이 나옴