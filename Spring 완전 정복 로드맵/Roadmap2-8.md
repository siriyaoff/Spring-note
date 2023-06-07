# 섹션 8. 빈 생명주기 콜백

## 빈 생명주기 콜백 시작

- 실제로는 앱이 시작될 때 미리 DB 커넥션 풀이나 네트워크 소켓을 연결해두고, 필요할 때마다 이미 연결되어 있는 객체를 사용해서 성능을 향상시킴
- `test/../lifecycle/NetworkClient.java` 생성
    - 외부 네트워크에 미리 연결하는 객체를 생성한다고 가정
    
    ```java
    public class NetworkClient {
    
        private String url;
    
        public NetworkClient() {
            System.out.println("생성자 호출, url = " + url);
            connect();
            call("Init connect msg");
        }
    
        public void setUrl(String url){
            this.url = url;
        }
    
        // 서비스 시작 시 호출
        public void connect(){
            System.out.println("connect: " + url);
        }
    
        public void call(String message) {
            System.out.println("call: " + url + " message = " + message);
        }
    
        // 서비스 종료 시 호출
        public void disconnect() {
            System.out.println("close: " + url);
        }
    }
    ```
    
- `test/../lifecycle/BeanLifeCycleTest.java` 생성
    
    ```java
    public class BeanLifeCycleTest {
        
        @Test
        public void lifeCycleTest() {
            ConfigurableApplicationContext ac = new AnnotationConfigApplicationContext(LifeCycleConfig.class);
            NetworkClient client = ac.getBean(NetworkClient.class);
            ac.close();
        }
    
        @Configuration
        static class LifeCycleConfig {
            @Bean
            public NetworkClient networkClient() {
                NetworkClient networkClient = new NetworkClient();
                networkClient.setUrl("http://hello-spring.dev");
                return networkClient;
            }
        }
        
    }
    ```
    
    - `ApplicationContext`는 `close()` 메소드가 없기 때문에 `ac`를 `ConfigurableApplicationContext`로 선언해야 함
    - 결과
        
        ```
        생성자 호출, url = null
        connect: null
        call: null message = Init connect msg
        ```
        
        - `NetworkClient` 객체를 생성하는 과정에서는 `url`이 설정되지 않았기 때문에 `null`이 출력됨
        - 생성자에 넣어놨던 초기화 작업이 예상대로 이루어지지 않음

### Spring Bean의 이벤트 라이프사이클

1. Spring Container 생성
2. Spring Bean 생성
3. 의존관계 주입
    - 생성자 주입의 경우 Bean이 생성되면서 동시에 의존 관계가 주입됨
4. 초기화 콜백
5. 사용
6. 소멸전 콜백
7. Spring 종료
- 싱글톤 빈은 spring container와 life cycle이 같기 때문에 spring container가 종료되기 직전에 소멸전 콜백이 실행됨
    - life cycle이 다른 bean들은 bean이 종료되기 직전에 소멸전 콜백이 실행됨
- 객체의 생성과 초기화는 웬만하면 분리하는게 좋음
    - 객체의 생성 : 생성자를 통해 파라미터를 받고 메모리 할당 후 객체 생성
    - 객체의 초기화 : 생성된 필드를 사용하는, 외부와의 연결 등의 무거운 작업

### Spring Lifecycle callback 종류

- `InitializingBean`, `DisposableBean` Interface
- config에 초기화 메소드, 종료 메소드 지정
- `@PostConstruct`, `@PreDestroy` 어노테이션

## 인터페이스 `InitializingBean`, `DisposableBean`

