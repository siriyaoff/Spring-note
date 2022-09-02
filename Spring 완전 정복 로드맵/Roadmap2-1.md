# 섹션 1. 객체 지향 설계와 스프링

## 이야기 - 자바 진영의 추운 겨울과 스프링의 탄생

- EJB → Hibernate → JPA
- JPA : 표준 인터페이스
    - Hibernate, EclipseLink 등의 구현체들이 존재함
- 스프링 이름은 J2EE(EJB)라는 겨울을 넘은 새로운 봄이란 뜻이었음
- 로드 존슨이 쓴 책의 예제 코드가 스프링의 시작이었음
- 스프링의 초기 설정을 갖춘게 스프링 부트임
- Reactive programming : async non-blocking하게 개발할 수 있게 만들어줌

## 스프링이란?

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-1%20(1).png)

- 스프링 데이터 : DB의 CRUD 보조
- 스프링 세션 : 세션 기능 보조
- 스프링 Rest Docs : 문서화 보조
- 스프링 배치 : 배치 처리에 특화됨

### 스프링 프레임워크

- 핵심 기술 : 스프링 DI container, AOP, event, …
    - 이번 강의에서는 핵심 기술에 초점을 맞춤
- 웹 기술 : Spring MVC, Spring WebFlux
- 데이터 접근 기술 : 트랜잭션, JDBC, ORM 지원, XML 지원
- 기술 통합 : Cache, Email, 원격접근, 스케줄링
- 테스트, 언어(코틀린, 그루비)

### 스프링 부트

- 스프링을 편리하게 사용할 수 있도록 지원
- 단독 실행이 가능한 Spring app을 쉽게 생성
    - Tomcat과 같은 웹 서버를 내장함  
    → 별도의 웹 서버 필요 없음
- 빌드 구성을 위한 starter 종속성 제공
- 써드파티 라이브러리 자동 구성
- 메트릭, 상태 확인, 외부 구성 등의 production 준비 기능 제공
- 관례에 의한 간결한 설정이 되어있음

> 스프링이라는 단어는 문맥에 따라 다르게 사용됨  
스프링 DI 컨테이너, 스프링 프레임워크, 스프링 생태계 등등
> 

### 스프링의 핵심 개념

- Java 기반의 프레임워크
- 객체 지향 언어가 가진 특징을 살려냄
→ 좋은 객체 지향 app 개발을 도와줌

## 좋은 객체 지향 프로그래밍이란?

- 객체 지향의 특징 : 추상화, 캡슐화, 상속, 다형성
- 객체 지향 프로그래밍 : 프로그램을 객체들의 모임으로 파악하고자 하는 것
    - 각각의 객체는 협력할 수 있음
    - 프로그램을 유연하고 변경이 용이하게 만들기 때문에 대규모 개발에 많이 사용됨
        - Component를 쉽고 유연하게 변경하면서 개발 가능함 == 다형성

### 다형성

- 실세계 비유 : 운전자 - 자동차, 정렬 알고리즘 등 역할만 잘 수행한다면 바꿔도 문제가 없음
- 역할과 구현을 분리
    - client는 대상의 역할(interface)만 알면 됨  
    내부 구조를 모르거나 변경되어도 상관없음
- 객체를 설계할 때 interface와 implementation을 명확히 분리
    - interface를 먼저 부여하고 구현 객체 만들기
    - 일반 상속 관계로도 다형성 형성이 가능은 하지만, 다중 상속이 안되기 때문에 interface를 사용하는게 나음

### 자바의 다형성

- 자식 객체는 메소드까지 상속받지만, 새로 구현해서 사용할 수 있음

