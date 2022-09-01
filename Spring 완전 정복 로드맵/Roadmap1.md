# 스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술

> **섹션 1. 프로젝트 환경설정**
> 

## 프로젝트 생성

[Spring Initializr](https://start.spring.io/)

- 프로젝트 선택
    - Project : Gradle Project
    - Spring Boot : 2.3.x
    - Language : Java
    - Packaging : Jar
    - Java : 11
- Project Metadata
    - Group : hello
    - Artifact : hello-spring
- Dependencies : Spring Web, Thymeleaf
    - Thymeleaf : html template engine

### IntelliJ에서

- src
    - main - java : 메인 코드
    - test : 테스트 코드
    - build.gradle : gradle 전체 설정
        - gradle : 버전 설정, 라이브러리 import하는 버전 관리 도구
        - sourceCompatibility 11로 바꿔야 함
- IntelliJ 설정
    - project structure
        - SDK : 11
        - language level : SDK default
    - Settings - Build, Execution, Deployment - Build Tools - Gradle
        - Build and run : IntelliJ IDEA
        - Gradle - Gradle JVM : 11

## 라이브러리 살펴보기

- 탐색기 - external libraries에서 볼 수 있음
- 오른쪽 gradle에서 dependency 확인 가능

### spring boot 라이브러리

- spring-boot-stareter-web
    - spring-boot-starter-tomcat : 웹서버 관련
    - spring-webmvc : spring web MVC
- spring-boot-starter-thymeleaf : thymeleaf 템플릿 엔진(View)
- spring-boot-starter(공통) : 스프링 부트 + 스프링 코어 + 로깅
    - spring-boot-starter-logging : logback, slf4j

### 테스트 라이브러리

- spring-boot-starter-test
    - junit : 테스트 framework
    - mockito : mock library
    - assertj : 테스트 코드 작성을 도와줌
    - spring-test : 스프링 통합 테스트 지원

## View 환경설정

### welcome page

- `src/main/resources/static/index.html` 으로 index.html 생성

```html
<!DOCTYPE HTML>
<html>
<head>
 <title>Hello</title>
 <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
Hello
<a href="/hello">hello</a>
</body>
</html>
```

- spring boot Welcome Page : static 폴더 안에서 `index.html`을 찾고, 없으면 `index` template을 찾음
찾게되면 welcome page로 사용함

### template engine

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(1).png)

- `hello/hellospring`에 `controller/HelloController.java` 추가
    
    ```java
    // HelloController.java
    package hello.hellospring.controller;
    
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.GetMapping;
    
    @Controller
    public class HelloController {
        @GetMapping("hello")
        public String hello(Model model) {
            model.addAttribute("data", "hello!!");
            return "hello";
        }
    }
    ```
    
- `resources/templates`에 `hello.html` 추가
    
    ```html
    <!DOCTYPE HTML>
    <html xmlns:th="http://www.thymeleaf.org">
    <head>
     <title>Hello</title>
     <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    </head>
    <body>
    <p th:text="'안녕하세요. ' + ${data}" >안녕하세요. 손님</p>
    </body>
    </html>
    ```
    

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(2).png)

- controller에서의 리턴값과 일치하는 파일을 `resources/templates`에서 찾아서 템플릿 엔진으로 처리함
- controller가 넘겨준 모델을 사용해서 viewResolver가 템플릿을 적용하고 렌더링함
- 위에서는 `localhost:8080/hello`를 `public String hello(Model model)`과 매핑하고, `data: "hello!!"` attribute를 추가한 다음 `template/hello.html` 템플릿으로 넘겨줌
    - viewResolver가 넘겨받은 attr를 사용해서 페이지 렌더링함
    - 기본 매핑 방법 : `resources/templates/{ViewName}.html`

> spring-boot-devtools 라이브러리를 추가하면 html만 컴파일해서 View 파일 변경 가능
> 

## 빌드하고 실행하기

- `hello-spring/`에서 `gradlew build` 실행해서 빌드 가능
    - 맥은 `./gradlew build`
- `hello-spring/build/libs`에 스냅샷 파일 생성됨
    - `java -jar hello-spring-0.0.1-SNAPSHOT.jar`로 실행

---

> **섹션 2. 스프링 웹 개발 기초**
> 
- 웹 개발 방법
    - 정적 컨텐츠 : 파일을 그대로 웹브라우저에 렌더링
    - MVC와 템플릿 엔진 : 템플릿 엔진(JSP, PHP 등)을 통해 html 파일을 동적으로 바꿔서 렌더링
        - controller, model, template engine(view)이 필요함
        → MVC 패턴
    - API : json으로 client에게 데이터 전달
        
        e.g. vue, react에서도 json으로 데이터만 내려주고 client에서 페이지를 렌더링하는 방식으로 구현 가능
        

## 정적 컨텐츠

- spinrg boot는 기본적으로 static content으로 구현됨
- `resources/static/hello-static.html` 생성
    
    ```html
    <!DOCTYPE HTML>
    <html>
    <head>
     <title>static content</title>
     <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    </head>
    <body>
    정적 컨텐츠 입니다.
    </body>
    </html>
    ```
    
    - `localhost:8080/hello-static.html`으로 접속 가능

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(3).png)

- controller에 hello-static 관련 매핑이 없기 때문에 static에서 찾아서 렌더링함

## MVC와 템플릿 엔진

- MVC : Model, View, Controller
    - Controller : 서버와 통신, 비즈니스 로직 등
    - View : 화면을 렌더링하는데 집중
- `HelloController.java`에 아래 추가:

```java
@GetMapping("hello-mvc")
public String helloMvc(@RequestParam("name") String name, Model model) {
    model.addAttribute("name", name);
    return "hello-template";
}
```

