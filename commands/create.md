---
name: create
description: "누락된 TC(Test Case) 자동 생성 - 프로젝트 패턴 기반"
argument-hint: "[파일명] - 특정 파일에 대한 TC만 생성 (선택사항)"
allowed-tools: ["Read", "Write", "Bash", "Glob", "Grep", "Edit"]
---

# TC Create Command

누락된 테스트 케이스를 프로젝트의 기존 패턴에 맞춰 자동 생성합니다.

## 핵심 원칙

**프로젝트 패턴 준수가 최우선입니다.**
- 기존 테스트 파일의 스타일을 분석하고 그대로 따릅니다
- 네이밍 컨벤션, import 패턴, 테스트 구조를 기존과 동일하게 유지합니다

## 실행 단계

### 1. 프로젝트 테스트 패턴 분석

먼저 프로젝트의 기존 테스트 파일들을 분석합니다:

```bash
# 테스트 파일 목록 확인
find . -path "*/test/*" -name "*Test*.java" | head -10
```

**분석 항목**:
1. **파일 위치 구조**: `src/test/java/`와 소스의 패키지 구조 매칭 방식
2. **클래스 네이밍**: `*Test.java`, `*Tests.java`, `Test*.java` 등
3. **메서드 네이밍**: 
   - `test_메서드명_시나리오()` 
   - `테스트메서드명()` 
   - `should_결과_when_조건()` 등
4. **import 패턴**:
   - JUnit 5: `org.junit.jupiter.api.*`
   - Mockito: `org.mockito.*`
   - AssertJ: `org.assertj.core.api.*`
   - 기타 프로젝트 특화 유틸리티
5. **테스트 구조 패턴**:
   - Given-When-Then 주석 사용 여부
   - `@BeforeEach` 설정 방식
   - Mock 객체 초기화 방식 (`@Mock`, `@InjectMocks`, `@MockBean`)
   - Assertion 스타일

기존 테스트 파일 2-3개를 읽어서 패턴을 파악합니다.

### 2. 대상 파일 확인

TC를 생성할 대상을 확인합니다:
- 인자로 특정 파일이 지정된 경우: 해당 파일만 대상
- 인자가 없는 경우: `/tc:check` 결과에서 TC가 누락된 메서드들

### 3. 소스 코드 분석

대상 소스 파일을 읽고 분석합니다:
1. 클래스명, 패키지명 추출
2. public 메서드 시그니처 파악
3. 의존성 (생성자 주입, 필드 주입) 파악
4. 메서드의 동작 파악 (리턴 타입, 예외 등)

### 4. TC 생성

분석된 패턴에 맞춰 TC를 생성합니다:

**테스트 파일이 이미 존재하는 경우**:
- 기존 파일에 누락된 테스트 메서드만 추가
- 기존 파일의 스타일을 그대로 따름

**테스트 파일이 없는 경우**:
- 프로젝트의 다른 테스트 파일 구조를 참고하여 새 파일 생성
- 같은 패키지 구조에 테스트 클래스 생성

### 5. 테스트 실행 및 검증

TC 생성 후 **반드시 테스트를 실행**하여 검증합니다.

#### 5.1 프로젝트 빌드 도구 확인

```bash
# Maven 프로젝트인 경우
ls pom.xml

# Gradle 프로젝트인 경우
ls build.gradle
```

#### 5.2 생성된 TC 실행

**Maven 프로젝트**:
```bash
# 특정 테스트 클래스만 실행
./mvnw test -Dtest=UserServiceTest

# 특정 메서드만 실행
./mvnw test -Dtest=UserServiceTest#test_updateUser_정상케이스
```

**Gradle 프로젝트**:
```bash
# 특정 테스트 클래스만 실행
./gradlew test --tests UserServiceTest

# 특정 메서드만 실행
./gradlew test --tests "UserServiceTest.test_updateUser_정상케이스"
```

#### 5.3 실행 결과 분석

테스트 실행 결과를 확인합니다:

- **성공**: 다음 단계로 진행
- **컴파일 에러**: import 누락, 타입 불일치 등 → 즉시 수정
- **테스트 실패**: assertion 실패, mock 설정 오류 등 → 분석 후 수정

### 6. 실패 시 자동 수정 (최대 3회 반복)

테스트가 실패하면 다음 프로세스를 수행합니다:

#### 6.1 에러 분석

에러 메시지를 분석하여 원인을 파악합니다:

| 에러 유형 | 원인 | 수정 방법 |
|-----------|------|-----------|
| `cannot find symbol` | import 누락 또는 클래스명 오류 | import 추가 또는 클래스명 수정 |
| `incompatible types` | 타입 불일치 | 리턴 타입/파라미터 타입 확인 후 수정 |
| `NullPointerException` | Mock 설정 누락 | when().thenReturn() 추가 |
| `AssertionError` | 예상값과 실제값 불일치 | assertion 로직 수정 |
| `MockitoException` | Mock 객체 설정 오류 | @Mock, @InjectMocks 확인 |

#### 6.2 TC 수정

에러 원인에 따라 TC를 수정합니다:

1. 에러 메시지에서 **실패한 테스트 메서드**와 **라인 번호** 확인
2. 해당 테스트 파일을 읽어서 문제 부분 파악
3. 수정 사항 적용
4. 다시 테스트 실행

#### 6.3 반복 제한

- **최대 3회**까지 수정 시도
- 3회 실패 시 현재 상태 보고 및 수동 검토 요청

```
⚠️ TC 자동 수정 3회 시도 후에도 실패

### 실패한 테스트
- UserServiceTest#test_updateUser_정상케이스

### 마지막 에러
```
java.lang.AssertionError: expected: <User@123> but was: <User@456>
```

### 시도한 수정
1. Mock 리턴값 수정
2. assertEquals → assertThat으로 변경
3. 객체 비교 방식 수정 (equals 기반)

💡 비즈니스 로직 확인이 필요합니다. 수동으로 검토해주세요.
```

### 7. 최종 결과 리포트

```
## TC 생성 및 검증 완료

### 생성된 테스트

#### UserServiceTest.java (수정)
- `test_updateUser_정상케이스()` ✅ 통과
- `test_updateUser_사용자없음_예외발생()` ✅ 통과

#### OrderControllerTest.java (신규 생성)
- `test_getOrder_정상조회()` ✅ 통과
- `test_getOrder_주문없음_404반환()` ✅ 통과 (1회 수정)

### 테스트 실행 결과
- 총 테스트: 4개
- 성공: 4개 ✅
- 실패: 0개

### 적용된 패턴
- 네이밍: `test_메서드명_시나리오()` 형식
- Mock: `@MockBean` + `@Autowired` 패턴
- Assertion: AssertJ 스타일
```

## 생성 규칙

### 기본 테스트 케이스
각 메서드에 대해 최소한 다음 케이스를 생성:
1. **정상 케이스**: 예상된 입력에 대한 정상 동작
2. **예외 케이스**: null 입력, 잘못된 파라미터 등 (해당되는 경우)

### Mock 처리
- 의존성이 있는 경우 Mock 객체 사용
- 프로젝트의 기존 Mock 패턴 따름

### 주의사항
- 생성된 TC는 **자동으로 실행 및 검증**됩니다
- 실패 시 **최대 3회 자동 수정** 시도
- 3회 실패 시 수동 검토 필요 부분을 명확히 표시
