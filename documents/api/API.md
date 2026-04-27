# 📡 API 명세 (API Specification)

## 🔗 기본 정보

- Base URL: `http://localhost:8080`
- 인증 방식: JWT Bearer Token
- 요청 헤더:
  ```text
  Authorization: Bearer {accessToken}
  ```
- 응답 형식: `application/json`

---

## 🎯 API 설계 특징

- RESTful API 설계 원칙 준수
- 도메인 중심 URL 구조 (`/api/{domain}`)
- 인증이 필요한 API는 JWT 기반 인증 적용
- 모든 응답은 공통 포맷으로 반환
- 동시성 제어가 필요한 API는 Redis 분산 락 적용

---

## ⚡ 핵심 기능 요약

- 회원가입 / 로그인 (JWT 인증)
- 상품 조회 및 검색 (v1 / v2 캐시 비교 구조)
- 장바구니 기능
- 주문 생성 (재고 동시성 제어)
- 선착순 쿠폰 발급 (Redis 락)
- 인기 검색어 (Redis Sorted Set)

---

## 🔍 검색 API 버전 전략

본 프로젝트는 성능 비교를 위해 검색 API를 v1 / v2로 분리했습니다.

| 버전 | 특징 |
| --- | --- |
| v1 | DB 직접 조회 (캐시 미적용) |
| v2 | Caffeine 캐시 적용 + 인기 검색어 집계 |

👉 동일 기능을 캐시 유무로 비교하여 성능 개선을 검증하는 구조입니다.

---

## 🧾 공통 응답 형식

### 성공 응답

```json
{
  "status": 200,
  "message": "성공 메시지",
  "data": {}
}
```

### 실패 응답

```json
{
  "status": 400,
  "message": "에러 메시지",
  "data": null
}
```

---

## 🚨 에러 코드 정의

| 코드 | 설명 |
| --- | --- |
| 400 | 잘못된 요청 |
| 401 | 인증 실패 |
| 403 | 권한 없음 |
| 404 | 리소스 없음 |
| 409 | 상태 충돌 (재고 부족, 중복 등) |
| 429 | 요청 과다 (Redis Lock 실패) |
| 500 | 서버 오류 |

---

## 📌 주요 API 목록

| Method | URL | 인증 | 설명 | 특징 |
| --- | --- | --- | --- | --- |
| GET | `/actuator/health` | ❌ | 서버 상태 | 모니터링 |
| POST | `/api/auth/signup` | ❌ | 회원가입 | |
| POST | `/api/auth/signin` | ❌ | 로그인 | JWT 발급 |
| PATCH | `/api/users/me` | ✅ | 회원 수정 | |
| GET | `/api/products` | ❌ | 상품 목록 | 페이징 |
| GET | `/api/products/{id}` | ❌ | 상품 상세 | |
| GET | `/api/v1/products/search` | ❌ | 검색 v1 | ❌ 캐시 |
| GET | `/api/v2/products/search` | ❌ | 검색 v2 | 🔴 캐시 |
| GET | `/api/products/search/popular` | ❌ | 인기 검색어 | 🔴 Redis |
| POST | `/api/carts` | ✅ | 장바구니 담기 | |
| GET | `/api/carts` | ✅ | 장바구니 조회 | |
| PATCH | `/api/carts/{id}` | ✅ | 수량 변경 | |
| DELETE | `/api/carts/items/{id}` | ✅ | 삭제 | |
| POST | `/api/orders` | ✅ | 주문 생성 | 🔴 재고 락 |
| GET | `/api/orders` | ✅ | 주문 조회 | |
| POST | `/api/coupons` | ✅ (관리자) | 쿠폰 생성 | |
| GET | `/api/coupons` | ✅ (관리자) | 쿠폰 목록 | |
| POST | `/api/coupons/{id}/issue` | ✅ | 쿠폰 발급 | 🔴 Redis 락 |
| GET | `/api/coupons/me` | ✅ | 내 쿠폰 | |

---

## 🔥 핵심 API 상세 설명

### 🛒 주문 생성 API

```http
POST /api/orders
```

#### 특징

- 장바구니 전체 상품 기반 주문 생성
- 최소 주문 금액 20,000원
- 쿠폰 1개 사용 가능
- 특가 상품 포함 시 쿠폰 사용 불가

#### 동시성 제어

- Redis 분산 락으로 동일 상품 주문 요청 제어
- DB 조건 검증으로 재고 음수 방지

---

### 🎟 쿠폰 발급 API

```http
POST /api/coupons/{couponId}/issue
```

#### 특징

- 사용자당 1회 발급
- 선착순 수량 제한
- 유효 기간: 발급일 + 5일

#### 동시성 제어

- Redis 분산 락 적용
- 중복 발급 및 초과 발급 방지

---

### 🔍 검색 API (v2)

```http
GET /api/v2/products/search
```

#### 특징

- Caffeine 기반 로컬 캐시 적용
- 동일 키워드 요청 시 캐시 반환
- TTL: 3~5분

#### 추가 기능

- Redis Sorted Set 기반 인기 검색어 집계
- 10분 단위 집계 + 중복 제거

---

## 🚀 정리

본 API는 단순 CRUD를 넘어,

- Redis 기반 동시성 제어
- 캐시 적용을 통한 성능 개선
- 실무형 이커머스 흐름 설계

를 중심으로 설계되었습니다.