---
name: check
description: "staged 변경사항에 대한 TC(Test Case) 존재 여부 확인"
argument-hint: "[파일명] - 특정 파일만 확인 (선택사항)"
allowed-tools: ["Read", "Bash", "Glob", "Grep"]
---

# TC Check Command

staged 변경사항에 대해 관련 테스트 케이스가 존재하는지 확인합니다.

## 실행 단계

### 1. 변경사항 분석

먼저 staged 변경사항을 확인합니다:

```bash
# 특정 파일이 지정된 경우
git diff --cached --name-only -- "*.java" | grep -E "지정된파일"

# 전체 staged 파일인 경우
git diff --cached --name-only -- "*.java"
```

**중요**: `src/test/` 경로의 파일은 제외하고 소스 파일만 분석합니다.

### 2. 변경된 클래스/메서드 식별

각 변경된 Java 파일에 대해:
1. 파일을 읽어서 클래스명 추출
2. `git diff --cached` 로 변경된 메서드 식별
3. 새로 추가된 메서드와 수정된 메서드 구분

### 3. 프로젝트 테스트 구조 파악

프로젝트의 테스트 파일 위치를 파악합니다:
- `src/test/java/` 하위 구조 확인
- 기존 테스트 파일 패턴 분석 (예: `*Test.java`, `*Tests.java`)

### 4. 관련 TC 검색

변경된 각 클래스에 대해:
1. 대응하는 테스트 클래스 파일 검색
2. 테스트 파일 내에서 변경된 메서드를 테스트하는 TC 존재 여부 확인
3. `@Test` 어노테이션이 있는 메서드 중 관련 메서드 호출 여부 확인

### 5. 결과 리포트 출력

다음 형식으로 결과를 출력합니다:

```
## TC 확인 결과

### 변경된 파일
- UserService.java (3개 메서드 변경)
- OrderController.java (1개 메서드 추가)

### TC 커버리지

| 소스 파일 | 변경 메서드 | TC 파일 | TC 존재 |
|-----------|-------------|---------|---------|
| UserService.java | createUser() | UserServiceTest.java | ✅ 있음 |
| UserService.java | updateUser() | UserServiceTest.java | ❌ 없음 |
| UserService.java | deleteUser() | - | ❌ TC 파일 없음 |
| OrderController.java | getOrder() | OrderControllerTest.java | ✅ 있음 |

### 요약
- 총 변경 메서드: 4개
- TC 존재: 2개 (50%)
- TC 누락: 2개

💡 누락된 TC를 생성하려면 `/tc:create` 명령을 사용하세요.
```

## 주의사항

- 테스트 파일(`*Test.java`, `*Tests.java`)의 변경은 분석에서 제외
- private 메서드는 직접 테스트 대상이 아니므로 별도 표시
- 인터페이스 메서드는 구현체에서 테스트되므로 구현체 기준으로 확인
