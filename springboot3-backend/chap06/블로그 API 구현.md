# Chap06. 블로그 API 구현

## API

- 네트워크의 API는 프로그램 간에 상호작용하기 위한 매개체

<aside>
💡 클라이언트(요청) → API(요청) → 서버(응답) → API(응답) → 클라이언트

</aside>

- 클라이언트(요청)
    - 구글 메인 화면 보여줘
- API(요청)
    - 클라이언트에게 받은 요청을 서버에게 줌
- 서버(응답)
    - API가 준 요청을 처리해 결과물을 만들어서 API에게 다시 전달
- API(응답)
    - 최종 결과물을 브라우저에 보내 화면을 볼 수 있게 됨

## REST API

- 웹의 장점을 최대한 살린 API
- Representational State Transfer
⇒ 자원을 이름으로 구분해 자원의 상태를 주고 받는 API
- URL의 설계 방식

### REST API의 특징

- 서버/클라이언트 구조
- 무상태
- 캐시 처리 가능
- 계층화
- 인터페이스 일관성

### 장점

- URL만 보고 무슨 행동을 하는 API인지 명확하게 알 수 있음
- 상태가 없어 클라이언트와 서버의 역할이 명확하게 분리됨
- HTTP 표준을 사용하는 모든 플랫폼에서 사용할 수 있음
- 주소와 메서드만 보고 요청의 내용을 파악할 수 있음

### 단점

- GET, POST와 같은 방식의 개수에 제한이 있음
- 설계에 공식적 표준 규약이 없음

### 사용 방법

- URL에는 동사를 쓰지 말고, 자원을 표시해야 함
    - 개발자가 URL을 설계할 때 get을 사용할 수도 있고, show를 사용할 수도 있음
    ⇒ 이 과정에서 헷갈릴 수 있기 때문에 동사 쓰지 않음
- 동사는 HTTP 메서드로 사용
    - HTTP 메서드는 서버에 요청 하는 방법을 나눈 것
    - POST(생성), GET(조회), PUT(수정), DELETE(삭제)가 있음
    ⇒ CRUD라고 함

## 엔티티 구성하기

| 컬럼명 | 자료형 | null 허용 | 키 | 설명 |
| --- | --- | --- | --- | --- |
| id | BIGINT | N | 기본키 | 일련번호, 기본키 |
| title | VARCHAR(255) | N |  | 게시물의 제목 |
| content | VARCHAR(255) | N |  | 내용 |

```java
/* Article.java */

@Entity //엔티티로 지정
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Article {
    @Id //id 필드를 기본키로 지정
    @GeneratedValue(strategy = GenerationType.IDENTITY) //기본키를 자동으로 1씩 증가
    @Column(name = "id", updatable = false)
    private Long id;

    @Column(name = "title", nullable = false) //'title'이라는 not null 컬럼과 매핑
    private String title;
    @Column(name = "content", nullable = false)
    private String content;

    @Builder //빌더 패턴으로 객체 생성
    public Article(String title, String content) {
        this.title = title;
        this.content = content;
    }
}
```

- @Builder: 롬복에서 지원하는 애너테이션
- 빌더 패턴: 개발자들이 많이 사용하는 디자인 패턴
- 빌더 패턴 방식으로 객체를 생성할 수 있음
- 빌더 패턴은 어느 필드에 어떤 값이 들어가는지 명시적으로 파악할 수 있음
    - 빌더 패턴을 사용하지 않았을 때
        
        `new Article(”abc”, “def”);`
        
        - 어느 필드에 어떤 값이 들어가는지 잘 모름
    - 빌더 패턴을 사용했을 때
        
        `Article.builder()
        .title(”abc”)
        .content(”def”)
        .build();`
        
        - 가독성이 높음
- @Getter, @NoArgsConstructor 애너테이션을 사용하여 별도의 get() 메서드, 생성자를 구현하지 않아도됨

```java
/* BlogRepository.java */

public interface BlogRepository extends JpaRepository<Article, Long> {

}
```

## 블로그 글 생성 API 구현

- DTO는 계층끼리 데이터를 교환하기 위해 사용하는 객체
- DTO는 단순하게 데이터를 옮기기 위해 사용하는 전달자 역할
    
    ⇒ 별도의 비즈니스 로직을 포함하지 않음
    

```java
/* AddArticleRequest */

@NoArgsConstructor // 기본 생성자 추가
@AllArgsConstructor // 모든 필드 값을 파라미터로 받는 생성자 추가
@Getter
public class AddArticleRequest {
    private String title;
    private String content;

    public Article toEntity() { // 생성자를 사용해 객체 생성
        return Article.builder()
                .title(title)
                .content(content)
                .build();
    }
}
```

- toEntity()는 빌더 패턴을 사용해 DTO를 엔티티로 만들어주는 메서드

