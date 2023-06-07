# 섹션 9. 빈 스코프

## 빈 스코프란?

- spring bean이 존재할 수 있는 범위
    - singleton : spring container의 시작부터 종료까지 유지되는 스코프
    - prototype : prototype bean의 생성과 DI까지만 spring container가 관여하고 그 이후는 관리하지 않음
    - 웹 관련
        - request : 웹 요청이 들어오고 나갈 때까지 유지되는 스코프
        - session : 웹 세션의 생성-종료까지 유지되는 스코프
        - application : 웹 서블릿 컨텍스트와 같은 범위로 유지되는 스코프
- Bean scope의 등록 방법
    - Component scan 자동 등록
        
        ```java
        @Scope("prototype")
        @Component
        public class HelloBean {}
        ```
        
    - 수동 등록
        
        ```java
        @Scope("prototype")
        @Bean
        PrototypeBean HelloBean() {
        		return new HelloBean();
        }
        ```
        

## 프로토타입 스코프

### 싱글톤 빈 요청

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-9%20(1).png)

1. 싱글톤 빈을 spring container에 요청
2. container는 관리하고 있던 빈 반환
3. 이후 같은 요청일 경우 같은 인스턴스를 반환

### 프로토타입 빈 요청

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-9%20(2).png)

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-9%20(3).png)

1. 프로토타입 빈을 container에 요청
2. container는 요청을 받았을 때 프로토타입 빈을 생성하고 DI 수행
3. 생성한 프로토타입 빈을 클라이언트에 반환
4. 이후 같은 요청일 경우 매번 새로운 프로토타입 빈을 생성 후 반환
- Spring container는 prototype bean을 생성하고, DI, 초기화까지만 처리함
- 이후의 prototype 빈 관리는 클라이언트가 담당함
    - 서버에서는 `@PreDestroy`와 같은 종료 메소드를 호출할 수 없음
    - 클라이언트가 종료 메소드를 실행해줘야 함
- `test../scope/SingletonTest.java` 생성
    
    ```java
    public class SingletonTest {
        @Test
        void singletonBeanFind() {
            AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SingletonBean.class);
            SingletonBean singletonBean1 = ac.getBean(SingletonBean.class);
            SingletonBean singletonBean2 = ac.getBean(SingletonBean.class);
            System.out.println("singletonBean1 = " + singletonBean1);
            System.out.println("singletonBean2 = " + singletonBean2);
            Assertions.assertThat(singletonBean1).isSameAs(singletonBean2);
    
            ac.close();
        }
    
        @Scope("singleton")
        static class SingletonBean {
            @PostConstruct
            public void init() {
                System.out.println("SingletonBean.init");
            }
            @PreDestroy
            public void destroy() {
                System.out.println("SingletonBean.destroy");
            }
        }
    }
    ```
    
    - 실행 결과
        
        ```java
        SingletonBean.init
        singletonBean1 = hello.core.scope.SingletonTest$SingletonBean@3cdf2c61
        singletonBean2 = hello.core.scope.SingletonTest$SingletonBean@3cdf2c61
        13:30:49.421 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@1b11171f, started on Sun May 28 13:30:49 KST 2023
        SingletonBean.destroy
        ```
        
    - 종료 메소드까지 정상적으로 호출되고, 같은 빈이 조회됨
