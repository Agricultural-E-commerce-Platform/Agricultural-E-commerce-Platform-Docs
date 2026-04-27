# 🛠 통합 테스트 로컬 실행 환경 구성 | 2026.04.24

### ❗ 문제 상황

- `./gradlew integrationTest` 실행 시 통합 테스트가 단위 테스트와 함께 실행되거나 별도로 실행되지 않음
- 통합 테스트 실행 시 DB에 이전 테스트 데이터가 남아 테스트 간 데이터 격리가 되지 않아 실패 발생
- `coupons` 테이블의 `issued_quantity` 컬럼이 DB에 남아 있어, 엔티티(`issued_count`)와 불일치로 insert 오류 발생

---

### 🔍 원인

- `build.gradle`에 `integrationTest` 태스크가 없어 `@Tag("integration")` 기반 테스트를 분리 실행할 수 없었음
- `application-test.yml` 파일이 없어 `@ActiveProfiles("test")` 환경에서 설정 값을 읽지 못해 `PlaceholderResolutionException` 발생
- 테스트 환경에서 `ddl-auto`가 `create-drop`으로 설정되지 않아 테스트 실행 시 DB가 초기화되지 않음
- 엔티티 컬럼명 변경(`issued_quantity → issued_count`) 이후 DB 스키마가 갱신되지 않아 컬럼 불일치 발생

---

### 🛠 해결 방법

- `build.gradle`에 `integrationTest` 태스크 추가  
  → `includeTags 'integration'` 설정으로 단위 테스트와 분리

- `application-test.yml` 생성  
  → `ddl-auto: create-drop` 설정으로 테스트마다 DB 자동 초기화

- 통합 테스트 클래스에 `@ActiveProfiles("test")` 적용

- DB에서 기존 `issued_quantity` 컬럼 제거 후  
  `issued_count` 컬럼에 `DEFAULT 0` 설정

---

### ✅ 결과

- `./gradlew integrationTest`로 통합 테스트만 별도 실행 가능
- IntelliJ Gradle > Other 탭에서도 `integrationTest` 태스크 실행 가능
- 테스트 간 DB 격리가 보장되어 전체 통합 테스트 정상 통과

---

### 📌 배운 점

- 통합 테스트와 단위 테스트는 실행 목적이 다르므로 **태스크 분리와 프로파일 관리가 필수적이다**
- `ddl-auto: create-drop` 설정은 테스트 환경에서 데이터 격리를 보장하는 효과적인 방법이다
- 엔티티 변경 시 DB 스키마도 반드시 함께 관리해야 하며, 테스트 환경에서는 자동 초기화를 통해 불일치를 방지할 수 있다