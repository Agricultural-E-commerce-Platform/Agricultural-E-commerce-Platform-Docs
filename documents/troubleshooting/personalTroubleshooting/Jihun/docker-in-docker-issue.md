# 🛠 Docker in Docker 환경에서 Docker 실행 불가 문제 | 2026.04.14


### ❗ 문제 상황

- 튜터님 서버(리눅스 환경)에 Docker 설치 후 실행 시도
- `sudo service docker start` 실행 시 `docker: unrecognized service` 오류 발생
- `sudo dockerd &` 실행 시 `Permission denied (you must be root)` 오류 발생

---

### 🔍 원인

- 해당 서버가 **Docker 컨테이너 내부에서 실행되는 환경 (Docker in Docker)** 이었음
- 컨테이너 내부에서는 기본적으로 Docker 데몬을 실행할 권한이 없음
- `systemd`가 없는 Alpine 기반 환경으로, `service` 및 `sudo` 명령어 사용이 제한됨

---

### 🛠 해결 방법

- Docker in Docker 환경에서는 별도의 설정이 필요하므로 서버 환경 재확인
- 튜터님께 Docker 실행 환경(Daemon 권한 포함) 재설정 요청
- Alpine 환경 특성에 맞게 `sudo` 없이 Docker 명령어를 사용할 수 있도록 서버 구성 변경

---

### ✅ 결과

- 서버 재설정 이후 `sudo` 없이 Docker 명령어 실행 가능
- `docker ps`, `docker-compose --version` 정상 동작 확인
- Docker 기반 배포 환경 정상 구성 완료

---

### 📌 배운 점

- Docker in Docker 환경에서는 일반적인 방식으로 Docker 데몬 실행이 불가능하며, 별도의 권한 설정이 필요하다
- Alpine 기반 컨테이너 환경에서는 `systemd`가 없기 때문에 서비스 실행 방식이 다르다
- 배포 환경에서는 단순 설치보다 **실행 환경(컨테이너 vs 호스트)을 먼저 파악하는 것이 중요하다**