- `test/../scope/PrototypeTest.java` 생성
    
    ```java
    public class PrototypeTest {
        @Test
        void prototypeBeanFind() {
            AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(PrototypeBean.class);
            System.out.println("find prototypeBean1");
            PrototypeBean prototypeBean1 = ac.getBean(PrototypeBean.class);
            System.out.println("find prototypeBean2");
            PrototypeBean prototypeBean2 = ac.getBean(PrototypeBean.class);
            System.out.println("prototypeBean1 = " + prototypeBean1);
            System.out.println("prototypeBean2 = " + prototypeBean2);
            Assertions.assertThat(prototypeBean1).isNotSameAs(prototypeBean2);
    
            ac.close();
        }
    
        @Scope("prototype")
        static class PrototypeBean {
            @PostConstruct
            public void init() {
                System.out.println("SingletonBean.init");
            }
            @PreDestroy
            public void destroy() {
                System.out.println("SingletonBean.destroy");
            }
        }
    }
    ```
    
    - 실행 결과
        
        ```java
        find prototypeBean1
        SingletonBean.init
        find prototypeBean2
        SingletonBean.init
        prototypeBean1 = hello.core.scope.PrototypeTest$PrototypeBean@3cdf2c61
        prototypeBean2 = hello.core.scope.PrototypeTest$PrototypeBean@13ad5cd3
        13:31:17.988 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@1b11171f, started on Sun May 28 13:31:17 KST 2023
        ```
        
    - 빈 요청이 들어올 때마다 생성메소드가 호출됨
    - 다른 빈이 조회됨
    - 종료 메소드가 실행되지 않음

## 프로토타입 스코프 - 싱글톤 빈과 함께 사용시 문제점

- 싱글톤 빈과 함께 사용하면 의도한 대로 동작하지 않음

### 프로토타입 빈 직접 요청

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-9%20(4).png)

1. client A가 spring container에 prototype bean 요청
2. container는 새로 생성 후 반환(`x01`)
3. A가 `addCount()` 호출
4. B가 prototype bean 요청
5. container는 새로 생성 후 반환(`x02`)
6. B가 `addCount()` 호출
- `test/../scope/SingletonWithPrototypeTest1.java` 생성
    
    ```java
    public class SingletonWithPrototypeTest1 {
        @Test
        void prototypeFind() {
            AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(PrototypeBean.class);
            PrototypeBean prototypeBean1 = ac.getBean(PrototypeBean.class);
            prototypeBean1.addCount();
            Assertions.assertThat(prototypeBean1.getCount()).isEqualTo(1);
            PrototypeBean prototypeBean2 = ac.getBean(PrototypeBean.class);
            prototypeBean2.addCount();
            Assertions.assertThat(prototypeBean2.getCount()).isEqualTo(1);
    
        }
    
        @Scope("prototype")
        static class PrototypeBean {
            private int count = 0;
    
            public void addCount() {
                count++;
            }
    
            public int getCount() {
                return count;
            }
    
            @PostConstruct
            public void init() {
                System.out.println("PrototypeBean.init " + this);
            }
    
            @PreDestroy
            public void destroy() {
                System.out.println("PrototypeBean.destroy");
            }
        }
    
    }
    ```
    

### 싱글톤에서 프로토타입 빈 사용

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-9%20(5).png)

1. `clientBean`은 싱글톤 빈으로, spring container 생성 시점에 생성되고, DI도 발생함
    - DI 시점에 prototype bean을 요청 후 spring container가 생성한 `prototypeBean`을 내부 필드로 보관
2. client A가 `clientBean`을 요청해서 받음
3. `clientBean.logic()` 호출
4. `prototypeBean`의 `addCount()` 호출
5. client B가 `clientBean`을 요청해서 받음(싱글톤이므로 같은 객체)
6. `clientBean.logic()` 호출
7. `prototypeBean`의 `addCount()` 호출
    - `clientBean`의 DI 시점에 생성된 `prototypeBean`을 사용
- `test/../SingletonWithPrototypetest1.java` 수정
    
    ```java
    public class SingletonWithPrototypeTest1 {
    ...
        @Test
        void singletonClientUsePrototype() {
            AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(ClientBean.class, PrototypeBean.class);
            ClientBean clientBean1 = ac.getBean(ClientBean.class);
            int count1 = clientBean1.logic();
            assertThat(count1).isEqualTo(1);
            ClientBean clientBean2 = ac.getBean(ClientBean.class);
            int count2 = clientBean2.logic();
            assertThat(count2).isEqualTo(2);
        }
    
        @Scope("singleton")
        static class ClientBean {
            private final PrototypeBean prototypeBean;
    
            @Autowired
            public ClientBean(PrototypeBean prototypeBean) {
                this.prototypeBean = prototypeBean;
            }
    
            public int logic(){
                prototypeBean.addCount();
                return prototypeBean.getCount();
            }
        }
    ...
    }
    ```
    
    - `ClientBean`의 생성 시점에 같이 생성된 `prototypeBean`을 사용함
