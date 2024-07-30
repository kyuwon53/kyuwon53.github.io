---
layout: post
categories: 공부 sql 
---

### SQL로 데이터 피벗 테이블 생성하기
이번 포스팅에서는 PostgreSQL을 사용하여 시리얼 번호를 기준으로 각 레벨별 타임스탬프를 컬럼으로 변환하고, 이를 하나의 행으로 만드는 방법을 살펴보겠습니다. PostgreSQL의 내장 함수를 사용할 수도 있지만, 범용적인 SQL 쿼리를 이용하여 이 작업을 수행해보겠습니다.

#### 데이터 예시
우선, 다음과 같은 형태의 데이터를 가진 테이블이 있다고 가정하겠습니다.

![예시1](/assets/img/여러row예시.png)

이 테이블에서 시리얼 번호별로 각 레벨별 타임스탬프를 한 줄로 펴서 아래와 같이 출력하고 싶습니다.

![출력 예시2](/assets/img/한줄쿼리출력.png)

#### SQL 쿼리
범용적인 SQL 쿼리를 사용하여 피벗 테이블을 생성하려면, 조건부 집계를 활용할 수 있습니다. 아래는 이 작업을 수행하는 SQL 쿼리 예시입니다.

```sql
select serial_no
	, max(case when "level" = 1 then "timestamp" else null end) as level_1
	, max(case when "level" = 2 then "timestamp" else null end) as level_2
	, max(case when "level" = 3 then "timestamp" else null end) as level_3
	, max(case when "level" = 4 then "timestamp" else null end) as level_4
from severity 
group by serial_no 
;
```
#### 쿼리 설명

- `SELECT` 절: 각 시리얼 번호에 대해 레벨 값을 조건부로 집계하여 새로운 컬럼으로 만듭니다.

- `MAX(CASE WHEN level = 1 THEN timestamp ELSE NULL END) AS level_1`: level이 1인 경우 `timestam`p 값을 level_1 컬럼으로 만듭니다.
- `MAX(CASE WHEN level = 2 THEN timestamp ELSE NULL END) AS level_2`: level이 2인 경우 `timestamp` 값을 level_2 컬럼으로 만듭니다.
- `MAX(CASE WHEN level = 3 THEN timestamp ELSE NULL END) AS level_3`: level이 3인 경우 `timestamp` 값을 level_3 컬럼으로 만듭니다.
- `MAX(CASE WHEN level = 4 THEN timestamp ELSE NULL END) AS level_4`: level이 4인 경우 `timestamp` 값을 level_4 컬럼으로 만듭니다.
- `FROM` 절: 원본 테이블(`severity`)을 지정합니다.

- `GROUP BY` 절: 시리얼 번호별로 그룹화하여 피벗 테이블을 생성합니다.
- `ORDER BY` 절: 결과를 시리얼 번호 순으로 정렬합니다.

이 쿼리는 각 시리얼 번호에 대해 레벨별 타임스탬프를 하나의 행으로 피벗하여 출력합니다. MAX 함수는 실제로는 하나의 값만 존재하기 때문에, 주어진 그룹에서 단 하나의 값을 선택하는 역할을 합니다.

### + QueryDSL로 데이터 피벗 테이블 생성하기
더 나아가서 이를 하나의 행으로 만드는 방법을 queryDSL을 사용하여 살펴보겠습니다. JPA로도 쿼리를 작성할 수 있지만, 동적 쿼리를 만들기 위해 queryDSL을 사용해보겠습니다.

queryDSL의 CaseBuilder를 사용하여 Case when 문법을 구현할 수 있습니다. 여기서 max 함수도 같이 사용하므로 주의가 필요합니다.

```java
queryFactory.selectDistinct(Projections.fields(SeverityQueryData.class,
                        severity.id.serialNo.as("serialNo"),
                        new CaseBuilder()
                                .when(severity.id.level.eq(1))
                                .then(severity.timestamp)
                                .otherwise(nullExpression()).max().as("good"),
                        new CaseBuilder()
                                .when(severity.id.level.eq(2))
                                .then(severity.timestamp)
                                .otherwise(nullExpression()).max().as("satisfactory"),
                        new CaseBuilder()
                                .when(severity.id.level.eq(3))
                                .then(severity.timestamp)
                                .otherwise(nullExpression()).max().as("unsatisfactory"),
                        new CaseBuilder()
                                .when(severity.id.level.eq(4))
                                .then(severity.timestamp)
                                .otherwise(nullExpression()).max().as("unacceptable")
                ))
                .from(severity)
                .where(inSerialNo(serialNo))
                .groupBy(severity.id.serialNo)
                .fetch();
```