- `resources/templates/hello-template.html`

```html
<html xmlns:th="http://www.thymeleaf.org">
<body>
<p th:text="'hello ' + ${name}">hello! empty</p>
</body>
```

- `http://localhost:8080/hello-mvc?name=sdlf`으로 접속 가능

## API

- 정적 컨텐츠를 제외하면, html로 내리냐, 데이터만 내리냐를 선택하면 됨
- `@ResponseBody` : html 파일을 리턴값으로 바꿈
- `HelloController.java`에 아래 내용 추가:
    
    ```java
    @GetMapping("hello-api")
    @ResponseBody
    public Hello helloApi(@RequestParam("name") String name){
        Hello hello = new Hello();
        hello.setName(name);
        return hello;
    }
    
    static class Hello {
        private String name;
        public String getName(){
            return name;
        }
        public void setName(String name){
            this.name=name;
        }
    }
    ```
    
    - `@ResponseBody`를 사용하고 객체를 반환하면 객체가 JSON으로 반환됨
    - `http://localhost:8080/hello-api?name=asdf`로 접속 가능
        
        ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(4).png)
        
    - `alt+Insert`로 getter and setter 자동 생성 가능
    - getter, setter 쓰는건 javabean 표준임
    

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(5).png)

- `@ResponseBody` : HTTP BODY에 문자 내용을 직접 반환
    - viewResolver 대신에 HttpMessageConverter 가 동작
    - 리턴값이 문자일 경우 : StringHttpMessageConverter
    - 리턴값이 객체일 경우 : MappingJackson2HttpMessageConverter
    - byte 등 기타 여러 HttpMessageConverter도 존재함
    - client의 HTTP Accept 헤더와, server에서 controller의 리턴 타입 정보를 조합해서 HttpMessageConverter가 선택됨

---

> **섹션 3. 회원 관리 예제 - 백엔드 개발**
> 

## 비즈니스 요구사항 정리

- 데이터 : 회원 ID, 이름
- 기능 : 회원 등록, 조회
- DB는 아직 선정되지 않은 상태라 가정

### 일반적인 웹 어플리케이션 구조

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(6).png)

- 컨트롤러 : 웹 MVC의 컨트롤러 역할
- 서비스 : 핵심 비즈니스 로직 구현
- 리포지토리 : DB 접근, 도메인 객체를 DB에 저장하고 관리
- 도메인 : 비즈니스 도메인 객체
e.g. 회원, 주문, 쿠폰 등등 주로 DB에 저장하고 관리됨

### 클래스 의존관계

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(7).png)

- DB가 아직 선정되지 않았기 때문에 MemberRepository는 interface로 설계
    - RDB, NoSQL 등으로 선정할 수 있음
    - 우선 메모리 기반의 DB로 구현함

## 회원 도메인과 리포지토리 만들기

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(8).png)

- `hello.hellospring` 안에 package `domain`, `respository` 생성
    - new → package로 생성 가능
- domain에는 [Member.java](http://Member.java) 생성
    
    ```java
    package hello.hellospring.domain;
    
    public class Member{
        private Long id;
        private String name;
    
        public Long getId() {
            return id;
        }
    
        public void setId(Long id) {
            this.id = id;
        }
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    }
    ```
    
- repository에는 interface MemberRepository, class MemoryMemberRepository 생성
    
    ```java
    // MemberRepository.java
    package hello.hellospring.repository;
    
    import hello.hellospring.domain.Member;
    
    import java.util.List;
    import java.util.Optional;
    
    public interface MemberRepository {
        Member save(Member member);
        Optional<Member> findById(Long id);
        Optional<Member> findByName(String name);
        List<Member> findAll();
    }
    
    // MemoryMemberRepository.java
    package hello.hellospring.repository;
    
    import hello.hellospring.domain.Member;
    
    import java.util.*;
    
    public class MemoryMemberRepository implements MemberRepository {
        private static Map<Long, Member> store=new HashMap<>();
        private static long sequence=0L;
    
        @Override
        public Member save(Member member) {
            member.setId(++sequence);
            store.put(member.getId(), member);
            return member;
        }
    
        @Override
        public Optional<Member> findById(Long id) {
            return Optional.ofNullable(store.get(id));
        }
    
        @Override
        public Optional<Member> findByName(String name) {
            return store.values().stream()
                    .filter(member -> member.getName().equals(name))
                    .findAny();
        }
    
        @Override
        public List<Member> findAll() {
            return new ArrayList<>(store.values());
        }
    }
    ```
    
    - new → Java Class에서 클래스, interface 생성 가능
    - `MemoryMemberRepository` 구현부 안에서 `alt + Insert`로 메소드 구현부 자동 추가 가능
    - 리턴값이 `null`을 반환할 수도 있을 때 `Optional.ofNullable` 으로 감싸주면 클라이언트에서 `null` 값을 처리할 수 있음
    - `findByName`의 리턴은 람다로 처리함

## 회원 리포지토리 테스트 케이스 작성

- main이나 controller로 테스트하는 것은 반복 실행이 불가능하고 준비가 오래 걸림
→ JUnit으로 편하게 테스트 가능
- `src/test/java/hello.hellospring`에 package `repository` 생성 후 `repository/MemoryMemberRepositoryTest` 클래스 생성
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(9).png)
    
    ```java
    package hello.hellospring.repository;
    
    import hello.hellospring.domain.Member;
    import org.junit.jupiter.api.AfterEach;
    import org.junit.jupiter.api.Test;
    
    import java.util.List;
    
    import static org.assertj.core.api.Assertions.assertThat;
    
    class MemoryMemberRepositoryTest {
        MemoryMemberRepository repository = new MemoryMemberRepository();
    
        @AfterEach
        public void afterEach() {
            repository.clearStore();
        }
    
        @Test
        public void save(){
            Member member = new Member();
            member.setName("spring");
    
            repository.save(member);
    
            Member result = repository.findById(member.getId()).get();
            //System.out.println("result = " + (result == member));
            //Assertions.assertEquals(member, result); // junit의 Assertions
            assertThat(member).isEqualTo(result); // assertj의 Assertions
        }
    
        @Test
        public void findByName() {
            Member member1 = new Member();
            member1.setName("spring1");
            repository.save(member1);
    
            Member member2 = new Member();
            member2.setName("spring2");
            repository.save(member2);
    
            Member result = repository.findByName("spring1").get();
            assertThat(result).isEqualTo(member1);
        }
    
        @Test
        public void findAll(){
            Member member1 = new Member();
            member1.setName("spring1");
            repository.save(member1);
    
            Member member2 = new Member();
            member2.setName("spring2");
            repository.save(member2);
    
            List<Member> result = repository.findAll();
    
            assertThat(result.size()).isEqualTo(2);
        }
    }
    ```
    
    - `Alt + Enter`로 static import하면 패키지 이름 생략 가능
    - 테스트 케이스들은 각각 독립적이게 설계되어야 함
        - 테스트 순서는 항상 일정하게 보장되지 않음(아래는 `@AfterEach`를 넣지 않았을 때 `MemoryMemberRepositoryTest` 클래스 테스트 돌린 결과)
            
            ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(10).png)
            
            - `MemberMemoryRepository repository`가 초기화되지 않아 두 번째 테스트인 `findByName`가 첫 번째 테스트에서 생성된 member들의 영향을 받음
    - `@AfterEach`로 테스트 하나가 끝날 때마다 실행될 내용 구현 가능
    - `MemoryMemberRepository` 클래스에 아래 메소드 추가:
        
        ```java
        public void clearStore() {
            store.clear();
        }
        ```
        
    - `gradlew test`로 테스트 실행 가능
    - TDD(Test-driven development) : 테스트 클래스를 먼저 작성하고 앱을 구현하는 방법

