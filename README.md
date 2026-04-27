# 🌾Agricultural-E-commerce-Platform-Docs

> 농산물 특가 판매 이커머스 플랫폼의 기획, 설계, API, ERD, 회의록, 트러블슈팅 문서를 관리하는 문서 레포지토리입니다.

---

## 📄 핵심 문서

| 문서                                                          | 설명 |
|-------------------------------------------------------------| --- |
| [서비스 정책](./documents/policy/service-policy/ServicePolicy.md) | 회원, 상품, 장바구니, 주문, 쿠폰, 검색 정책 정리 |
| [팀 협업 정책](./documents/policy/team-policy/TeamPolicy.md)                          | Git 전략, PR 규칙, 테스트 및 협업 방식 정리 |
| [ERD 설계](./documents/erd/ERD.md)                                     | 데이터베이스 구조, 주요 관계, 설계 포인트 정리 |
| [API 명세](./documents/api/API.md)                                | 주요 API 목록, 요청/응답 형식, 에러 코드 정리 |

---

## 🚨 트러블슈팅

| 구분                                                               | 설명 |
|------------------------------------------------------------------| --- |
| [팀 트러블슈팅](./documents/troubleshooting/teamTroubleshooting/)      | 프로젝트 진행 중 팀 단위로 해결한 주요 기술 이슈 |
| [개인 트러블슈팅](./documents/troubleshooting/personalTroubleshooting/) | 팀원별 개인 학습 및 문제 해결 기록 |

---

## 📝 회의록

프로젝트 진행 과정에서 논의한 내용과 의사결정 과정을 기록합니다.

- [회의록 바로가기](./documents/meetings/)

---

## 🔑 주요 기술 키워드

- Java 17
- Spring Boot 3.x
- Spring Security + JWT
- Spring Data JPA
- QueryDSL
- MySQL
- Redis
- Caffeine Cache
- Quartz Scheduler
- Docker
- GitHub Actions

---

## 🧩 핵심 구현 포인트

### 1. 선착순 쿠폰 발급 동시성 제어

- Redis 분산 락 기반 쿠폰 발급 제어
- 사용자 중복 발급 방지
- 쿠폰 수량 초과 발급 방지

### 2. 주문 재고 차감 정합성 보장

- 상품 ID 기준 Redis 락 적용
- DB 조건 검증으로 재고 음수 방지
- 장바구니는 재고 예약이 아닌 임시 저장소로 설계

### 3. 검색 성능 개선

- 검색 API v1 / v2 분리
- v1: 캐시 미적용
- v2: Caffeine Cache 적용
- Redis Sorted Set 기반 인기 검색어 집계

---

## 📅 프로젝트 기간

```text
2026.04.08 ~ 2026.04.28
```

---

## 📅 마일스톤

프로젝트는 설계 → 핵심 기능 개발 → 동시성/캐싱 구현 → 배포 및 발표 순서로 진행되었습니다.

| 단계 | 기간 | 핵심 목표 |
| --- | --- | --- |
| Week 1 | 4/8 ~ 4/14 | 설계 및 핵심 기능 개발 |
| Week 2 | 4/15 ~ 4/21 | 캐싱 및 동시성 제어 구현 |
| Final | 4/22 ~ 4/28 | 테스트, 배포, 문서화 및 발표 |

- [전체 마일스톤 보기](./documents/milestone/Milestone.md)


---

## 👥 팀원

| 이름 | 역할 | 담당 도메인 |
| --- | --- | --- |
| 정지훈 | 팀장 / DevOps | Auth, User, Common, Infra |
| 정은지 | 부팀장 | Product, Search |
| 김예은 | QA / 서기 | Order, Cart |
| 이중현 | QA 총괄 / DB | Coupon, Admin |

---


## 📌 문서 관리 기준

- 코드와 문서를 분리하여 관리합니다.
- 서비스 정책, ERD, API 명세는 변경 시 함께 업데이트합니다.
- 트러블슈팅은 문제 상황, 원인, 해결 방법, 결과를 기준으로 작성합니다.
- 회의록은 프로젝트 의사결정 이력을 남기는 용도로 관리합니다.
