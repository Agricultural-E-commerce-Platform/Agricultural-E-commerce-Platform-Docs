# 📊 ERD 설계 문서

## 1. ERD 다이어그램

ERD 이미지

---

## 2. 데이터 흐름

본 프로젝트의 주요 데이터 흐름은 다음과 같습니다.

```text
회원 → 상품 조회 → 장바구니 → 주문 → 쿠폰 적용 → 주문 완료 처리
```

주문은 장바구니 전체 상품을 기준으로 생성되며, 주문 성공 시 재고 차감, 쿠폰 사용 처리, 장바구니 초기화가 함께 수행됩니다.

---

## 3. 주요 설계 원칙

- 모든 테이블은 `id`를 기본키(PK)로 사용합니다.
- 모든 테이블은 `created_at` 컬럼을 가집니다.
- `users` 테이블은 회원 정보 변경 이력을 위해 `updated_at` 컬럼을 추가로 가집니다.
- 장바구니 상품 삭제는 하드 딜리트 방식으로 처리합니다.
- 주문 상품과 장바구니 상품은 가격 스냅샷을 저장합니다.
- 선착순 쿠폰 중복 발급 방지를 위해 `user_coupons`에 `(user_id, coupon_id)` 유니크 제약을 둡니다.
- 상품 재고 차감은 Redis 분산 락과 DB 조건부 UPDATE를 함께 사용하여 재고 정합성을 보장합니다.
- 선착순 쿠폰 발급은 Redis 분산 락을 사용하여 동시성 문제를 제어합니다.

---

## 4. 테이블 간 연관 관계

| 관계 | 설명 |
| --- | --- |
| `users : carts` | 1 : 1 |
| `users : orders` | 1 : N |
| `users : user_coupons` | 1 : N |
| `coupons : user_coupons` | 1 : N |
| `carts : cart_items` | 1 : N |
| `products : cart_items` | 1 : N |
| `orders : order_items` | 1 : N |
| `products : order_items` | 1 : N |
| `orders : user_coupons` | N : 0..1 |

---

## 5. 주요 테이블 설명

### users

회원 정보를 저장하는 테이블입니다.

| 컬럼 | 설명 |
| --- | --- |
| `email` | 로그인 ID로 사용되는 이메일, 유니크 제약 적용 |
| `password` | 암호화된 비밀번호 |
| `role` | 사용자 권한, `ADMIN` / `USER` |
| `updated_at` | 회원 정보 수정일 |

---

### products

농산물 상품 정보를 저장하는 테이블입니다.

| 컬럼 | 설명 |
| --- | --- |
| `type` | 상품 유형, `NORMAL` / `SPECIAL` |
| `normal_price` | 정상가 |
| `sale_price` | 판매가 |
| `special_price` | 특가 가격 |
| `stock` | 상품 재고 |
| `status` | 상품 상태 |
| `sale_start_time` | 특가 시작 시간 |
| `sale_end_time` | 특가 종료 시간 |
| `version` | 버전 관리 컬럼 |

상품 상태는 다음 값을 가집니다.

```text
READY / ON_SALE / SALE_ENDED / SOLD_OUT / HIDDEN
```

---

### carts

사용자별 장바구니를 저장하는 테이블입니다.

| 컬럼 | 설명 |
| --- | --- |
| `user_id` | 장바구니 소유 회원 ID, 유니크 제약 적용 |

회원은 하나의 장바구니만 가질 수 있습니다.

---

### cart_items

장바구니에 담긴 상품을 저장하는 테이블입니다.

| 컬럼 | 설명 |
| --- | --- |
| `cart_id` | 장바구니 ID |
| `product_id` | 상품 ID |
| `price` | 장바구니에 담긴 시점의 상품 가격 |
| `quantity` | 상품 수량 |

장바구니는 재고를 예약하지 않으며, 실제 재고 차감은 주문 시점에 수행됩니다.

---

### coupons

선착순 쿠폰 정보를 저장하는 테이블입니다.

| 컬럼 | 설명 |
| --- | --- |
| `discount_price` | 할인 금액 |
| `total_count` | 총 발급 가능 수량 |
| `issued_count` | 현재 발급된 수량 |
| `start_time` | 발급 시작 시간 |
| `end_time` | 발급 종료 시간 |

---

### user_coupons

사용자에게 발급된 쿠폰을 저장하는 테이블입니다.

| 컬럼 | 설명 |
| --- | --- |
| `user_id` | 쿠폰을 발급받은 회원 ID |
| `coupon_id` | 발급된 쿠폰 ID |
| `status` | 쿠폰 상태 |
| `expired_at` | 쿠폰 만료 시간 |

쿠폰 상태는 다음 값을 가집니다.

```text
AVAILABLE / USED / EXPIRED
```

중복 발급 방지를 위해 `(user_id, coupon_id)` 유니크 제약을 적용합니다.

---

### orders

주문 정보를 저장하는 테이블입니다.