![Untitled](https://github.com/siriyaoff/Spring-note/blob/main/Spring%20%EC%99%84%EC%A0%84%20%EC%A0%95%EB%B3%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5/images/Roadmap2-1%20(2).png)

```java
public class MemberService {
	// private MemberRepository memberRepository = new MemoryMemberRepository();
	private MemberRepository memberRepository = new MemoryMemberRepository();
}
```

- 부모 객체에 자식 인스턴스를 넣을 수 있음
    - 오버라이드된 자식 메소드가 실행됨
- 객체 간의 협력 관계를 통해 다형성을 쉽게 이해할 수 있음
    - interface를 구현한 객체 instance를 실행 시점에 유연하게 변경 가능
    - 클라이언트 - 서버 관계 : client를 변경하지 않고 server의 구현 기능을 유연하게 변경할 수 있음

### 다형성의 한계

- interface 자체가 변하면 양쪽 모두에 큰 변경이 발생함
    - interface를 안정적으로 설계하는 것이 중요

### 스프링과 객체 지향

- IoC, DI 등은 다형성을 활용해서 역할과 구현을 편리하게 다룰 수 있도록 지원함

## 좋은 객체 지향 설계의 5가지 원칙(SOLID)

- SOLID : 클린 코드의 저자 로버트 마틴이 정리함
- SRP(Single Responsibility Principle)
- OCP(Open/Closed Principle)
- LSP(Liskov Substitution Principle)
- ISP(Interface Segregation Principle)
- DIP(Dependency Inversion Principle)

### Single Responsibility Principle

- 한 클래스는 하나의 책임만 가져야 함
- 변경이 있을 때 파급 효과가 적으면 SRP를 잘 따른 것  
e.g. UI 변경, 객체의 생성과 사용을 분리

### Open/Closed Principle

- 확장에는 열려있으나, 변경에는 닫혀있어야 함
- 다형성을 사용해야 함
- interface를 구현한 새로운 클래스를 만들어서 새로운 기능을 구현  
e.g. `JdbcMemberRepository` 객체
- 문제점
    - 위의 `MemberService`의 경우 repository를 클라이언트가 직접 선택하는 꼴임(주석처리하고 지정해줘야 함)
    - 즉 구현 객체를 변경하려면 client의 코드를 변경해야 함  
    → 다형성을 사용했지만 OCP를 지킬 수 없음
    - client의 코드를 변경할 필요가 없도록 Spring container가 DI, IoC와 같은 기능을 통해서 별도의 설정자를 제공함

### Liskov Substitution Principle

- 객체는 프로그램의 정확성을 깨뜨리지 않으면서 하위 타입의 instance로 바꿀 수 있어야 함
    - 다형성에서 하위 클래스는 interface 규약을 다 지켜야 함
- interface를 구현한 구현체를 믿고 사용하기 위해서 필요

### Interface Segregation Principle

- 특정 client를 위한 interface 여러 개가 범용 interface 하나보나 나음
    - 자동차 interface : 운전, 정비 interface로 분리
    - 사용자 client : 운전자, 정비사 client로 분리
- 대체 가능성이 높아짐

### Dependency Inversion Principle

- 프로그래머는 추상화에 의존해야 함
- 구현체에 의존하게 되면 변경이 복잡해짐
- OCP에서 설명한 `MemberService`는 interface와 구현체에 동시에 의존함  
`MemberRepository m = new MemoryMemberRepository();`
    - DIP 위반

### 정리

- 객체 지향의 핵심은 다형성이지만, 다형성 만으로는 OCP, DIP를 지킬 수 없음

## 객체 지향 설계와 스프링

- 스프링은 DI container을 통해 다형성, OCP, DIP 지원
    - DI(Dependency Injection) : 의존관계, 의존성 주입
    - DI container를 통해 DI 수행
- client 코드의 변경 없이 기능 확장 가능
- 사실 OCP, DIP 원칙을 지키며 개발하려고 하면 결국 DI container를 만들게 됨 → 스프링이 만들어짐

### 실무 고민

- 이상적으로는 모든 설계에 interface를 부여하는게 좋음
    - 추상화라는 비용이 발생함
        - 사용할 객체가 런타임에 결정되기 때문에 구현 클래스를 보기 위해서 디버깅할 때 시간 소모가 커짐
- 기능을 확장할 가능성이 없다면 구현체를 직접 사용하고, 꼭 필요할 때 리팩터링을 통해 interface를 도입하는 방법도 좋음