## 회원 서비스 개발

- `main/java/hello.hellospring`에 package `service` 생성 후 `service/HelloSpringApplication` 클래스 생성
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(11).png)
    
    ```java
    package hello.hellospring.service;
    
    import hello.hellospring.domain.Member;
    import hello.hellospring.repository.MemberRepository;
    import hello.hellospring.repository.MemoryMemberRepository;
    
    import java.util.List;
    import java.util.Optional;
    
    public class MemberService {
        private final MemberRepository memberRepository = new MemoryMemberRepository();
    
        /**
         * 회원 가입
         */
        public Long join(Member member) {
            validateDuplicateMember(member);
            memberRepository.save(member);
            return member.getId();
        }
    
        private void validateDuplicateMember(Member member) {
            // 같은 이름이 있는 중복 회원이 존재하면 안됨
            memberRepository.findByName(member.getName())
                    .ifPresent(m -> {
                        throw new IllegalStateException("이미 존재하는 회원입니다.");
                    });
            /*
            Optional<Member> result = memberRepository.findByName(member.getName());
            result.ifPresent(m -> {
                throw new IllegalStateException("이미 존재하는 회원입니다.");
            });
            */
        }
    
        /**
         * 전체 회원 조회
         */
        public List<Member> findMembers() {
            return memberRepository.findAll();
        }
    
        public Optional<Member> findOne(Long memberId) {
            return memberRepository.findById(memberId);
        }
    }
    ```
    
- `null`일 가능성이 있으면 `Optional`로 감싸서 chain rule으로 처리 가능
    - 기존에는 if문을 사용해서 처리해야 함
- service 클래스는 비즈니스 로직을 처리하기 때문에 네이밍도 비즈니스 용어에 가깝게 사용해야 함
e.g. join, findMembers, …
c.f. repository 클래스 : findById, findByName, …

## 회원 서비스 테스트

- `Ctrl + Shift + T`로 테스트 생성 가능

```java
// MemberService
package hello.hellospring.service;

import hello.hellospring.domain.Member;
import hello.hellospring.repository.MemoryMemberRepository;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.api.Assertions.assertThrows;

class MemberServiceTest {

    MemberService memberService;
    MemoryMemberRepository memberRepository;

    @BeforeEach
    public void beforeEach() {
        memberRepository = new MemoryMemberRepository();
        memberService = new MemberService(memberRepository);
    }
    @AfterEach
    public void afterEach() {
        memberRepository.clearStore();
    }

    @Test
    void join() {
        // given
        Member member = new Member();
        member.setName("spring");

        // when
        Long savedId = memberService.join(member);

        // then
        Member findMember = memberService.findOne(savedId).get();
        assertThat(member.getName()).isEqualTo(findMember.getName());
    }

    @Test
    public void dupMemEXC() {
        // given
        Member member1 = new Member();
        member1.setName("spring");
        Member member2 = new Member();
        member2.setName("spring");

        // when
        memberService.join(member1);
        IllegalStateException e = assertThrows(IllegalStateException.class, () -> memberService.join(member2));
        assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");
//        try {
//            memberService.join(member2);
//            fail();
//        } catch (IllegalStateException e) {
//            assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");
//        }

        // then
    }
    @Test
    void findMembers() {
    }

    @Test
    void findOne() {
    }
}
```