- prototype bean은 참조를 통해 사용할 때마다 새로 생성되는 것은 아님

## 프로토타입 스코프 - 싱글톤 빈과 함께 사용시 Provider로 문제 해결

### 스프링 컨테이너에 요청

- singleton이 prototype을 사용할 때마다 새로 요청
- `logic()` 메소드를 아래와 같이 수정하면 됨
    
    ```java
    static class ClientBean {
    
    		@Autowired
    		private ApplicationContext ac;
    
    		public int logic() {
    				PrototypeBean prototypeBean = ac.getBean(PrototypeBean.class);
    				prototypeBean.addCount();
    				int count = prototypeBean.getCount();
    				return count;
    		}
    }
    ```
    
- DL(Dependency Lookup) : 직접 필요한 의존관계를 찾는 작업
    - DI는 의존관계를 외부에서 주입받음
    - `ApplicationContext` 자체를 주입받게 되면 spring container에 종속적인 코드가 되고, 단위 테스트도 어려워짐

### `ObjectFactory`, `ObjectProvider`

- `test/../SingletonWithPrototypeTest1.java` 수정
    
    ```java
    @Test
    void singletonClientUsePrototype() {
    ...
        assertThat(count2).isEqualTo(1);
    }
    
    @Scope("singleton")
    static class ClientBean {
    //        private final PrototypeBean prototypeBean;
    
        @Autowired
        private ObjectProvider<PrototypeBean> prototypeBeanProvider;
    
    //        @Autowired
    //        public ClientBean(PrototypeBean prototypeBean) {
    //            this.prototypeBean = prototypeBean;
    //        }
    
        public int logic(){
            PrototypeBean prototypeBean = prototypeBeanProvider.getObject();
            prototypeBean.addCount();
            return prototypeBean.getCount();
        }
    }
    ```
    
    - `ObjectProvider` 타입을 `ObjectFactory`로 바꿔도 동작함
    - `prototypeBeanProvider.getObject()`를 통해 항상 새로운 prototype bean이 생성됨
        - `ObjectProvider.getObject()` : spring container를 통해 해당 빈을 찾아서 반환
- `ObjectFactory` : spring에 의존
- `ObjectProvider` : `ObjectFactory`를 상속함, 스트림 처리 등의 부가 기능이 많음, spring에 의존

### JSR-330 Provider

- `javax.inject:javax.inject:1` 라이브러리를 gradle 파일에 추가해야 함
    
    ```java
    dependencies {
    	implementation 'javax.inject:javax.inject:1'
    ...
    ```
    
- `test/../SingletonWithPrototypeTest1.java` 수정
    
    ```java
    @Scope("singleton")
    static class ClientBean {
    
        @Autowired
        private Provider<PrototypeBean> prototypeBeanProvider;
    
        public int logic(){
            PrototypeBean prototypeBean = prototypeBeanProvider.get();
            prototypeBean.addCount();
            return prototypeBean.getCount();
        }
    }
    ```
    
    - `provider.get()` 또한 `ObjectProvider.getObject()`와 동일하게 새로운 prototype bean이 생성됨
- `get()` 메소드 하나만 정의되어 있을 정도로 기능이 단순함
- 별도의 라이브러리가 필요함
- 자바 표준이기 때문에 spring이 아닌 곳에서도 사용 가능

### 정리

- prototype bean 사용 예시 : 사용할 때마다 DI 주입이 완료된 새로운 객체가 필요한 경우
    - 대부분의 경우 싱글톤으로 사용 가능함
