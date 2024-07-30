---
layout: post
categories: 공부 스프링
---

### QueryDSL에서 동적 정렬 구현하기
Pageable을 사용하면 페이징과 정렬을 쉽게 처리할 수 있습니다. 하지만 때때로 페이징 없이 정렬만 필요한 경우도 있습니다. 예를 들어, 현재 회사에서는 센서 데이터를 표시할 때 시간 정렬은 필요하지만 페이징은 필요하지 않습니다. 이런 경우 Pageable에 정렬 정보만 제공할 수 있지만, QueryDSL을 사용하여 동적으로 정렬을 적용하려는 경우에는 Pageable을 사용하지 않고 정렬 기능만 따로 구현하는 방법이 더 유용할 수 있습니다.

#### 요청 형식
정렬 요청은 Pageable을 사용하고 있는 다른 요청들과 일관성을 유지하기 위해 `sort=id,desc&createdAt`과 같은 형식을 사용합니다.

```java
// 요청 파라미터
private String[] sort;

```

이제 이 파라미터를 `QueryDSL`에서 동적으로 정렬하는 방법을 살펴보겠습니다.

#### 정렬 처리 기능
1. 정렬 정보가 없을 때 처리: 정렬 정보가 제공되지 않을 경우 기본 동작을 유지합니다.
2. 정렬 필드와 방향 분리: 정렬 방향이 명시되지 않은 경우 기본값(ascending)을 사용합니다.
3. `OrderSpecifier` 객체 생성: 요청에 맞는 `OrderSpecifier` 객체를 생성합니다.
4. 쿼리에 정렬 적용: 정렬 정보에 맞춰 orderBy 쿼리를 적용합니다.


#### `CustomOrder` 클래스
정렬 필드와 방향을 명시적으로 담기 위해 CustomOrder 클래스를 정의했습니다.

```java
protected record CustomOrder(String entityField, Order direction) {
        public static CustomOrder of(String entityField, Order direction) {
            return new CustomOrder(entityField, direction);
        }
    }
```

#### 정렬 문자열 파싱
정렬 문자열을 `CustomOrder` 객체로 변환하는 메서드를 구현했습니다.

```java
 protected static CustomOrder separateOrder(String sort) {

        String[] sortArray = sort.split(",");
        String entityField = sortArray[0];
        Order direction = sortArray.length > 1 ? Order.valueOf(sortArray[1].toUpperCase()) : Order.ASC;
        return CustomOrder.of(entityField, direction);
    }
```

#### `OrderSpecifier` 객체 생성
파싱한 정보를 바탕으로 `OrderSpecifier` 객체를 생성하는 메서드를 작성했습니다.

```java
protected static <ITEM> OrderSpecifier getOrder(String sort, PathBuilder<ITEM> entity) {

        CustomOrder separateOrder = separateOrder(sort);
        return new OrderSpecifier(separateOrder.direction, entity.get(separateOrder.entityField));
    }

```

#### 정렬 적용
여러 개의 정렬 조건을 처리하고 쿼리에 적용하는 메서드를 구현했습니다.

```java
 public static <ITEM> JPQLQuery<ITEM> applySorting(String[] sort, JPQLQuery<ITEM> query, PathBuilder<ITEM> entity) {
        List<OrderSpecifier> orderSpecifiers = new ArrayList<>();

        if (sort != null && sort.length > 0) {
            for (String sortText : sort) {
                orderSpecifiers.add(getOrder(sortText, entity));
            }
            return query.orderBy(orderSpecifiers.toArray(new OrderSpecifier[orderSpecifiers.size()]));
        }
        return query;
    }
```

정렬 정보가 없을 경우 기존 쿼리를 반환하고, 정렬 정보가 있을 경우 쿼리에 정렬을 적용하여 반환합니다.

#### 적용 예
이제 위에서 정의한 메서드를 QueryDSL 쿼리에서 사용할 수 있습니다. 

```java
JPAQuery<Measure> query = queryFactory.select(.. 생략 ..)
                                        .from(.. 생략 ..);

applySorting(params.getSort(), query, new PathBuilder<>(Measure.class, "measure")).fetch();
```

### +) 추가 문제와 해결
`@ModelAttribute`를 사용하여 쿼리 파라미터를 바인딩할 때 문제가 발생했습니다. 예를 들어, `sort=timestamp,desc`라는 요청이 예상과 다르게 동작했습니다.

- 예상

```java
new String[]{"timestamp,desc"}
```

- 실제

```java
new String[]{"timestamp", "desc"}
```

정렬 조건이 하나일 때 `timestamp와 desc`가 분리되어 전달되면서 정렬이 제대로 적용되지 않았습니다. 이 경우, 다시 결합해주는 로직을 추가했습니다.

```java
private static String[] mergeSortFields(String[] sort) {
    if (sort.length == 2) {
        String second = sort[1].toUpperCase();
        if (second.equals(Order.DESC.name()) || second.equals(Order.ASC.name())) {
            sort = new String[]{String.join(",", sort)};
        }
    }
    return sort;
}
```

이렇게 개선된 로직은 정렬 조건이 하나일 때에도 올바르게 동작하도록 합니다.

### 결론
이 글에서는 QueryDSL을 사용하여 동적으로 정렬을 적용하는 방법을 설명했습니다. 이를 통해 페이징 없이도 정렬만 필요한 경우에 유연하게 대처할 수 있습니다. 