- 테스트는 한글로 바꿔도 됨
- 빌드할 때 테스트 코드는 포함되지 않음
- MMR의 hashmap이 static이기 때문에, 아래와 같이 test class에서 MMR을 새로 만들어도 MemberService 내부의 MMR과 DB가 공유됨:
    
    ```java
    // MemberServiceTest.java
    class MemberServiceTest {
    
        MemberService memberService = new MemberService();
        MemoryMemberRepository memberRepository = new MemoryMemberRepository();
    
        @AfterEach
    		public void afterEach() { memberRepository.clearStore(); }
    		...
    }
    
    // MemberService.java
    public class MemberService {
        private final MemberRepository memberRepository = new MemoryMemberRepository();
    		
    		/**
         * 회원 가입
         */
    		...
    }
    ```
    
    - `afterEach()`를 실행하면 `memberService`의 DB도 지워짐
    → 테스트 클래스가 정상적으로 테스트 되지만, MMR의 map 선언에서 static을 빼면 오류남
        - MMR 별로 다른 DB를 가져야 하기 때문에 원래 static이 빠지는게 정상적인 구조임
    - 서로 독립적인 DB를 가지는 `MemberService` 인스턴스들을 생성할 수 없음
- 아래와 같이 `MemberService`클래스의  `memberRepository`를 생성자에서 초기화하도록 바꿔줌(Dependency Injection)
    
    ```java
    // MemberServiceTest.java
    class MemberServiceTest {
    
        MemberService memberService;
        MemoryMemberRepository memberRepository;
    
        @BeforeEach
        public void beforeEach() {
            memberRepository = new MemoryMemberRepository();
            memberService = new MemberService(memberRepository);
        }
    
        @AfterEach
    		public void afterEach() { memberRepository.clearStore(); }
    		...
    }
    
    // MemberService.java
    public class MemberService {
        private final MemberRepository memberRepository;
    
        public MemberService(MemberRepository memberRepository) {
            this.memberRepository = memberRepository;
        }
    		
    		/**
         * 회원 가입
         */
    		...
    }
    ```
    
    - MMR의 hashmap의 선언에서 static을 빼도 test가 독립적으로 동작함
        - 서로 독립적인 DB를 가지는 `MemberService` 인스턴스들을 생성할 수 있음(static을 뺄 경우)
        강의에서는 static을 빼지 않았기 때문에 아직 MMR의 map이 static으로 선언된 상태임
    - 테스트 클래스에서는 하나의 테스트가 끝날 때마다 `memberRepository`를 초기화하고, 시작 전에 초기화된 `memberRepository`로 `memberService`를 초기화함(`afterEach`도 `beforeEach`로 붙일 수 있음)
- 클래스에 `final` 붙이면 변경 불가능이 아니라 상속할 수 없게됨

---

> **섹션 4. 스프링 빈과 의존관계**
> 

### 스프링 빈 등록 방법 2가지

1. component scan과 자동 의존관계 설정
    - `@Component` annotation 사용
        - `@Controller`, `@Service`, `@Repository` 등의 annotation은 `@Component`를 포함하기 때문에 spring bean으로 등록됨
            - `@Controller` : 외부 요청 받음
            - `@Service` : 비즈니스 로직 만듦
            - `@Repository` : 데이터 저장
    - `@Autowired`로 component bean 간의 의존관계 설정
    - main method가 있는 패키지의 하위 패키지까지만 component scan의 범위에 포함됨
    - 기본적으로 spring bean은 singleton으로 등록됨(singleton이 아니도록 설정은 가능)
        
        → 같은 spring bean이면 같은 instance임
        
2. 자바 코드로 직접 spring bean 등록

## 컴포넌트 스캔과 자동 의존관계 설정

- `controller/MemberController` 클래스 생성

```java
package hello.hellospring.controller;

import hello.hellospring.service.MemberService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;

@Controller
public class MemberController {
    private final MemberService memberService;

    @Autowired
    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }
}
```

- `@Autowired` : 생성자와 연관된 객체인 argument(`memberService`)를 spring container에서 가져옴
    - 의존 관계를 외부(spring)에서 넣어줌 → DI
    - spring bean 끼리 연결시킬 때 사용
    - 생성자가 하나 뿐이라면 `@Autowired` 생략 가능
- 실행하면 `hello.hellospring.service.MemberService`가 없다고 나옴
spring bean을 등록해야 함
- 아래와 같이 `MemberService`, `MemoryMemberRepository`도 spring bean 등록해줌:
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(12).png)
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(13).png)
    
    - spring bean : spring container에 의해 관리되는 객체
    → 공용으로 사용할 수 있음
    - 위처럼 annotation이 있으면 자동으로 스프링 빈으로 등록됨
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(14).png)
    

## 자바 코드로 직접 스프링 빈 등록하기

- `MemberService`, `MemoryMemberRepository`의 annotation들(`@Service`, `@Repository`, `@Autowired`)을 제거하고 진행
    - `MemberController`의 annotation은 그대로 둠
- `hello.hellospring/SpringConfig` 클래스 생성

```java
package hello.hellospring;

import hello.hellospring.repository.MemberRepository;
import hello.hellospring.repository.MemoryMemberRepository;
import hello.hellospring.service.MemberService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SpringConfig {

    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
}
```

- DI
    - Field Injection
        
        ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(15).png)
        
    - Setter Injection
        
        ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(16).png)
        
        ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(17).png)
        
        - setter가 public이기 때문에 앱이 조립된 후에도 누구든지 호출할 수 있음
    - Constructor Injection
        
        ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(18).png)
        
    - Constructor Injection이 권장됨
- 정형화된 controller, service, repository는 component scan 사용
정형화되지 않거나 상황에 따라 구현 클래스를 변경해야 할 경우 config 파일을 통해 spring bean으로 등록
    - e.g. MemoryMemberRepository를 다른 DB로 바꿀 때
    아래 리턴하는 객체만 바꿔주면 됨
        
        ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(19).png)
        