```java
/* BlogService */

@RequiredArgsConstructor //final이 붙거나 @NotNull이 붙은 필드의 생성자 추가
@Service //빈으로 등록
public class BlogService {
    private final BlogRepository blogRepository;

		// 블로그 글 추가 메서드
    public Article save(AddArticleRequest request){
        return blogRepository.save(request.toEntity());
    }
}
```

- @RequiredArgConstructor는 빈을 생성자로 생성하는 롬복에서 지원하는 애너테이션
⇒ final 키워드나 @NotNull이 붙은 필드로 생성자를 만들어줌
- @Service 애너테이션은 해당 클래스를 빈으로 서블릿 컨테이너에 등록해줌
- save() 메서드는 JpaRepository에서 지원하는 저장 메서드 save()로 AddArticleRequest 클래스에 저장된 값들을 article 데이터베이스에 저장

```java
/* BlogApiController */

@RequiredArgsConstructor
@RestController //HTTP Response Body에 객체 데이터를 JSON 형식으로 반환하는 컨트롤러
public class BlogApiController {

    private final BlogService blogService;
		
		// HTTP 메서드가 POST일 때 전달받은 URL과 동일하면 메서드로 매핑
    @PostMapping("/api/articles")
    // @RequestBody로 요펑 본문 값 매핑
    public ResponseEntity<Article> addArticle(@RequestBody AddArticleRequest request) {
        Article savedArticle = blogService.save(request);
        // 요청한 자원이 성공적으로 생성되었으며 저장된 블로그 글 정보를 응답 객체에 담아 전송
        return ResponseEntity.status(HttpStatus.CREATED)
                .body(savedArticle);
    }
}
```

- @RestController 애너테이션을 클래스에 붙이면 HTTP 응답으로 객체 데이터를 JSON 형식으로 반환
- @PostMapping() 애너테이션은 HTTP 메서드가 POST일 때 요청받은 URL과 동일한 메서드와 매핑
⇒ /api/articles는 addArticle()메서드와 매핑
- @RequestBody 애너테이션은 HTTP를 요청할 때 응답에 해당하는 값을 @RequestBody 애네터이션이 붙은 대상 객체인 AddArticleRequest에 매핑
- ResponseEntity.status().body()는 응답 코드로 201, 즉, created를 응답하고 테이블에 저장된 객체를 반환
    - 200: 요청이 성공적으로 수행
    - 201: 요청이 성공적으로 수행되었고, 새로운 리소스가 생성
    - 400: 요청 값이 잘못되어 요청에 실패
    - 403: 권한이 없어 요청에 실패
    - 404: 요청 값으로 찾은 리소스가 없어 요청에 실패
    - 500: 서버 상에 문제가 있어 요청에 실패

### 생성 API 실행 테스트

```java
/* BlogApiControllerTest.java */

@SpringBootTest // 테스트용 애플리케이션 컨텍스트
@AutoConfigureMockMvc // MockMvc 생성 및 자동 구성
class BlogApiControllerTest {

    @Autowired
    protected MockMvc mockMvc;

    @Autowired
    protected ObjectMapper objectMapper; // 직렬화, 역직렬화를 위한 클래스

    @Autowired
    private WebApplicationContext context;

    @Autowired
    BlogRepository blogRepository;

    @BeforeEach // 테스트 실행 전 실행하는 메서드
    public void mockMvcSetUp() {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(context)
                .build();
        blogRepository.deleteAll();
    }
    
    @DisplayName("addArticle: 블로그 글 추가에 성공한다.")
    @Test
    public void addArticle() throws Exception {
        //given
        final String url = "/api/articles";
        final String title = "title";
        final String content = "content";
        final AddArticleRequest userRequest = new AddArticleRequest(title, content);

        //객체 JSON으로 직렬화
        final String requestBody = objectMapper.writeValueAsString(userRequest);

        //when
        //설정한 내용을 바탕으로 요청 전송
        ResultActions result = mockMvc.perform(post(url)
                .contentType(MediaType.APPLICATION_JSON_VALUE)
                .content(requestBody));

        //then
        result.andExpect(status().isCreated());

        List<Article> articles = blogRepository.findAll();

        assertThat(articles.size()).isEqualTo(1); // 크기가 1인지 검증
        assertThat(articles.get(0).getTitle()).isEqualTo(title);
        assertThat(articles.get(0).getContent()).isEqualTo(content);
    }
}
```

- ObjectMapper클래스: 자바 객체를 JSON 데이터로 변환하는 직렬화 또는 반대로 JSON을 데이터로 변환하는 역직렬화할 때 사용
- writeValueAsString() 메서드를 사용해서 객체를 JSON으로 직렬화해줌
- MockMvc를 사용해서 HTTP 메서드, URL, 요청 본문, 요청 타입 등을 설정한 뒤 설정한 내용을 바탕으로 요청함
- contentType() 메서드는 요청을 보낼 때 JSON, XML 등 다양한 타입 중 하나를 골라 요청 보냄
- assertThat() 메서드로는 블로그 글의 개수가 1인지 확인

