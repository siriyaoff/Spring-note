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