- `@Autowired`를 통한 DI는 spring container이 관리하는 객체(spring bean)에서만 동작함

---

> **섹션 5. 회원 관리 예제 - 웹 MVC 개발**
> 

## 회원 웹 기능 - 홈 화면 추가

- `controller/HomeController` 클래스 추가

```java
package hello.hellospring.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HomeController {

    @GetMapping("/")
    public String home() {
        return "home";
    }
}
```

- `templates/home.html` 페이지 추가

```java
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<div class="container">
 <div>
 <h1>Hello Spring</h1>
 <p>회원 기능</p>
 <p>
 <a href="/members/new">회원 가입</a>
 <a href="/members">회원 목록</a>
 </p>
 </div>
</div> <!-- /container -->
</body>
</html>
```

- `HomeController`에 의해 `"/"`으로 라우팅되기 때문에 welcome page가 작동하지 않음
(컨트롤러가 static보다 우선순위가 높음)

## 회원 웹 기능 - 등록

1. `MemberController` 클래스에 아래 매핑 추가
    
    ```java
    @GetMapping("/members/new")
    public String createForm() {
        return "members/createMemberForm";
    }
    ```
    
2. `templates/members` 디렉토리 추가 후 `/members/createMemberForm.html` 추가
    
    ```java
    <!DOCTYPE HTML>
    <html xmlns:th="http://www.thymeleaf.org">
    <body>
    <div class="container">
        <form action="/members/new" method="post">
            <div class="form-group">
                <label for="name">이름</label>
                <input type="text" id="name" name="name" placeholder="이름을
    입력하세요">
            </div>
            <button type="submit">등록</button>
        </form>
    </div> <!-- /container -->
    </body>
    </html>
    ```
    
    - `name` attribute가 서버로 넘어올 때 키로 설정됨
3. `controller`에 `MemberForm` 클래스 추가
    
    ```java
    package hello.hellospring.controller;
    
    public class MemberForm {
        private String name;
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    }
    ```
    
4. `MemberController` 클래스에 아래 `@PostMapping` 추가
    
    ```java
    @PostMapping("/members/new")
    public String create(MemberForm form) {
        Member member = new Member();
        member.setName(form.getName());
    
        memberService.join(member);
    
        return "redirect:/";
    }
    ```
    
    - get : URL을 직접 쳐서 조회할 때 사용
    - post : 데이터를 폼같은 곳에 넣어서 전달할 때 사용
    - `<input>`의 `name: 'name'`이기 때문에 `form`의 `name` 필드에 값이 들어감

## 회원 웹 기능 - 조회

- `MemberController`에 아래 매핑 추가
    
    ```java
    @GetMapping("/members")
    public String list(Model model) {
        List<Member> members = memberService.findMembers();
        model.addAttribute("members", members);
        return "members/memberList";
    }
    ```
    
- `/templates/members/memberList.html` 추가
    
    ```java
    <!DOCTYPE HTML>
    <html xmlns:th="http://www.thymeleaf.org">
    <body>
    <div class="container">
      <div>
        <table>
          <thead>
          <tr>
            <th>#</th>
            <th>이름</th>
          </tr>
          </thead>
          <tbody>
          <tr th:each="member : ${members}">
            <td th:text="${member.id}"></td>
            <td th:text="${member.name}"></td>
          </tr>
          </tbody>
        </table>
      </div>
    </div> <!-- /container -->
    </body>
    </html>
    ```
    
    - `model.members`에 회원 리스트가 들어가있음
        - `memberService.findMembers`가 `MMR.findAll` 호출
    - `member.id`, `member.name`은 getter, setter를 이용한 property 접근을 함
    - 모든 멤버 리스트에 대해 thymeleaf가 위 템플릿을 적용시켜줌
        
        ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(20).png)
        
    - 현재는 회원 정보가 메모리에 있기 때문에 서버 재시작하면 날아감

---

> **섹션 6. 스프링 DB 접근 기술**
> 
- Jdbc : 고대 기술
- spring JdbcTemplate : Jdbc의 노가다 없애줌
- JPA : DB 접근을 편리하게 바꿔줌
- spring data JPA : JPA를 편리하게 사용할 수 있도록 감싸주는 것

## H2 데이터베이스 설치

- [https://www.h2database.com/html/download-archive.html](https://www.h2database.com/html/download-archive.html)에서 1.4.200 버전 설치
    - 웹 콘솔 실행
    - JDBC URL : `jdbc:h2:~/test`인 상태에서 한 번 연결
    - `~/test.mv.db` 파일 생성된 것 확인
    - 이후에는 JDBC URL을 `jdbc:h2:tcp://localhost/~/test`로 바꾸고 연결
        - IntelliJ와 동시 접속 가능
- 아래 코드 실행해서 `MEMBER` 테이블 생성

```sql
drop table if exists member CASCADE;
create table member
(
 id bigint generated by default as identity,
 name varchar(255),
 primary key (id)
);
```

- `Long`은 `bigint`로 변환해야 함
- `id`는 `generated by default as identity` 때문에 입력이 없어도 자동으로 채워짐

- 아래 쿼리로 데이터 삽입 가능
    
    ```sql
    insert into member(name) values('spring2')
    ```
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(21).png)
    

- 루트 위치에서 `sql` 폴더 생성해서 sql 코드 관리할 수도 있음
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(22).png)
    

## 순수 Jdbc

### 환경 설정

