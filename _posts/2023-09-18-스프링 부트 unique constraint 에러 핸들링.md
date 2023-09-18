---
layout: post
categories: 공부, 스프링
---

## Unique Constraint 에러 핸들링 어떻게 하지? 
멀티 컬럼을 unique 제약으로 거는 경우가 있습니다. 지금 진행 하고 있는 사이드 프로젝트를 예로 들어보겠습니다. 

사용자(User)와 테스크(Task)가 있습니다. 사용자는 테스크를 생성할 수 있습니다. 여기서 테스크는 달력에서 하루라고 볼 수 있습니다. 
즉, 오늘 테스크 생성, 내일 테스크 생성 이렇게 말이죠. 여기서 하나의 조건이 있습니다. 바로 같은 날 두개의 테스크는 생성이 불가합니다. 
왜냐하면, 테스크를 나누지 않고 테스크 안에서 투두라는 것으로 분리하기로 했기 때문이죠. 좀 더 쉽게 이해하기 위해 아까 위에서 테스크는 달력에서의 하루하고 했습니다. 하나의 사용자는 9월 18일을 두 번 만들 수 없다는 뜻입니다. 지금의 정책은 그렇습니다. 

이럴 때 데이터 무결성을 위해 사용자와 테스크 날짜를 unique 제약으로 묶어줍니다.

```java
@Table(uniqueConstraints = @UniqueConstraint(columnNames = {"user_id", "task_date"}))
public class Task {
    // ... 생략
}
```

그러면, 서로 다른 사용자가 같은 날짜에 테스크를 생성하는 것은 정상. 
같은 사용자가 같은 날짜에 테스크를 여러번 생성하는 것은 에러가 납니다. 

![JdbcSQLIntegrityConstraintViolationException](/assets/img/JdbcSQLIntegrityConstraintViolationException.png)

다음과 같은 에러가 발생하게 되죠. 그렇다면, 서버에서는 이 제약 조건 에러를 그대로 두는게 맞는가? 핸들링하는게 나은가요? 
일단, 저는 response status code가 500으로 떨어지는 것이 맞지 않는다고 생각합니다. 서버 에러라기 보단 이 경우는 사용자에게 Bad Request를 보냈다고 하는게 맞아 보입니다. 

그래서, 커스텀 예외를 만들고 save시 `DataIntegrityViolationException`가 발생하면 커스텀 예외를 발생하고 `ControllerAdvice`에서 처리를 해주려고 합니다. 

```java
// .. task save 처리 로직 중 나머지 생략

        try {
            Tasks savedTask = tasksRepository.save(task);
            return new TaskAddResponse(savedTask);
        } catch (DataIntegrityViolationException e) {
            throw new UserAndTaskDateUniqueConstraintViolationException(user.getName(), taskAddRequest.getTaskDate());
        }
```
다음과 같이 `DataIntegrityViolationException`을 잡아서 길지만 커스텀한 예외를 던져줍니다. 

```java

    /**
     * 사용자의 이미 존재하는 테스크 날짜에 요청이 온 경우
     */
    @ResponseStatus(BAD_REQUEST)
    @ExceptionHandler(UserAndTaskDateUniqueConstraintViolationException.class)
    public ResponseEntity<Map<String, String>> handleUserAndTaskDateUniqueConstraintViolationException(UserAndTaskDateUniqueConstraintViolationException exception) {
        log.error("UserAndTaskDateUniqueConstraintViolationException", exception);

        Map<String, String> errorResponseBody = getErrorResponseBody(exception);
        return new ResponseEntity<>(errorResponseBody, BAD_REQUEST);
    }

```

그러면 에러를 핸들링한느 `ErrorControllerAdvice`에서 이 예외를 가지고 400과 에러 메세지를 body에 담아 응답합니다. 

![제약 조건 테스트 결과](/assets/img/제약조건테스트%20결과.png)
테스트가 잘 통과되네요. 

```json

MockHttpServletResponse:
           Status = 400
    Error message = null
          Headers = [Vary:"Origin", "Access-Control-Request-Method", "Access-Control-Request-Headers", Content-Type:"application/json;charset=UTF-8"]
     Content type = application/json;charset=UTF-8
             Body = {"error":"Already Exists test user's task for this date( 2023-08-22 )."}
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```

응답도 원하는 대로 잘 나옵니다. 

이렇게 에러를 처리하는 방법도 있지만, DB에 부하를 주는 것이 맞나? 라는 의문이 있습니다. 그렇다고 DB에 userId와 TaskDate가 존재하는지 매번 조회하는 것 또한 비용이 발생하는 부분이라 어떤 것이 맞느냐? 상황에 따라 다를 거 같습니다. 

지금은, 클라이언트에서 테스크를 생성하는 버튼 자체를 제공하지 않는다면 누군가 의도적으로 API를 쏘지 않는 이상 해당 이슈가 발생할 일이 매우 희박합니다. 그러나 매 생성마다 DB를 조회하면 해당 이슈가 발생할 일이 매우 희박하나 조회는 매번 해야하는 상황이 됩니다. 따라서, 저는 예외를 핸들링하는 방법으로 해결했습니다. 
