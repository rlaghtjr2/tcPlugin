# TC Helper Plugin

Git 변경사항에 대한 TC(Test Case) 확인 및 자동 생성 플러그인

## Features

- **`/tc:check`**: staged 변경사항에 대한 TC 존재 여부 확인
- **`/tc:create`**: 누락된 TC 자동 생성 (프로젝트 패턴 기반)

## Usage

### TC 확인
```
/tc:check                    # staged 변경사항 전체 확인
/tc:check UserService.java   # 특정 파일만 확인
```

### TC 생성
```
/tc:create                   # 누락된 TC 자동 생성
/tc:create UserService.java  # 특정 파일에 대한 TC 생성
```

## How It Works

1. **패턴 분석**: 프로젝트의 기존 테스트 파일을 분석하여 패턴 파악
   - 테스트 파일 위치 구조
   - 클래스/메서드 네이밍 컨벤션
   - import 패턴 (Mockito, AssertJ 등)
   - 테스트 구조 패턴

2. **TC 생성**: 분석된 패턴에 맞춰 일관된 스타일의 TC 생성

## Requirements

- Java 프로젝트
- JUnit 5
- Git