- `build.gradle`에 jdbc, h2 관련 라이브러리 추가
    
    ```java
    implementation 'org.springframework.boot:spring-boot-starter-jdbc'
    runtimeOnly 'com.h2database:h2'
    ```
    
    - Java는 DB를 사용하려면 Jdbc 드라이버가 필요함
    - h2 : DB에 대한 client
- 스프링 부트 데이터베이스 연결 설정 추가
    
    ```java
    spring.datasource.url=jdbc:h2:tcp://localhost/~/test
    spring.datasource.driver-class-name=org.h2.Driver
    spring.datasource.username=sa
    ```
    
    - import 해줘야함(코끼리 버튼)
    - username도 뒤에 공백 없이 추가해줘야 함

### Jdbc repository 구현

- `repository/JdbcMemberRepository` 클래스 생성
    
    ```java
    package hello.hellospring.repository;
    import hello.hellospring.domain.Member;
    import org.springframework.jdbc.datasource.DataSourceUtils;
    import javax.sql.DataSource;
    import java.sql.*;
    import java.util.ArrayList;
    import java.util.List;
    import java.util.Optional;
    public class JdbcMemberRepository implements MemberRepository {
        private final DataSource dataSource;
        public JdbcMemberRepository(DataSource dataSource) {
            this.dataSource = dataSource;
        }
        @Override
        public Member save(Member member) {
            String sql = "insert into member(name) values(?)";
            Connection conn = null;
            PreparedStatement pstmt = null;
            ResultSet rs = null;
            try {
                conn = getConnection();
                pstmt = conn.prepareStatement(sql,
                        Statement.RETURN_GENERATED_KEYS);
                pstmt.setString(1, member.getName());
                pstmt.executeUpdate();
                rs = pstmt.getGeneratedKeys();
                if (rs.next()) {
                    member.setId(rs.getLong(1));
                } else {
                    throw new SQLException("id 조회 실패");
                }
                return member;
            } catch (Exception e) {
                throw new IllegalStateException(e);
            } finally {
                close(conn, pstmt, rs);
            }
        }
        @Override
        public Optional<Member> findById(Long id) {
            String sql = "select * from member where id = ?";
            Connection conn = null;
            PreparedStatement pstmt = null;
            ResultSet rs = null;
            try {
                conn = getConnection();
                pstmt = conn.prepareStatement(sql);
                pstmt.setLong(1, id);
                rs = pstmt.executeQuery();
                if(rs.next()) {
                    Member member = new Member();
                    member.setId(rs.getLong("id"));
                    member.setName(rs.getString("name"));
                    return Optional.of(member);
                } else {
                    return Optional.empty();
                }
            } catch (Exception e) {
                throw new IllegalStateException(e);
            } finally {
                close(conn, pstmt, rs);
            }
        }
        @Override
        public List<Member> findAll() {
            String sql = "select * from member";
            Connection conn = null;
            PreparedStatement pstmt = null;
            ResultSet rs = null;
            try {
                conn = getConnection();
                pstmt = conn.prepareStatement(sql);
                rs = pstmt.executeQuery();
                List<Member> members = new ArrayList<>();
                while(rs.next()) {
                    Member member = new Member();
                    member.setId(rs.getLong("id"));
                    member.setName(rs.getString("name"));
                    members.add(member);
                }
                return members;
            } catch (Exception e) {
                throw new IllegalStateException(e);
            } finally {
                close(conn, pstmt, rs);
            }
        }
        @Override
        public Optional<Member> findByName(String name) {
            String sql = "select * from member where name = ?";
            Connection conn = null;
            PreparedStatement pstmt = null;
            ResultSet rs = null;
            try {
                conn = getConnection();
                pstmt = conn.prepareStatement(sql);
                pstmt.setString(1, name);
                rs = pstmt.executeQuery();
                if(rs.next()) {
                    Member member = new Member();
                    member.setId(rs.getLong("id"));
                    member.setName(rs.getString("name"));
                    return Optional.of(member);
                }
                return Optional.empty();
            } catch (Exception e) {
                throw new IllegalStateException(e);
            } finally {
                close(conn, pstmt, rs);
            }
        }
        private Connection getConnection() {
            return DataSourceUtils.getConnection(dataSource);
        }
        private void close(Connection conn, PreparedStatement pstmt, ResultSet rs)
        {
            try {
                if (rs != null) {
                    rs.close();
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
            try {
                if (pstmt != null) {
                    pstmt.close();
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
            try {
                if (conn != null) {
                    close(conn);
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        private void close(Connection conn) throws SQLException {
            DataSourceUtils.releaseConnection(conn, dataSource);
        }
    }
    ```
    
    - DB 관련 리소스들은 외부 네트워크와의 연결이기 때문에 쓴 직후 release해야 함
- `hello.hellospring/SpringConfig.java` 에서 `memberRepository()`의 리턴을 `JdbcMemberRepository()`로 수정
    
    ```java
    @Configuration
    public class SpringConfig {
    
        private DataSource dataSource;
    
        @Autowired
        public SpringConfig(DataSource dataSource) {
            this.dataSource = dataSource;
        }
    
        @Bean
        public MemberService memberService() {
            return new MemberService(memberRepository());
        }
    
        @Bean
        public MemberRepository memberRepository() {
            // return new MemoryMemberRepository();
            return new JdbcMemberRepository(dataSource);
        }
    }
    ```
    
    - `dataSource`는 `@Autowired`로 가져와야 함
    - `DataSource` : DB connection을 획득할 때 사용하는 객체
        - spring boot는 DB connection을 바탕으로 `DataSource`를 생성하고 spring bean으로 만듦
        → DI 받을 수 있음