- `ObjectProvider`, `JSR-330 Provider` 등은 prototype 이외에도 DL이 필요할 경우 사용 가능
    - `@Lookup` 어노테이션도 있긴 함
    - 대부분의 경우 `ObjectProvider`가 낫고, spring이 아닌 다른 컨테이너에서도 사용할 수 있어야 한다면 `JSR-330 Provider`를 사용해야 함
        - 다른 기능들도 자바 표준과 스프링이 겹칠 때가 많은데, 위와 마찬가지로 다른 컨테이너를 사용하지 않는다면 스프링 추천(`@PostConstruct`, `@PreDestroy`와 같은 어노테이션은 스프링에서도 표준을 권장하는 경우)

## 웹 스코프

- 웹 환경에서만 동작함
- spring이 해당 스코프의 종료 시점까지 관리함
    - 종료 메소드가 호출됨

### 웹 스코프 종류

- `request` : HTTP 요청 하나가 들어오고 나갈 때까지 유지되는 스코프
    - 각각의 요청마다 bean instance가 생성됨
- `session` : HTTP Session과 동일한 생명주기를 가지는 스코프
- `application` : `ServletContext`와 동일한 생명 주기를 가지는 스코프
- `websocket` : 웹 소켓과 동일한 생명 주기를 가지는 스코프

### HTTP request 요청 당 각각 할당되는 request 스코프

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-9%20(6).png)

- A의 요청에 의해 만들어진 request bean은 이후 A가 요청할 때마다 호출됨

## request 스코프 예제 만들기

- `build.gradle`에 web 라이브러리 추가
    
    ```java
    implementation 'org.springframework.boot:spring-boot-starter-web'
    ```
    
    - web 라이브러리가 없으면 `AnnotationConfigApplicationContext`를 기반으로 앱 구동
    - web 라이브러리를 추가하면 `AnnotationConfigServletWebServerApplicationContext`를 기반으로 앱 구동
- `8080` 포트 사용 중일 경우 `application.properties` 파일에 `server.port=9090` 넣어서 포트 변경

### request 스코프 예제 개발

- `../common/MyLogger.java` 생성
    
    ```java
    @Component
    @Scope(value = "request")
    public class MyLogger {
        
        private String uuid;
        private String requestURL;
    
        public void setRequestURL(String requestURL) {
            this.requestURL = requestURL;
        }
        
        public void log(String message) {
            System.out.println("[" + uuid + "][" + requestURL + "] " + message);
        }
        
        @PostConstruct
        public void init() {
            uuid = UUID.randomUUID().toString();
            System.out.println("[" + uuid + "] request scope bean create:" + this);
        }
        
        @PreDestroy
        public void close() {
            System.out.println("[" + uuid + "] request scope bean close:" + this);
        }
    }
    ```
    
    - `requestURL`은 bean이 생성되는 시점에 알 수 없기 때문에 외부에서 seter로 입력받음
- `../web/LogDemoController.java` 생성
    
    ```java
    @Controller
    @RequiredArgsConstructor
    public class LogDemoController {
    
        private final LogDemoService logDemoService;
        private final MyLogger myLogger;
    
        @RequestMapping("log-demo")
        @ResponseBody
        public String logDemo(HttpServletRequest request) {
            String requestURL = request.getRequestURL().toString();
            myLogger.setRequestURL(requestURL);
    
            myLogger.log("controller test");
            logDemoService.logic("testId");
            return "OK";
        }
    }
    ```
    
    - 원래는 로거는 공통 처리가 가능한 스프링 인터셉터나 서블릿 필터에 구현하는게 좋음
- `../web/LogDemoService.java` 생성
    
    ```java
    @Controller
    @RequiredArgsConstructor
    public class LogDemoController {
    
        private final LogDemoService logDemoService;
        private final MyLogger myLogger;
    
        @RequestMapping("log-demo")
        @ResponseBody
        public String logDemo(HttpServletRequest request) {
            String requestURL = request.getRequestURL().toString();
            myLogger.setRequestURL(requestURL);
    
            myLogger.log("controller test");
            logDemoService.logic("testId");
            return "OK";
        }
    }
    ```
    
    - request scope를 사용하지 않고, 파라미터로 정보를 넘길 수도 있는데, 관련 없는 정보까지 서비스 계층으로 넘어감
        - 웹과 관련된 부분은 컨트롤러까지에서만 사용돼야 함
        - 서비스 계층은 웹 기술에 종속되지 않는 것이 좋음
    - `myLogger`는 request scope이기 때문에 요청이 들어와야 생성되는데, `LogDemoController`가 처음 singleton 빈으로 등록되는 과정에서 DI가 필요하기 때문에 오류가 남

