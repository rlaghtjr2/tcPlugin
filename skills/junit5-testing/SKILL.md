---
name: JUnit 5 Testing
description: "JUnit 5 테스트 작성, 테스트 패턴 분석, TC 구조 설계에 대한 지식을 제공합니다. 사용자가 Java 테스트 작성, 테스트 커버리지 확인, 테스트 패턴 분석을 요청할 때 활성화됩니다."
version: 1.0.0
---

# JUnit 5 Testing Skill

Java 프로젝트의 테스트 케이스 작성 및 분석을 위한 지식을 제공합니다.

## JUnit 5 핵심 어노테이션

### 테스트 정의
- `@Test` - 테스트 메서드 표시
- `@DisplayName("설명")` - 테스트 설명
- `@Disabled` - 테스트 비활성화
- `@Nested` - 중첩 테스트 클래스

### 생명주기
- `@BeforeAll` - 모든 테스트 전 1회 실행 (static)
- `@AfterAll` - 모든 테스트 후 1회 실행 (static)
- `@BeforeEach` - 각 테스트 전 실행
- `@AfterEach` - 각 테스트 후 실행

### 파라미터화 테스트
- `@ParameterizedTest` - 파라미터화 테스트
- `@ValueSource` - 단일 값 배열
- `@CsvSource` - CSV 형식 데이터
- `@MethodSource` - 메서드에서 데이터 제공

## Mockito 패턴

### 기본 사용
```java
@Mock
private UserRepository userRepository;

@InjectMocks
private UserService userService;

@BeforeEach
void setUp() {
    MockitoAnnotations.openMocks(this);
}
```

### Stubbing
```java
when(userRepository.findById(1L)).thenReturn(Optional.of(user));
when(userRepository.save(any())).thenAnswer(inv -> inv.getArgument(0));
doThrow(new RuntimeException()).when(mock).method();
```

### 검증
```java
verify(userRepository).save(any());
verify(userRepository, times(2)).findById(anyLong());
verify(userRepository, never()).delete(any());
```

## 테스트 구조 패턴

### Given-When-Then
```java
@Test
void should_returnUser_when_validId() {
    // Given
    User expected = new User(1L, "test");
    when(repository.findById(1L)).thenReturn(Optional.of(expected));
    
    // When
    User result = service.getUser(1L);
    
    // Then
    assertThat(result).isEqualTo(expected);
}
```

### Arrange-Act-Assert
```java
@Test
void testGetUser() {
    // Arrange
    User expected = new User(1L, "test");
    when(repository.findById(1L)).thenReturn(Optional.of(expected));
    
    // Act
    User result = service.getUser(1L);
    
    // Assert
    assertEquals(expected, result);
}
```

## 네이밍 컨벤션

### 일반적인 패턴들
- `test_메서드명_시나리오()` - 언더스코어 구분
- `testMethodName()` - camelCase
- `should_결과_when_조건()` - BDD 스타일
- `givenX_whenY_thenZ()` - Given-When-Then

### 예시
```java
// 언더스코어 스타일
void test_createUser_정상생성()
void test_createUser_중복이메일_예외발생()

// should-when 스타일
void should_createUser_when_validInput()
void should_throwException_when_duplicateEmail()

// camelCase 스타일
void testCreateUserSuccess()
void testCreateUserDuplicateEmailThrowsException()
```

## Assertion 스타일

### JUnit 5 기본
```java
assertEquals(expected, actual);
assertTrue(condition);
assertThrows(Exception.class, () -> method());
assertAll(
    () -> assertEquals(1, result.getId()),
    () -> assertEquals("test", result.getName())
);
```

### AssertJ (유창한 API)
```java
assertThat(result).isNotNull();
assertThat(result.getName()).isEqualTo("test");
assertThat(list).hasSize(3).contains("a", "b");
assertThatThrownBy(() -> method())
    .isInstanceOf(IllegalArgumentException.class)
    .hasMessage("error");
```

## Spring Boot Test

### 슬라이스 테스트
- `@WebMvcTest` - Controller 레이어
- `@DataJpaTest` - Repository 레이어
- `@JsonTest` - JSON 직렬화

### 통합 테스트
```java
@SpringBootTest
@AutoConfigureMockMvc
class IntegrationTest {
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private ExternalService externalService;
}
```

## 테스트 파일 구조 분석 가이드

프로젝트의 테스트 패턴을 분석할 때:

1. **파일 위치 확인**: `src/test/java/` 구조
2. **네이밍 패턴 확인**: `*Test.java`, `*Tests.java` 등
3. **import 패턴 확인**: 사용하는 라이브러리
4. **구조 패턴 확인**: Given-When-Then, AAA 등
5. **Mock 패턴 확인**: `@Mock`, `@MockBean` 등
6. **Assertion 패턴 확인**: JUnit, AssertJ 등

기존 테스트 파일 2-3개를 읽어서 일관된 패턴을 파악합니다.
