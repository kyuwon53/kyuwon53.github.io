---
layout: post
categories: 스프링
---
![alt](.\img\SpringBootApplication.png)

- @ComponentScan에만 적용된다. 
- @Entity 스캔 또는 `Spring Data Repository` 스캔에는 영향을 미치지 않는다. 
  - @EntityScan 및 @Enable..을 추가해야 한다
- 검색할 패키지를 지정하면 지정한 각 클래스의 패키지가 검색된다. 


- ComponentScan 설정을 하지 않았다면 `@SpringBootApplication`이 정의된 곳이 **base package** 가 된다. 

<br>

## @ComponentScan ? 
- `@Component`, `@Service`, `@Repository`, `@Controller` , `@Configuration` 어노테이션이 부연된 Class들을 **자동으로 스캔하여 Bean으로 등록** 해준다.
- 검색할 패키지를 정의하기 위해 `basePackageClasses` 또는 `basePackages`를 지정할수 있다.
- 특정 패키지가 정의되지 않은 경우 이 주석을 선언하는 클래스의 패키지에서 찾는다. 
- `@ComponentScan`을 사용할 때 기본 어노테이션 (@Autowired 및 friends)이 가정된다. 

#### 검색할 패키지를 정의 : `basePackageClasses` 또는 `basePackages`

```java
@ComponentScan(basePackages = "com.codesoom")
public class AppConfig {} 
```
- `basePackages` 패키지 경로를 직접 적어 스캔할 위치를 지정할 수 있다. 

```java
@ComponentScan(basePackageClasses = App.class)
public class AppConfig {} 
```
- `basePackageClasses`: Class가 위치한 곳에서부터 모든 어노테이션이 부여된 Class를 빈으로 등록 
- **TypeSafe**하기 때문에 `basePackages` 보다 사용을 추천한다. 

<hr>

## 참고 
- [스프링 component-scan 개념 및 동작 과정](https://velog.io/@hyun-jii/%EC%8A%A4%ED%94%84%EB%A7%81-component-scan-%EA%B0%9C%EB%85%90-%EB%B0%8F-%EB%8F%99%EC%9E%91-%EA%B3%BC%EC%A0%95)
- [Springboot 에서 @ComponentScan 설정 및 사용법](https://oingdaddy.tistory.com/254)
- [[스프링 핵심기술] - @Component와 @ComponentScan](https://jjingho.tistory.com/9)
- [Spring- @ComponentScan 어노테이션이란?](https://galid1.tistory.com/510)

<hr>

내용 더 정리하자 