## 스코프와 Provider

- `Provider`를 사용해서 해결 가능
- `../web/LogDemoController.java` 수정
    
    ```java
    @Controller
    @RequiredArgsConstructor
    public class LogDemoController {
    ...
        private final ObjectProvider<MyLogger> myLoggerProvider;
    
        @RequestMapping("log-demo")
        @ResponseBody
        public String logDemo(HttpServletRequest request) {
            String requestURL = request.getRequestURL().toString();
            MyLogger myLogger = myLoggerProvider.getObject();
            myLogger.setRequestURL(requestURL);
    ...
        }
    }
    ```
    
- `../web/LogDemoService.java` 수정
    
    ```java
    @Service
    @RequiredArgsConstructor
    public class LogDemoService {
    
        private final ObjectProvider<MyLogger> myLoggerProvider;
    
        public void logic(String id) {
            MyLogger myLogger = myLoggerProvider.getObject();
            myLogger.log("service id = " + id);
        }
    }
    ```
    
- 실행 결과
    
    ```java
    [8624c07a-a1cc-4133-8416-ea3c62398f4a] request scope bean create:hello.core.common.MyLogger@7344b0f7
    [8624c07a-a1cc-4133-8416-ea3c62398f4a][http://localhost:8080/log-demo] controller test
    [8624c07a-a1cc-4133-8416-ea3c62398f4a][http://localhost:8080/log-demo] service id = testId
    [8624c07a-a1cc-4133-8416-ea3c62398f4a] request scope bean close:hello.core.common.MyLogger@7344b0f7
    ```
    
    - `ObjectProvider`를 사용하면 `getObject()` 메소드 호출 시점까지 request scope 빈의 등록을 지연할 수 있음
    - `ObjectProvider.getObject()`를 호출하는 시점에는 request가 진행중이기 때문에 request scope 빈이 정상적으로 생성됨
    - `ObjectProvider.getObject()`를 다른 클래스들에서 호출해도, 같은 요청이면 같은 빈이 반환됨

## 스코프와 프록시

- `LogDemoController.java`, `LogDemoService.java` 모두 `Provider` 사용하기 전으로 수정
- `../common/MyLogger.java` 수정
    
    ```java
    @Component
    @Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
    public class MyLogger {
    ...
    ```
    
    - 위와 동일하게 정상적으로 동작
    - 적용 대상이 interface이면 `ScopedProxyMode.INTERFACES`
    - 적용 대상이 class면 `ScopedProxyMode.TARGET_CLASS`
- `LogDemoController`, `LogDemoService`에서 주입된 `myLogger`를 `myLogger.getClass()`로 출력하면 `class hello.core.common.MyLogger$$EnhancerBySpringCGLIB$a2b3d298`과 같이 나옴

### 웹 스코프와 프록시 동작 원리

- `proxyMode` 옵션을 사용하면 spring container는 CGLIB을 사용해 `MyLogger`를 상속받는 프록시 객체를 생성하고 빈으로 등록함
    - `ac.getBean("myLogger", MyLogger.class)`로 조회하면 프록시 객체가 조회됨
    - DI도 프록시 객체가 주입됨

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-9%20(7).png)

- 프록시 객체는 빈 요청이 오면 그때 진짜 빈을 요청하는 위임 로직을 포함함
    - 원본 객체를 상속받기 때문에 다형성을 가짐
    - request scope와는 관계 없이 싱글톤처럼 동작함

### 정리

- `Provider`, 프록시 둘 다 객체 조회를 필요한 시점까지 지연시킬 수 있는 방법들임
    - web scope가 아니더라도 사용 가능
- 프록시는 싱글톤을 사용하는 코드와 유사함
    - 어노테이션만 변경해서 원본 객체를 프록시 객체로 대체 가능