- 애플리케이션을 assembly하는 부분만 바꿔서 편리하게 member repository 교체 가능
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(23).png)
    
    - 객체지향이 아닐 경우 MMR을 사용하던 MemberService의 코드를 모두 고쳐야 했음
    - spring의 DI를 사용하면 기존 코드를 손대지 않고 설정(assembly하는 부분 수정)만으로 구현 클래스를 변경할 수 있음
    - 데이터를 DB에 저장하기 때문에 서버를 재시작해도 데이터가 보존됨
- OCP(Open-Closed Principle) : 확장에는 열려있고, 수정, 변경에는 닫혀있음

## 스프링 통합 테스트

- `test/java/hello.hellospring/service/MemberServiceIntegrationTest.java` 생성

```java
package hello.hellospring.service;

import hello.hellospring.domain.Member;
import hello.hellospring.repository.MemberRepository;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.api.Assertions.assertThrows;

@SpringBootTest
//@Transactional
class MemberServiceIntegrationTest {

    @Autowired
    MemberService memberService;
    @Autowired
    MemberRepository memberRepository;

    @Test
    void join() {
        // given
        Member member = new Member();
        member.setName("spring");

        // when
        Long savedId = memberService.join(member);

        // then
        Member findMember = memberService.findOne(savedId).get();
        assertThat(member.getName()).isEqualTo(findMember.getName());
    }

    @Test
    public void dupMemEXC() {
        // given
        Member member1 = new Member();
        member1.setName("spring");
        Member member2 = new Member();
        member2.setName("spring");

        // when
        memberService.join(member1);
        IllegalStateException e = assertThrows(IllegalStateException.class, () -> memberService.join(member2));
        assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");

        // then
    }
    @Test
    void findMembers() {
    }

    @Test
    void findOne() {
    }
}
```

- DB는 한 번 지워줘야 함(실제로는 테스트용 DB를 따로 개설해서 사용)
- `join` 테스트를 반복할 수 있도록 만들기 위해 `@Transactional` 사용
    - 테스트가 끝나면 DB를 롤백해줌
    - DB는 보통 insert query가 실행된 다음 커밋해야 반영됨
- `@SpringBootTest` : spring container와 테스트를 함께 실행함

## 스프링 JdbcTemplate

- build.gradle의 설정은 순수 Jdbc와 동일함
- JdbcTemplate, MyBatis와 같은 라이브러리는 JDBC API에서 반복 코드를 제거해줌
    - SQL은 직접 작성해야 함
- `main/java/hello.hellospring/repository/JdbcTemplateMemberRepository.java` 생성

```java
package hello.hellospring.repository;

import hello.hellospring.domain.Member;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;
import org.springframework.jdbc.core.simple.SimpleJdbcInsert;

import javax.sql.DataSource;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Optional;
public class JdbcTemplateMemberRepository implements MemberRepository {
    private final JdbcTemplate jdbcTemplate;

    public JdbcTemplateMemberRepository(DataSource dataSource) {
        jdbcTemplate = new JdbcTemplate(dataSource);
    }

    @Override
    public Member save(Member member) {
        SimpleJdbcInsert jdbcInsert = new SimpleJdbcInsert(jdbcTemplate);
        jdbcInsert.withTableName("member").usingGeneratedKeyColumns("id");
        Map<String, Object> parameters = new HashMap<>();
        parameters.put("name", member.getName());
        Number key = jdbcInsert.executeAndReturnKey(new
                MapSqlParameterSource(parameters));
        member.setId(key.longValue());
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) {
        List<Member> result = jdbcTemplate.query("select * from member where id = ?", memberRowMapper(), id);
        return result.stream().findAny();
    }

    @Override
    public List<Member> findAll() {
        return jdbcTemplate.query("select * from member", memberRowMapper());
    }

    @Override
    public Optional<Member> findByName(String name) {
        List<Member> result = jdbcTemplate.query("select * from member where name = ?", memberRowMapper(), name);
        return result.stream().findAny();
    }

    private RowMapper<Member> memberRowMapper() {
        return (rs, rowNum) -> {
            Member member = new Member();
            member.setId(rs.getLong("id"));
            member.setName(rs.getString("name"));
            return member;
        };
    }
}
```

- 생성자가 하나일 경우 `@Autowired` 생략 가능
- `SpringConfig.java`의 `memberRepository()`에서 리턴 문장을 아래로 변경

```java
return new JdbcTemplateMemberRepository(dataSource);
```

- 테스트 실행해서 검증

## JPA

- JPA : 기존의 반복코드, 기본적인 SQL을 JPA가 처리해줌
    - SQL과 데이터 중심의 설계에서 객체 중심의 설계로 패러다임 전환 가능
- `build.gradle` 파일에 JPA 관련 라이브러리 추가

```java
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
```

- `resources/application.properties` 파일에 JPA 설정 추가

```java
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=none
```

- `~.ddl-auto` : 자동으로 테이블 생성해줌

### JPA entity 매핑

- `domain/Member.java` 아래와 같이 수정

```java
...
@Entity
public class Member{
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
...
```

- `repository/JpaMemberRepository.java` 생성

```java
package hello.hellospring.repository;

import hello.hellospring.domain.Member;

import javax.persistence.EntityManager;
import java.util.List;
import java.util.Optional;

public class JpaMemberRepository  implements MemberRepository{

    private final EntityManager em;

    public JpaMemberRepository(EntityManager em) {
        this.em = em;
    }

    @Override
    public Member save(Member member) {
        em.persist(member);
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) {
        Member member = em.find(Member.class, id);
        return Optional.ofNullable(member);
    }

    @Override
    public Optional<Member> findByName(String name) {
        List<Member> result = em.createQuery("select m from Member m where m.name = :name", Member.class)
                .setParameter("name", name)
                .getResultList();
        
        return result.stream().findAny();
    }

    @Override
    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }
}
```