- `NetworkClient.java`를 interface `InitializingBean`, `DisposableBean`의 구현체로 설정
    
    ```java
    public class NetworkClient implements InitializingBean, DisposableBean {
    ...
        @Override
        public void afterPropertiesSet() throws Exception {
            connect();
            call("Initial connect message");
        }
    
        @Override
        public void destroy() throws Exception {
            disconnect();
        }
    }
    ```
    
    - `afterPropertiesSet()` : `InitializingBean` interface의 메소드
    - `destroy()` : `DisposableBean` interface의 메소드
    - `BeanLifeCycleTest.java` 실행 결과
        
        ```java
        생성자 호출, url = null
        connect: null
        call: null message = Init connect msg
        connect: http://hello-spring.dev
        call: http://hello-spring.dev message = Initial connect message
        22:15:07.049 [main] DEBUG org.springframework.context.annotation.AnnotationConfigApplicationContext - Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@659499f1, started on Fri May 26 22:15:06 KST 2023
        close: http://hello-spring.dev
        ```
        
- 초기화, 소멸 interface의 단점
    - Spring 전용 interface이기 때문에, 스프링 의존도가 높아짐
    - override 하는 형식이기 때문에 메소드의 이름을 변경할 수 없음
    - 외부 라이브러리에서는 적용할 수 없음
- 초창기에 나온 방법으로, 현재는 거의 사용하지 않음

## 빈 등록 초기화, 소멸 메서드 지정

- `NetworkClient.java` 가 구현하는 인터페이스 제거, 메소드들은 초기화, 소멸 메소드로 지정
    
    ```java
    public class NetworkClient {
    ...
        public void init() {
            System.out.println("NetworkClient.init");
            connect();
            call("Initial connect message");
        }
    
        public void close() {
            System.out.println("NetworkClient.close");
            disconnect();
        }
    }
    ```
    
- `BeanLifeCycleTest.java` config 수정
    
    ```java
    public class BeanLifeCycleTest {
    ...
        @Configuration
        static class LifeCycleConfig {
            @Bean(initMethod = "init", destroyMethod = "close")
            public NetworkClient networkClient() {
                NetworkClient networkClient = new NetworkClient();
                networkClient.setUrl("http://hello-spring.dev");
                return networkClient;
            }
        }
    }
    ```
    
- 결과는 위와 같음
- config 사용할 경우의 특징
    - 메소드 이름을 자유롭게 지정 가능
    - 스프링 코드에 의존하지 않음
    - config를 사용하기 때문에 외부 라이브러리에 대해서도 초기화, 종료 메소드를 적용시킬 수 있음
- `@Bean`의 `destroyMethod`는 기본값이 `"(inferred)"`로 설정되어 있음
    - 대부분의 라이브러리에서 `close`, `shutdown` 등의 이름을 종료 메소드로 사용하는데, `destroyMethod`가 기본값으로 설정된 경우 이런 이름의 메소드를 자동으로 호출함
    - 따라서 `@Bean`을 이용해 spring bean으로 등록할 경우 종료 메소드는 명시하지 않아도 잘 동작함
    - `destroyMethod=""`으로 명시해서 추론 기능을 사용하지 않을 수 있음

## 어노테이션 `@PostConstruct`, `@PreDestroy`

- `BeanLifeCycleTest.java`에서 `@Bean` 뒤의 `initMethod`, `destroyMethod` 설정 삭제
- `NetworkClient.java` 수정
    
    ```java
    public class NetworkClient {
    ...
        @PostConstruct
        public void init() {
            System.out.println("NetworkClient.init");
            connect();
            call("Initial connect message");
        }
    
        @PreDestroy
        public void close() {
            System.out.println("NetworkClient.close");
            disconnect();
        }
    }
    ```
    
    - 실행 결과는 위와 같음
    - 두 어노테이션 모두 `javax.annotation.*`이기 때문에 스프링에 종속적이지 않고 다른 container에서도 동작함
- 메소드에 직접 어노테이션을 붙여야 하기 때문에 외부 라이브러리에 적용하지 못함
    - 외부 라이브러리는 `initMethod`, `destroyMethod`를 사용해야 함
- 스프링에서 가장 권장하는 방법임