## 블로그 글 조회 API 구현

### 글 목록 조회 API

```java
/* BlogService.java */

@RequiredArgsConstructor
@Service
public class BlogService {
    
    private final BlogRepository blogRepository;

		... 생략 ...

		public List<Article> findAll() {
				return blogRepository.findAll();
		}
}
```

- JPA 지원 메서드인 findAll()을 호출해 article 테이블에 저장되어 있는 모든 데이터를 조회

```java
/* ArticleResponse.java */

@Getter
public class ArticleResponse {

    private final String title;
    private final String content;

    public ArticleResponse(Article article){
        this.title = article.getTitle();
        this.content = article.getContent();
    }
}
```

- 응답을 위한 DTO
- 글은 제목과 내용으로 구성되어 있으므로 해당 필드를 가지는 클래스 구현
- 엔티티를 인수로 받는 생성자를 추가

```java
/* BlogApiController.java */

@RequiredArgsConstructor
@RestController // HTTP Response Body에 객체 데이터를 JSON 형식으로 반환하는 컨트롤러
public class BlogApiController {

    private final BlogService blogService;
    
    ... 생략 ...
		
		@GetMapping("/api/articles")
    public ResponseEntity<List<ArticleResponse>> findAllArticles() {
        List<ArticleResponse> articles = blogService.findAll()
                .stream()
                .map(ArticleResponse::new)
                .toList();
        return ResponseEntity.ok()
                .body(articles);
    }
}
```

- /api/articles GET 요청이 오면 글 목록을 조회할 findAllArticles() 메서드
⇒ 전체 글 목록을 조회하고 응답하는 역할
- 요청이 오면 글 전체를 조회하는 findAll() 메서드를 호출한 다음 응답용 객체인 ArticleResponse로 파싱해 body에 담아 클라이언트에 전송

```java
/* BlogApiControllerTest.java */

@SpringBootTest
@AutoConfigureMockMvc
class BlogApiControllerTest {

		... 생략 ...

		@DisplayName("findAllArticles: 블로그 글 목록 조회에 성공한다.")
    @Test
    public void findAllArticles() throws Exception {
        //given
        final String url = "/api/articles";
        final String title = "title";
        final String content = "content";

        blogRepository.save(Article.builder()
                .title(title)
                .content(content)
                .build());

        //when
        final ResultActions resultActions = mockMvc.perform(get(url)
                .accept(MediaType.APPLICATION_JSON_VALUE));

        //then
        resultActions
                .andExpect(status().isOk())
                .andExpect(jsonPath("$[0].content").value(content))
                .andExpect(jsonPath("$[0].title").value(title));
    }

```

### 글 상세조회 API

```java
/* BlogService.java */

@RequiredArgsConstructor
@Service
public class BlogService {

		... 생략 ...
		
		public Article findById(long id) {
        return blogRepository.findById(id)
                .orElseThrow(() -> new IllegalArgumentException("not found: " + id));
    }
}
```

- 글의 ID를 이용해 글을 조회
- findById() 메서드는 JPA에서 제공하는 메서드를 사용해 ID를 받아 엔티티 조회
- 조회해서 값이 없으면 IllegalArgumentException 예외 발생

```java
/* BlogApiController.java */

@RequiredArgsConstructor
@RestController
public class BlogApiController {
	
		... 생략 ...
	
		@GetMapping("/api/articles/{id}")
		// URL 경로에서 값 추출
    public ResponseEntity<ArticleResponse> findArticles(@PathVariable long id) {
        Article article = blogService.findById(id);
        return ResponseEntity.ok()
                .body(new ArticleResponse(article));
    }
}
```

- /api/articles/{id} GET 요청이 오면 블로그 글을 조회하기 위해 매핑할 findArticle() 메서드
- @PathVariable 애너테이션은 URL에서 값을 가져오는 애너테이션
⇒ /api/articles/3 GET 요청을 받으면 id에 3이 들어옴
- id 값은 서비스 클래스의 findById() 메서드로 넘어가 3번 블로그 글을 찾음
- 글을 찾으면 3번 글의 정보를 body에 담아 웹 브라우저로 전송

```java
/* BlogApiControllerTest */

@SpringBootTest
@AutoConfigureMockMvc
class BlogApiControllerTest {
		
		... 생략 ...

    @DisplayName("findArticle: 블로그 글 조회에 성공한다.")
    @Test
    public void findArticle() throws Exception {
        //given
        final String url = "/api/articles/{id}";
        final String title = "title";
        final String content = "content";

        Article savedArticle = blogRepository.save(Article.builder()
                .title(title)
                .content(content)
                .build());

        //when
        final ResultActions resultActions = mockMvc.perform(get(url, savedArticle.getId()));

        //then
        resultActions
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.content").value(content))
                .andExpect(jsonPath("$.title").value(title));
    }
}
```

- 글 조회 API는 제목과 글 내용을 가져옴
