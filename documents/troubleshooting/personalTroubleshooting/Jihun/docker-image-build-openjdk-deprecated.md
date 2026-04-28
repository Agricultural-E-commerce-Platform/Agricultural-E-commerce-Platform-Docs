# 🛠 Docker 이미지 빌드 실패 | 2026.04.15

### ❗ 문제 상황

- main 브랜치 머지 이후 CI 파이프라인에서 Docker 이미지 빌드 실패
- `openjdk:17-jdk-slim: not found` 오류 발생

---

### 🔍 원인

- `openjdk` 공식 Docker 이미지가 deprecated 되어 Docker Hub에서 제거됨
- Dockerfile 실행 단계에서 `openjdk:17-jdk-slim` 이미지를 사용하고 있었음

---

### 🛠 해결 방법

- Dockerfile 베이스 이미지 변경

```dockerfile
# 기존
FROM openjdk:17-jdk-slim

# 변경
FROM eclipse-temurin:17-jre-jammy
```
- eclipse-temurin은 OpenJDK의 공식 권장 대체 이미지로, 안정적으로 유지 관리됨

### ✅ 결과
- Docker 이미지 빌드 정상 성공
- CI 파이프라인 정상 동작 확인

### 📌 배운 점
- Docker 베이스 이미지도 deprecated 될 수 있으므로 지속적인 확인이 필요하다
- 공식 이미지 정책 변경에 대비하여 대체 이미지를 파악해두는 것이 중요하다
- OpenJDK 기반 프로젝트에서는 eclipse-temurin 이미지 사용이 권장된다