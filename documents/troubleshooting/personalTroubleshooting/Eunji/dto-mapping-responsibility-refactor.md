# 🛠  Product 엔티티 → DTO 변환 구조 개선 | 2026.04.14


### ❗ 문제 상황

`ProductService`에서 엔티티를 DTO로 변환할 때,  
서비스 레이어에서 직접 DTO를 생성하며 모든 필드를 나열하고 있었다.

```
return products.map(product -> new ProductListResponse(
    product.getId(),
    product.getName(),
    product.getType(),
    product.getNormalPrice(),
    product.getSalePrice(),
    product.getSpecialPrice(),
    product.getStock(),
    product.getStatus(),
    product.getImageUrl(),
    product.getCreatedAt()
));
```
- 필드가 많아질수록 서비스 코드가 길어짐
- DTO 필드 변경 시 서비스 코드까지 수정 필요
- 변환 로직이 서비스에 존재하여 책임이 모호함

### 🔍 원인
- 엔티티 → DTO 변환 책임이 서비스 레이어에 위치
- DTO는 단순 데이터 구조로만 사용되고 변환 책임이 없음
- 결과적으로 서비스 레이어가 과도한 역할을 담당

### 🛠 해결 방법
- DTO 내부에 정적 팩토리 메서드(from)를 추가하여 엔티티 → DTO 변환 책임을 DTO로 이동

```
// ProductListResponse.java
public static ProductListResponse from(Product product) {
return new ProductListResponse(
product.getId(),
product.getName(),
product.getType(),
product.getNormalPrice(),
product.getSalePrice(),
product.getSpecialPrice(),
product.getStock(),
product.getStatus(),
product.getImageUrl(),
product.getCreatedAt()
);
}
```
```
// ProductDetailResponse.java
public static ProductDetailResponse from(Product product) {
return new ProductDetailResponse(
product.getId(),
product.getName(),
product.getType(),
product.getStatus(),
product.getNormalPrice(),
product.getSalePrice(),
product.getSpecialPrice(),
product.getStock(),
product.getImageUrl(),
product.getCreatedAt()
);
}
```
- 서비스에서는 DTO를 직접 생성하지 않고, from() 메서드 사용
```
return products.map(ProductListResponse::from);
return ProductDetailResponse.from(product);
```

### ✅ 결과
- 서비스 레이어는 비즈니스 로직에만 집중
- DTO 변경 시 DTO 클래스만 수정하면 되어 유지보수성 향상
- 메서드 참조 사용으로 코드 가독성 개선

### 📌 배운 점
- DTO 변환 로직은 서비스가 아닌 DTO가 책임지는 것이 적절하다
- 정적 팩토리 메서드는 객체 생성 책임을 명확하게 분리할 수 있다
- 역할 분리를 통해 코드의 가독성과 유지보수성을 동시에 개선할 수 있다