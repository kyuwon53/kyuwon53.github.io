---
layout: post
categories: 회고 코드숨
---
## Facts (사실, 객관)
- 스프링 4주차 - 도메인 시작
- 계층으로 분리하여 개발 (도메인, 레포지토리, 서비스, 컨트롤러)
- 테스트코드 먼저 작성하며 개발 


## Findings (배운점)

### javadoc 
- 단순하고 확실하게 적어주자. 
- 그냥 주석이 아니라 빌드를 통해 하나의 정적 사이트로 만드는 기능을 갖고 있다. 
- 코드를 문서화하는 것이기에 개발과 동시에 쉬운 문서화가 가능

### [@ComponentScan ](https://github.com/Kyuwon53/Kyuwon53.github.io/blob/main/Spring/%5BSpring%5D%40ComponentScan.md)
- `ComponentScan` 설정을 하지 않았다면 `@SpringBootApplication`이 정의된 곳이 **base package**가 된다. 
- 즉, 특정 패키지가 정의되지 않은 경우 `@SpringBootApplication`을 선언하는 클래스의 패키지에서 스캔하여 **Bean**을 등록한다. 

### Entity
- 식별자를 가진 객체 
- 객체의 상태가 변하더라도 식별자를 통해 그 객체를 식별할 수 있다. 
- JPA를 사용하는 경우 DB테이블 구조가 된다. 

### Repository
- 데이터를 관리하는 클래스 
- `@Repository` 사용
- 인터페이스 생성 후 
  ```java
  public interface ProductRepository extends CrudRepository<Product, Long> {
  }
  ```
  - 이렇게 사용하면 어노테이션이 필요없다. 
  - 자동으로 CRUD 메서드가 자동으로 생성된다.

## Feelings (느낌, 주관)
- 테스트 작성 시 어느순간 controller와 service의 테스트 코드가 같다는 걸 느꼈다. 각각의 목적이 무엇인지 어떤 것을 테스트해야 하는지 고민해  봐야겠다. 
- 파도 파도 계속 나오는 공부 해야 할 것들에 대해 조급함이 있는 것 같다. 좋은 문화를 갖춘 곳으로 이직을 하고 싶다는 갈망이 나날이 커져서 그런 것 같다. 지금 현재 하고 있는 것들부터 차근차근 제대로 하고 넘어가야 하는데 많은 것을 담으려다 보니 어느 하나 제대로 알고 넘어가지 못하는 거 같아서 반성 중이다. 
- 오류 로그를 봐도 봐도 새로운 오류가 나오니 짜릿하다.
- 기록을 남기는 습관을 들여야 하는데 글 쓰는 게 세상에서 제일 어렵다. 
