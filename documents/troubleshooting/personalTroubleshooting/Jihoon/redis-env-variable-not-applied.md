# 🛠 Redis 연결 실패 (환경변수 적용 문제) | 2026.04.27

### ❗ 문제 상황

- AWS EC2 배포 후 Redis 연결 시 `UnknownHostException` 발생
- `your-elasticache-endpoint`로 접속 시도하는 오류 발생
- `/actuator/health` 상태가 DOWN으로 표시됨

---

### 🔍 원인

- `docker-compose`에서 `${REDIS_HOST}` 환경변수가 정상적으로 치환되지 않음
- `.env.prod` 파일을 사용했지만, `docker-compose`는 기본적으로 `.env` 파일만 자동 인식
- application 설정과 환경변수 간 우선순위를 잘못 이해하고 있었음

---

### 🛠 해결 방법

- `.env.prod`를 `.env`로 통합하여 `docker-compose`에서 자동 로딩되도록 수정
- `docker-compose` 환경변수 치환 방식 정리
- 컨테이너를 `--force-recreate` 옵션으로 재생성하여 설정 반영

```bash
docker-compose up -d --force-recreate
```

### ✅ 결과
- Redis 연결 정상화
- /actuator/health 상태 UP 확인
- 캐시 및 동시성 기능 정상 동작
### 📌 배운 점
- docker-compose의 ${변수}와 env_file은 동작 방식이 다르다
- 환경변수는 컨테이너 생성 시점에만 적용되므로, 변경 시 재생성이 필요하다
- 배포 환경에서는 설정 우선순위와 적용 시점을 명확히 이해하는 것이 중요하다
