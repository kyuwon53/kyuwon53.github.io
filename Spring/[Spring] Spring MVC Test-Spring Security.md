# [Spring] Spring MVC Test - Spring Security TEST

스프링 시큐리티를 스프링 MVC 테스트를 진행하려 했으나 계속해서 테스트 실패를 했다. 처음에는 코드에 누락된 부분들이 있나 살펴봤지만 도통 알수가 없었다. 

![](./img/authenticationError.PNG)

알고보니 테스트에서 이미 에러가 나서 안에 로직도 안타는 것이었다. 결론은 MVC Test시에 `MockMvc`에 설정을 해줘야한다. 

```java
    @Before
    public void setup() {
        mvc = MockMvcBuilders
                .webAppContextSetup(context)
                .apply(springSecurity()) // spring security 설정
                .build();
    }
```
- 스프링 시큐리티를 스프링 MVC 테스트와 통합할 때 필요한 모든 초기 세팅을 수행한다. 

저 설정 1줄 추가 했더니 거짓말같이 아주 잘된다. 

- [참고 ](https://godekdls.github.io/Spring%20Security/testing/#1921-setting-up-mockmvc-and-spring-security)
