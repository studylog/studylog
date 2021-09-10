# TestCode

테스트 코드의 중요성은 모두 다 알고 있을 것이다. 하지만 테스트코드를 처음 작성을 시작하려면 쉽지 않은게 사실이다. <br/>
그래서 테스트코드를 [ToDoList](https://github.com/Seungpang/example-code/tree/main/todo-server) 를 통해서 테스트 코드도 작성해보고 JUnit과 Mockito에 대해 간략하게 살펴보자!<br/>

+ **JUnit**: Java에 Unit테스트를 위한 프레임워크
+ **Mockito**: Mock 객체를 쉽게 만들고 관리하고 검증할 수 있는 방법을 제공한다.
+ **Mock**: 진짜 객체와 비슷하게 동작하지만 프로그래머가 직접 그 객체의 행동을 관리하는 객체.

##테스트 전략
| 애노테이션 | 설명 | 부모클래스 | Bean |
| -------|---|--------| ---- |
|@SpringBootTest|통합테스트, 전체|IntegrationTest|Bean 전체|
|@WebMvcTest|단위 테스트, Mvc 테스트| MockApiTest|MVC 관련된 Bean|
|@DataJpaTest|단위 테스트, Jpa 테스트| RepositoryTest|JPA 관련 Bean|
|None|단위 테스트, Service 테스트|MockTest|None|
|None|POJO, 도메인 테스트|None|None|

###서비스 테스트
```java
@ExtendWith(MockitoExtension.class)
class TodoServiceTest {

    @Mock
    private TodoRepository todoRepository;

    @InjectMocks // 가짜
    private TodoService todoService;

    @Test
    void searchById() {
        TodoEntity entity = new TodoEntity();
        entity.setId(1L);
        entity.setTitle("test");
        entity.setOrder(0L);
        entity.setCompleted(false);
        Optional<TodoEntity> optional = Optional.of(entity);

        //Mock들의 동작을 정의해줘야 함
        given(this.todoRepository.findById(anyLong()))
            .willReturn(optional);

        TodoEntity actual = this.todoService.searchById(1L);

        TodoEntity expected = optional.get();

        assertEquals(expected.getId(), actual.getId());
        assertEquals(expected.getTitle(), actual.getTitle());
        assertEquals(expected.getOrder(), actual.getOrder());
        assertEquals(expected.getCompleted(), actual.getCompleted());
    }

    @Test
    public void searchByIdFailed() {
        given(this.todoRepository.findById(anyLong()))
            .willReturn(Optional.empty());

        assertThrows(ResponseStatusException.class, () -> {
            this.todoService.searchById(1234L);
        });
    }
}

+ 오직 테스트의 관심사만 테스트를 진행하기 때문에 예외 발생시 디버깅 작업도 명확해진다.
+ 외부 의존도가 낮기 때문에 테스트 하고자하는 부분만 명확하게 테스트가 가능하다.
    + 하지만 이것이 단점이기도 한다. 해당 테스트만 진행하지 외부 의존을 갖는 코드까지 테스트하지 않으니 실제 환경에서 제대로 동작하지 않을 가능성이 있다.


```

### 컨트롤러 테스트
```java
@WebMvcTest(TodoController.class) 
//해당 controller쪽 관련된 bean들만 올려서 테스트 진행할 때 (Controller Advice, Filter들도 올라감)
class TodoControllerTest {

  @Autowired
  MockMvc mvc; //메서드의 호출을 가상으로 만들어 줄때

  @MockBean
  TodoService todoService;

  private TodoEntity expected;

  @BeforeEach
  void setup() {
    this.expected = new TodoEntity();
    this.expected.setId(1L);
    this.expected.setTitle("TEST");
    this.expected.setOrder(0L);
    this.expected.setCompleted(false);
  }

  @Test
  void create() throws Exception {
    when(this.todoService.add(any(TodoRequest.class)))
            .then((i) -> {
              TodoRequest reqeust = i.getArgument(0, TodoRequest.class);
              return new TodoEntity(this.expected.getId(), reqeust.getTitle(),
                      this.expected.getOrder(), this.expected.getCompleted());
            });

    TodoRequest request = new TodoRequest();
    request.setTitle("title");

    ObjectMapper mapper = new ObjectMapper();
    String content = mapper.writeValueAsString(request);

    this.mvc.perform(post("/")
                    .contentType(MediaType.APPLICATION_JSON)
                    .content(content))
            .andDo(print())
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.title").value("title"));
  }
}
```
+ @MockBean 으로 객체를 주입받아 Mocking 작업을 진행한다.
+ 테스트의 관심사는 오직 Request와 그에 따른 Response이다.


### 테스트를 잘 하기 위한 기반
+ 클래스나 메서드가 SRP를 잘 지키고, 크기가 적절히 작아야 한다.
    + 그래야 테스트를 집중력 있게 만들 수 있고 한 메서드에 너무 많은 테스트를 수행하지 않아도 된다.
    + 이게 테스트를 하는 것이 장점이 되기도 한다. (테스트를 하면 자연스럽게 역할이 확인되면서 쪼개짐)
+ 적절한 Mocking을 통한 격리성 확보해야 한다.
    + 단위테스트가 만능은 아니지만, 위의 SRP처럼 해당 메서드의 역할을 정확히 테스트하려면 주변 조건을 적절히 통제해야 한다.
+ 당연히 잘 돌겠지라는 생각말고 꼼꼼히 테스트 && 너무 과도하게 많은 테스트와 코드량이 생기지 않도록 적절히 끊기
    + 테스트코드도 코드 리뷰시에 적절한 테스트를 하는지 확인이 필요하다.
+ 테스트코드 개선을 위한 노력
    + 테스트코드도 리팩토링이 필요하다.
    + 테스트코드의 기법들도 지속적인 고민이 필요하다.(통합테스트 등)

  

------
아직 작성할게 많이 남았지만 여기서 한번 끊고갑니다..
-----

