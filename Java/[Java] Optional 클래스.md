# [Java] Optional 클래스 

## java.util.Optional<T> 클래스 
- 'T'타입의 객체를 포장해주는 래퍼 클래스(Wrapper class)이다. 
- Optional 인스턴스는 모든 타입의 참조 변수를 저장할 수 있다. 

- 복잡한 조건문 없이 `널(null)` 로 발생하는 예외를 처리할 수 있다. 

<hr>

## Optional 객체 생성

### of() 
- null이 아닌 명시된 값을 가지는 Optional 객체를 반환
- null이 저장되면 `NullPointerException` 예외가 발생한다. 
  - null이 될 가능성이 있다면 `ofNullable()` 메소드를 사용하여 Optional 객체를 생성하는 것이 좋다. 

### ofNullable()
- 명시된 값이 null이면 비어있는 Optional객체를 반환한다. 

### orElse
- Optional 안의 값이 `null`이든 아니든 **항상 호출**

### ofElseGet
- Optional 안의 값이 `null`일 경우에만 호출된다.

<hr>

## Optional 객체 접근
- get() 메소드를 사용하면 Optional객체에 저장된 값에 접근할 수 있다. 
- Optional 객체에 저장된 값이 null이면, `NoSuchElementException`예외가 발생하므로 `isParent()`메소드를 사용하여 null인지 아닌지 판단하는 것이 좋다. 

```java
Optional<String> optional = Optional.ofNullable("Optional 객체");
if(optional.isPresent()){
  System.out.println(optional.get());
}
```
- 결과 `Optional 객체`

## 참조 
- [[Java] Optional이란?](https://mangkyu.tistory.com/70)
- [Optional 클래스](http://www.tcpschool.com/java/java_stream_optional)