- `persist` : save하는 함수
- JPA를 통한 데이터 변경은 항상 `@Transactional` 안에서 실행되어야 함
    - 서비스 계층에 트랜잭션 추가(`service/MemberService.java`)
    
    ```java
    import org.springframework.transaction.annotation.Transactional;
    ...
    @Transactional
    public class MemberService {
    ...
    ```
    
- `SpringConfig.java` 수정

```java
...
@Configuration
public class SpringConfig {

    private final DataSource dataSource;
    private EntityManager em;

    @Autowired
    public SpringConfig(DataSource dataSource, EntityManager em) {
        this.dataSource = dataSource;
        this.em = em;
    }

    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        // return new MemoryMemberRepository();
        // return new JdbcMemberRepository(dataSource);
        // return new JdbcTemplateMemberRepository(dataSource);
        return new JpaMemberRepository(em);
    }
}
```

## 스프링 데이터 JPA

- 스프링 데이터 JPA가 interface를 보고 구현체를 만들어서 등록해줌
- `repository/SpringDataJpaMemberRepository.java` 생성

```java
...
public interface SpringDataJpaMemberRepository extends JpaRepository<Member, Long>, MemberRepository {
    Optional<Member> findByName(String name);
}
```

- `SpringConfig.java` 변경

```java
@Configuration
public class SpringConfig {

    private final MemberRepository memberRepository;

    public SpringConfig(MemberRepository memberRepository){
        this.memberRepository=memberRepository;
    }

    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository);
    }
}
```

- 스프링 데이터 JPA가 `SpringDataJpaMemberRepository`를 스프링 빈으로 자동 등록함

### 스프링 데이터 JPA의 제공 클래스

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(24).png)

- 제공 기능
    - interface를 통한 기본적인 CRUD
    - `findByName()`, `findByEmail()`과 같은 메서드(이름을 같게 적어놓으면 자동으로 사용됨)
    - 페이징 기능
- 기본적으로 JPA, 스프링 데이터 JPA를 사용하고, 복잡한 동적 쿼리는 Querydsl 라이브러리 사용
    - 커버가 안될 경우 Native 쿼리 또는 JdbcTemplate 사용

---

> **섹션 7. AOP**
> 

## AOP가 필요한 상황

- 모든 메소드의 호출 시간을 측정하고 싶을 때

```java
public Long join(Member member) {
    long start = System.currentTimeMillis();

    try{
        validateDuplicateMember(member);
        memberRepository.save(member);
        return member.getId();
    } finally {
        long finish = System.currentTimeMillis();
        long timeMs = finish - start;
        System.out.println("join = " + timeMs + "ms");
    }
}
public List<Member> findMembers() {
    long start = System.currentTimeMillis();
    try {
        return memberRepository.findAll();
    } finally {
        long finish = System.currentTimeMillis();
        long timeMs = finish - start;
        System.out.println("findMembers " + timeMs + "ms");
    }
}
```

- `try...finally`로 일일히 감싸고, `start`, `finish`, `timeMs`로 시간 계산

### 문제

- 공통 관심 사항(cross-cutting concern) : 앱 전반에 걸쳐 필요한 기능(transaction, auth 등)
    - 시간 측정 기능 : 공통 관심 사항
- 핵심 관심 사항(core concern) : 핵심 비즈니스 로직
- 시간 측정 로직과 비즈니스 로직이 섞여서 유지보수가 어려움
- 시간 측정 로직을 별도의 공통 로직으로 만들기가 매우 어렵고, 수정해야 할 경우 모두 수동으로 변경해야 함

## AOP 적용

- AOP(Aspect Oriented Programming) : cross-cutting concern, core concern을 분리함

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(25).png)

- 시간 측정 로직을 한 군데에 모아놓고, 적용할 서비스를 지정하면 적용됨

`hello.hellospring.app` 패키지 추가 후 `TimeTraceApp` 클래스 추가

```java
@Aspect
@Component
public class TimeTraceAop {
    @Around("execution(* hello.hellospring..*(..))")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        System.out.println("START: " + joinPoint.toString());
        try {
            return joinPoint.proceed();
        } finally {
            long finish = System.currentTimeMillis();
            long timeMs = finish - start;
            System.out.println("END: " + joinPoint.toString() + " " + timeMs + "ms");
        }
    }
}
```

- SpringConfig에 Bean으로 등록하거나 `@Component` 어노테이션 붙이면 springbean에 등록됨
- `MemberService`에 추가했던 시간 측정 로직은 다시 지우면 됨

### 해결

- 핵심 관심 사항과 공통 관심 사항을 분리함
- 시간 측정 로직을 별도의 공통 로직으로 만듦
    - 핵심 관심 사항을 깔끔하게 유지 가능
    - 유지보수가 쉬움(수정, 적용 대상 선택 등)
        - `"execution(* hello.hellospring.service..*(..))"`과 같이 원하는 범위 설정 가능  
        (패키지 단위로 많이 함)

### 스프링의 AOP 동작 방식 설명

- AOP 적용 전 의존관계
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(26).png)
    
- AOP 적용 후 의존관계
    
    ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(27).png)
    
    - AOP가 적용되어 있다면 프록시(가짜) memberService를 만듦
    - spring container에서는 가짜 spring bean이 실행된 다음 `joinPoint.proceed()`메소드를 통해 실제 spring bean이 호출됨
        - `memberService.getClass()`로 확인 가능
- AOP 적용 결과
    - 적용 전
        
        ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(28).png)
        
    - 적용 후
        
        ![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap1%20(29).png)