#### 코드 설명
- `CaseBuilder` 사용: `new CaseBuilder()`를 사용하여 조건부로 값을 선택합니다.
- `when(severity.id.level.eq(1)).then(severity.timestamp)`: `level`이 1인 경우 `timestamp` 값을 반환합니다.
- `otherwise(nullExpression()).max()`: 조건을 만족하지 않으면 null 값을 반환하고, 이를 그룹화된 결과에서 max 함수로 처리합니다.
- `Projections.fields`: 결과를 `SeverityQueryData` 클래스의 필드에 매핑합니다.
- `severity.id.serialNo.as("serialNo")`: 시리얼 번호를 매핑합니다.
- `new CaseBuilder() ... .max().as("level_1")`: 각 레벨별 타임스탬프를 매핑합니다.
- `GroupBy` 및 `Fetch`: 시리얼 번호별로 그룹화하고 결과를 조회합니다.
   
이 쿼리는 각 시리얼 번호에 대해 레벨별 타임스탬프를 하나의 행으로 피벗하여 출력합니다. `max` 함수는 실제로는 하나의 값만 존재하기 때문에, 주어진 그룹에서 단 하나의 값을 선택하는 역할을 합니다.

### 코드 개선
레벨 조건과 별칭을 Enum 클래스로 묶어서 변경에 대응하면 유지보수가 더 용이할 것 같습니다. 다음은 그 예시입니다:

```java
// 로직만 참고 
queryFactory.selectDistinct(Projections.fields(SeverityQueryData.class,
                        severity.id.serialNo.as("serialNo"),
                        new CaseBuilder()
                                .when(severity.id.level.eq(GOOD.getKey()))
                                .then(severity.timestamp)
                                .otherwise(nullExpression()).max().as(GOOD.getTitle()),
                        new CaseBuilder()
                                .when(severity.id.level.eq(SATISFACTORY.getKey()))
                                .then(severity.timestamp)
                                .otherwise(nullExpression()).max().as(SATISFACTORY.getTitle()),
                        new CaseBuilder()
                                .when(severity.id.level.eq(UNSATISFACTORY.getKey()))
                                .then(severity.timestamp)
                                .otherwise(nullExpression()).max().as(UNSATISFACTORY.getTitle()),
                        new CaseBuilder()
                                .when(severity.id.level.eq(UNACCEPTABLE.getKey()))
                                .then(severity.timestamp)
                                .otherwise(nullExpression()).max().as(UNACCEPTABLE.getTitle())
                ))
                .from(severity)
                .where(inSerialNo(serialNo))
                .groupBy(severity.id.serialNo)
                .fetch();

// 별도 Enum 클래스

public enum SeverityLevel {
    GOOD(1, "good"),
    SATISFACTORY(2, "satisfactory"),
    UNSATISFACTORY(3, "unsatisfactory"),
    UNACCEPTABLE(4, "unacceptable");
    private final int key;
    private final String title;
}
```
이렇게 하면 코드가 더 깔끔해지고, 레벨 조건이 추가되거나 변경될 때 유연하게 대처할 수 있을 것 같습니다.

`alias`까지 `enum`으로 불러오는 것이 변경에 더 나은 대응이 될지는 확실하지 않습니다. DTO의 필드값이 변경되면, DTO가 바뀔 때 해당 부분도 함께 수정해야 하므로, 하드코딩해도 마찬가지입니다. 따라서, 레벨을 판단하는 부분만 `enum`으로 사용하는 것이 충분할 것 같습니다.

### 마무리
이번 포스팅에서는 PostgreSQL을 사용하여 시리얼 번호를 기준으로 각 레벨별 타임스탬프를 컬럼으로 변환하는 방법을 살펴보았습니다. 이를 위해 범용적인 SQL 쿼리를 작성하고, queryDSL을 사용하여 동적 쿼리를 작성하는 방법도 다뤘습니다.

SQL 쿼리 예시에서는 MAX와 CASE WHEN 문을 활용하여 시리얼 번호별로 레벨에 따른 타임스탬프를 한 행으로 변환하는 방법을 설명했습니다. 이 방식은 간단하고 직관적인 피벗 테이블을 생성하는 데 유용합니다.

QueryDSL 쿼리에서는 CaseBuilder를 사용하여 동적 쿼리를 생성하고, 레벨별 타임스탬프를 처리하는 방법을 소개했습니다. 여기서 레벨을 enum으로 정의하여 조건을 관리하면 코드가 보다 구조화되고, 변경 시 관리가 용이할 수 있습니다. 그러나, alias까지 enum으로 관리하는 것이 항상 더 나은 대응이 될지는 확실하지 않습니다. DTO의 필드가 변경되면 해당 부분도 함께 수정해야 하므로, 레벨을 판단하는 부분만 enum으로 사용하는 것이 실용적일 수 있습니다.

이와 같은 접근 방법은 데이터 분석 및 보고서 생성에 매우 유용하며, 다양한 레벨의 정보를 효율적으로 집계하고 정리하는 데 도움이 될 것입니다. 이를 통해 데이터의 가독성과 분석 효율성을 높일 수 있습니다.