| 컬럼 | 설명 |
| --- | --- |
| `user_id` | 주문 회원 ID |
| `user_coupon_id` | 주문에 사용된 쿠폰 ID, 선택값 |
| `total_price` | 최초 상품 총 금액 |
| `discount_price` | 총 할인 금액 |
| `final_price` | 최종 결제 금액 |
| `status` | 주문 상태 |

주문 상태는 다음 값을 가집니다.

```text
COMPLETED
```

---

### order_items

주문 상품 정보를 저장하는 테이블입니다.

| 컬럼 | 설명 |
| --- | --- |
| `order_id` | 주문 ID |
| `product_id` | 상품 ID |
| `price` | 주문 시점의 상품 가격 |
| `quantity` | 주문 수량 |

상품 가격은 주문 시점의 스냅샷으로 저장하여, 이후 상품 가격이 변경되어도 주문 내역이 유지되도록 설계했습니다.

---

## 6. 설계 포인트

### 1. 주문 데이터 스냅샷 저장

`orders`와 `order_items`에는 주문 당시의 금액 정보를 저장합니다.

이를 통해 상품 가격이나 할인 정책이 변경되더라도 기존 주문 내역의 금액이 변하지 않도록 보장합니다.

---

### 2. 장바구니와 주문 책임 분리

장바구니는 상품을 임시로 담는 역할만 수행합니다.

재고 차감은 장바구니 단계가 아니라 주문 생성 시점에 수행하여, 실제 구매 시점의 상품 상태와 재고를 기준으로 검증합니다.

---

### 3. 선착순 쿠폰 중복 발급 방지

`user_coupons` 테이블에 `(user_id, coupon_id)` 유니크 제약을 적용하여 같은 사용자가 동일 쿠폰을 중복 발급받을 수 없도록 설계했습니다.

Redis 분산 락으로 동시 발급 요청을 제어하고, DB 제약으로 중복 발급을 한 번 더 방어합니다.

---

### 4. 재고 정합성 보장

상품 주문 시 동일 상품에 대한 동시 주문 요청을 Redis 분산 락으로 제어합니다.

이후 DB 조건부 UPDATE를 통해 재고가 충분한 경우에만 차감되도록 하여 재고 음수 발생을 방지합니다.

---

### 5. 상태 기반 도메인 관리

상품, 주문, 쿠폰은 상태값을 기준으로 비즈니스 규칙을 제어합니다.

- 상품 상태: 판매 대기, 판매 중, 판매 종료, 품절, 비공개
- 쿠폰 상태: 사용 가능, 사용 완료, 만료
- 주문 상태: 결제 완료

---

## 7. DBML

```dbml
Table users {
  id bigint [pk]
  email varchar [not null, unique]
  password varchar [not null]
  name varchar [not null]
  phone varchar [not null]
  address varchar [not null]
  role varchar [not null] // ADMIN / USER
  updated_at datetime
  created_at datetime [not null]
}

Table products {
  id bigint [pk]
  name varchar [not null]
  type varchar [not null] // NORMAL / SPECIAL
  normal_price bigint [not null]
  sale_price bigint [not null]
  special_price bigint
  stock int [not null]
  status varchar [not null] // READY / ON_SALE / SALE_ENDED / SOLD_OUT / HIDDEN
  image_url varchar
  sale_start_time datetime
  sale_end_time datetime
  version bigint
  created_at datetime [not null]
}

Table carts {
  id bigint [pk]
  user_id bigint [not null, unique]
  created_at datetime [not null]
}

Table cart_items {
  id bigint [pk]
  cart_id bigint [not null]
  product_id bigint [not null]
  price bigint [not null]
  quantity int [not null]
  created_at datetime [not null]
}

Table coupons {
  id bigint [pk]
  name varchar [not null]
  discount_price bigint [not null]
  total_count int [not null]
  issued_count int [not null]
  start_time datetime [not null]
  end_time datetime [not null]
  created_at datetime [not null]
}

Table user_coupons {
  id bigint [pk]
  user_id bigint [not null]
  coupon_id bigint [not null]
  status varchar [not null] // AVAILABLE / USED / EXPIRED
  expired_at datetime [not null]
  created_at datetime [not null]

  indexes {
    (user_id, coupon_id) [unique]
  }
}

Table orders {
  id bigint [pk]
  user_id bigint [not null]
  user_coupon_id bigint
  total_price bigint [not null]
  discount_price bigint [not null, default: 0]
  final_price bigint [not null]
  status varchar [not null] // COMPLETED
  created_at datetime [not null]
}

Table order_items {
  id bigint [pk]
  order_id bigint [not null]
  product_id bigint [not null]
  price bigint [not null]
  quantity int [not null]
  created_at datetime [not null]
}

Ref: carts.user_id > users.id
Ref: cart_items.cart_id > carts.id
Ref: cart_items.product_id > products.id

Ref: user_coupons.user_id > users.id
Ref: user_coupons.coupon_id > coupons.id

Ref: orders.user_id > users.id
Ref: orders.user_coupon_id > user_coupons.id

Ref: order_items.order_id > orders.id
Ref: order_items.product_id > products.id
```