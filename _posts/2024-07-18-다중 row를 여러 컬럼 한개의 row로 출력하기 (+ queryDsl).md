---
layout: post
categories: 공부 sql 
---

여러 개의 데이터가 있는데, 하나의 기준으로 여러 행을 한줄로 펴는 작업을 하고 싶습니다. 

![예시1](/assets/img/여러row예시.png)

위의 예시를 보면 시리얼 번호, 레벨, 시간 컬럼을 가진 `severity`라는 테이블이 있습니다. 이 테이블에서 시리얼 번호별 레벨별 시간을 출력하고 싶습니다. 

![출력 예시2](/assets/img/한줄쿼리출력.png)

위의 예시를 보면 시리얼번호를 기준으로 level 값들이 컬럼이 되고 데이터는 시간이 된 걸 볼 수 있습니다. 

현재, DB는 postgreSQL을 쓰고 있습니다. postgreSQL의 내장 함수의 기능을 쓸 수도 있지만, 범용적인 SQL 쿼리를 이용해서 위처럼 출력해보도록 하겠습니다. 

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

센서별 심각도를 담고 있는 테이블입니다. 데이터는 임의로 넣어두었고, 심각도 단게는 총 4단계로 각 심각도가 측정된 시간들이 `timestamp` 컬럼에 저장되어있습니다. 
시리얼 번호별로 수집할 것이기 때문에 `group by` 함수로 묶고, `max` , `case when` 함수를 사용해 해당하는 `level`의 `timestamp`를 출력합니다. 
여기서 `max`함수는 데이터를 보면 하나의 컬럼만 존재하기 때문에 `max`로서의 기능이 아니라 `group by`를 사용하기 위한 목적으로 쓰여졌다는걸 알 수 있습니다. 

### + queryDSL

더 나아가서 이 쿼리를 `queryDSL`로 짜보겠습니다. 
JPA로 쿼리를 쓸 수도 있지만 동적 쿼리를 만들기 위해 `queryDSL`을 사용해보겠습니다. 

달리 어려울 건 없고 Case When 문법을 사용하는 법만 알면 됩니다.

queryDSL의 `Case Builder`를 사용하여 Case when 문법을 사용하면 되는데, 여기서 `max` 문법도 같이 쓰기 때문에 주의해서 사용해야합니다. 

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

`new CaseBuilder()`를 사용해서 조건은 `when`에 조건을 만족하면 출력할 데이터를 `then()`에 입력하고 조건이 맞지 않을 경우 출력할 데이터를 `otherwise()`에 입력해주면 됩니다. 
group by를 사용하기 위해 Case When 절의 마지막에 `max` 연산자를 넣어주고, 별칭 이 데이터를 담을 객체에 지정된 필드명을 적어주면 됩니다. 

level 조건과 별칭은 Enum 클래스로 묶어서 변경에 대응하면 좋을거 같아요. 

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

alias까지 enum으로 불러오는 것이 변경에 더 나은 대응인지는 잘 모르겠습니다. DTO 필드값이 변경되면 DTO가 바꼈을 때 해당 부분도 바껴야하는데, 이를 하드코딩을 해도 마찬가지라 level을 판단하는 부분만 사용해도 충분할 거 같습니다. 
