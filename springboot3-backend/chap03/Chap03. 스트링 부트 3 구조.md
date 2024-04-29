# Chap03. 스프링 부트 3 구조

<aside>
💡 각각의 계층은 자신의 책임에 맞는 역할을 수행, 필요에 따라 다른 계층과 소통함<br>
  &emsp;&nbsp; 계층은 개념의 영역, 컨트롤러/서비스/레포지토리는 실제 구현의 영역

</aside>

## 프레젠테이션 계층

- 컨트롤러
- HTTP 요청을 받고 요청을 비즈니스 계층에 전달
- 여러 개의 컨트롤러 존재 가능

## 비즈니스 계층

- 서비스
- 모든 비즈니스 로직 처리
⇒ 비즈니스는 서비스를 만들기 위한 로직
- 주문 서비스
    - 주문 개수, 가격 등 데이터 처리 로직
    - 예외 처리
    - 주문을 받거나 취소하는 프로세스 로직

## 퍼시스턴스 계층

- 레포지토리
- 모든 데이터베이스 관련 로직 처리
- 데이터베이스 계층과 상호작용하기 위한 객체 사용(DAO)

## 디렉터리 구성

### main

- 실제 코드 작성 공간
- 프로젝트에 필요한 소스 코드나 리소스 파일로 구성
- main 구성
    - java
    - resources
        - templates ⇒ HTML과 같은 뷰 파일 디렉터리
        - static ⇒ JS, CSS, 이미지같은 정적 파일 디렉터리
        - application.yml ⇒ 스프링 부트 설정 파일
        (스프링 부트 서버가 시작되면 자동 로딩, 데이터베이스 설정 정보, 로깅 설정 정보 포함)

### test

- 프로젝트 소스 코드를 테스트할 목적의 코드나 리소스 파일로 구성

### build.gradle

- 빌드를 설정하는 파일
- 의존성, 플러그인 설정같이 빌드에 필요한 설정들로 구성
- JPA인 스프링 데이터 JPA, 로컬 환경과 테스트 환경에서 사용하는 인메모리 데이터베이스 H2, 반복 메서드 작성 작업 줄여주는 롬복 라이브러리 설정

### settings.gradle

- 빌드할 프로젝트의 정보를 설정

## 프레젠테이션, 서비스, 퍼시스턴스 계층 만들기

<aside>
💡 HTTP 요청 → TestController(프레젠테이션) → TestService(비즈니스) → MemberRespository(퍼시스턴스) → 데이터베이스
</aside>

<br><br>

```java
/* TestController.java */

@RestController
public class TestController {
    @Autowired //테스트 서비스 빈 주입
    TestService testService;

    @GetMapping("/test")
    public List<Member> getAllMembers() {
        List<Member> members = testService.getAllMembers();
        return members;
    }
}
```

```java
/* TestService.java */

@Service
public class TestService {
    @Autowired
    MemberRepository memberRepository; //1. 빈 주입

    public List<Member> getAllMembers() {
        return memberRepository.findAll(); //2. 멤버 목록 얻기
    }
}
```
- MemberRepository라는 빈 주입
- findAll() 메서드 호출 ⇒ 멤버 테이블에 저장된 데이터 목록 가져오기

```java
/* Member.java */

@NoArgsConstructor(access= AccessLevel.PROTECTED)
@AllArgsConstructor
@Getter
@Entity
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id", updatable = false)
    private Long id; // DB 테이블의 'id' 컬럼과 매칭
    
    @Column(name = "name", nullable = false)
    private String name; // DB 테이블의 'name' 컬럼과 매칭
}
```

- DB에 접근할 때 사용할 Member DAO 객체
- member라는 이름의 테이블에 접근하는 데 사용할 객체

```java
/* MemberRepository.java */

@Repository
public interface MemberRepository  extends JpaRepository<Member,Long> {
}
```
- DB에서 데이터를 가져오는 퍼시스턴트 계층 역할 인터페이스
- member라는 이름의 테이블에 접근해 Member 클래스에 매핑하는 구현체

## 요청-응답 과정

<aside>
💡 포스트맨(/test GET) → 톰캣 → 스프링 컨테이너(디스패처 서블릿 → /test GET 요청과 getAllMembers() 메서드 매치 → 뷰 리졸버) → 톰캣 → 포스트맨

</aside>

- 디스패처 서블릿은 URL을 분석하고 요청을 처리할 수 있는 컨트롤러 찾음
    - TestController가 /test라는 패스에 대한 GET 요청을 처리할 수 있는 getAllMembers() 메서드를 가지고 있음
    - 디스패처 서블릿은 TestController에게 /test GET 요청을 전달
- getAllMembers() 메서드는 비즈니스 계층과 퍼시스턴스 계층을 통해 필요한 데이터를 가져옴
- 뷰 리졸버는 템플릿 엔진을 사용해 HTML 문서를 만들거나 JSON, XML 등 데이터 생성
