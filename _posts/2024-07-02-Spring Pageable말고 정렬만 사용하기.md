---
layout: post
categories: 공부 스프링
---

조회 요청시, `Pageable`을 사용하여 페이징과 정렬을 할 수 있다. 종종 정렬만 사용하고 싶은 경우가 있다. 예로 현재 회사에서는 센서 데이터를 보여줄 때, 시간 정렬은 필요하지만 페이징은 필요하지 않다. 이런 경우 `Pageable`에 정렬 정보만 줘도 되지만 QueryDsl에서 정렬을 동적으로 사용하기 위해, `Pageable`을 사용하지 않고 정렬 기능만 따로 구현을 해주었다. 

요청은 `Pageable`을 사용하고 있는 다른 요청들과 공통화하기 위해 `sort=id,desc&createdAt`와 같은 형식을 사용한다. 

```java
// 요청 파라미터
private String[] sort;

```

이제 해당 파라미터를 queryDsl에서 동적으로 정렬을 주도록 처리해야하는데, queryDsl에서 `orderBy`에 `OrderSpecifier`를 넘겨주면 된다. 

먼저 `sort=id,desc&createdAt`와 같은 요청이 왔을 때, 해야할 기능을 정리한다. 

1. 해당 값이 오지 않았을 때 처리 
2. 정렬 entity field와 정렬 방향(asc, desc) 분리 
    - 정렬 방향이 오지 않았을 경우 기본 값(asc) 처리
3. 요청 값에 맞춰 `OrderSpecifier` 객체 생성
4. 정렬하기 원하는 entity에 `orderBy` 쿼리 적용  

요청 값을 분리하여 정렬하고자 하는 객체 필드와 정렬 방향을 리턴하는 메소드를 만들자. 
해당 정보를 명시적으로 담고 사용하기 위해 객체를 선언해주었다. 

```java
protected record CustomOrder(String entityField, Order direction) {
        public static CustomOrder of(String entityField, Order direction) {
            return new CustomOrder(entityField, direction);
        }
    }
```

그 다음, `id,desc`와 같은 문자열이 넘어오면 분리해서 `CustomOrder`를 생성해주는 메소드를 만들었다. 

```java
 protected static CustomOrder separateOrder(String sort) {

        String[] sortArray = sort.split(",");
        String entityField = sortArray[0];
        Order direction = sortArray.length > 1 ? Order.valueOf(sortArray[1].toUpperCase()) : Order.ASC;
        return CustomOrder.of(entityField, direction);
    }
```

이제 파싱한 정보를 가지고 `OrderSpecifier` 객체로 만들어주는 메서드가 필요하다. 

```java
protected static <ITEM> OrderSpecifier getOrder(String sort, PathBuilder<ITEM> entity) {

        CustomOrder separateOrder = separateOrder(sort);
        return new OrderSpecifier(separateOrder.direction, entity.get(separateOrder.entityField));
    }

```

정렬 정보를 가진 문자열을 가지고 정렬 쿼리 인스턴스인 `OrderSpecifier`를 만드는 메서드를 모두 구현했다. 
이제, 여러개의 sort가 올 경우 이를 가지고 정렬을 만들고 실제 쿼리에 적용하는 부분만 만들면 된다. 

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

정렬 정보가 없다면 기존 쿼리를 반환하고, 정렬 정보가 있다면 쿼리에 정렬을 적용하여 리턴하는 함수이다. 

해당 메소드를 queryDsl 함수에서 다음과 같이 사용하면 된다. 

```java
 JPAQuery<Measure> query = queryFactory.select(.. 생략 ..)
                                                    .from()
 ;

applySorting(params.getSort(), query, new PathBuilder(Measure.class, "measure")).fetch()
```

고민만 하던 것을 실제로 구현하고 실무에 사용해봤다. 

