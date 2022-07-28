---
layout: post
categories: 공부 스프링
---

실전! 스프링 부트와 JPA활용 강의를 듣다가 
`Service`로직을 테스트하는 중 발생할 수 있는 에러들을 정리해보았다. 

애너테이션에 대한 이야기이다. 

먼저 테스트 코드를 작성한 뒤 테스트를 돌렸더니 다음과 같은 오류가 발생했다. 

![memberService null](/assets/img/%ED%85%8C%EC%8A%A4%ED%8A%B8%20%EC%BD%94%EB%93%9C%20%EC%98%A4%EB%A5%98.png)

오류 내용을 보면 `memberService`가 null이라고 나온다. 분명 `Autowired`로 빈 등록까지 다했는데 null 이라고 뜬다. 여기서 `@SpringBootTest`붙여주면 해당 오류는 사라진다. 

다시 테스트를 돌려보면 다음과 같은 에러가 발생한다. 

![테스트 실패](/assets/img/%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%BD%94%EB%93%9C%20%EC%8B%A4%ED%8C%A8.png)

```java
    @Test
    public void 회원가입() throws Exception {
        //given
        Member member = new Member();
        member.setName("kim");

        //when
        Long saveId = memberService.join(member);

        //then
        assertThat(memberRepository.findOne(saveId)).isEqualTo(member);
    }

```

테스트 코드에서는 회원을 가입하고 그 해당 회원을 조회하기 때문에 성공해야 하는데 

```console
Expected :jpabook.jpashop.domain.Member@121f9c52
Actual   :jpabook.jpashop.domain.Member@44a0e68f
```
다음과 같이 객체가 다르다. 이 경우 `@Transactional`을 달아주면 테스트는 통과하게 된다. 

자 그러면 여기서 `@SpringBootTest` 과 `@Transactional`은 무슨 기능을 하고 있는 것일까?    

<br/>

***

<br/>

## @SpringBootTest

![@SpringBootTest](/assets/img/SpringBootTestAnnotation.png)

간단하게 말하면 스프링 부트를 띄우고 테스트를 한다. 이게 없으면 `@Autowired`가 다 실패하게 된다. 

스프링 부트 어플리케이션 테스트에 필요한 거의 모든 의존성을 제공하고 있다. 

통합 테스트를 제공하는 기본적인 스프링 부트 테스트 애너테이션이다. 

자동으로 **H2 데이터베이스**를 실행해준다. 

여러 기능들이 있는데 

* 전체 빈들 중 특정 빈을 선택해서 생성
* 특정 빈을 Mock으로 대체
* 특정 Configuration을 선택하여 설정
* 테스트 웹 환경을 자동으로 설정 
* 테스트에 사용할 프로퍼티 파일 선택, 특정 속성 추가 

이와 같은 기능들을 사용하기 위해서는 `@RunWith(SpringRunner.class)`를 함께 사용해야한다. 

## @RunWith(SpringRunner.class)

테스트를 진행할때 JUnit에 내장된 실행자 외에 다른 실행자를 실행시킨다. (여기서는 SpringRunner)

스프링 부트 테스트와 JUnit 사이에 연결자 역할을 한다. 

하지만 이 기능은 `JUnit4`에서 사용되고 `JUnit5`에서는 안쓴다. 

## @ExtendWith(SpringExtension.class)

`JUnit5`에서는 `@RunWith(SpringRunner.class)` 말고 `@ExtendWith(SpringExtension.class)`을 사용한다.

하지만 스프링부트가 버전이 바뀌면서 이또한 생략할 수 있게 되었다. 이미 `@SpringBootTest`가 가지고 있기 때문이다.

## @Transactional 

테스트에서 이 애노테이션을 추가하면 자동으로 롤백이 발생한다. 기본으로 롤백으로 처리해서 다른 테스트에 영향을 주지 않기 위한 기능이다. 

데이터 변경 시에는 트랜잭션이 필요하다. 

`@Transactional` 에 대해서는 따로 또 알아보도록 하자! 
