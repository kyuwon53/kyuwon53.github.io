---
layout: post
categories: 공부, jpa ,자바
---

jpa를 사용하여 삭제할 때, 기본 제공되는 메서드에는 `delete`와 `deleteById`가 있다. 
`delete`는 삭제하려는 `Entity`를 받아서 삭제한다. 
`deleteById`는 삭제하려는 `Entity`의 `id`값을 받아서 삭제한다. 

이 둘의 차이가 뭘까 문득 궁금해졌다. 처리하는 로직이 어떻게 다를까? 구글링을 해보니 많은 블로그에서 
`deleteById`는 id에 해당하는 Entity를 찾고 없다면 Exception을 발생시키고 delete를 호출한다고 나와있었다. 
![예전 deleteById](/assets/img/regacy%20deleteById.png)


진짜 그런가? `SimpleJpaRepository` 구현 클래스를 직접 들어가보니 ???? 바껴있었다. 현재 `SimpleJpaRepository`에서 `deleteById`구현은 

![new deleteById](/assets/img/new%20deleteById.png)
exception을 던지지 않고, `ifPresent`를 사용하여 이미 가지고 있다면 `delete` 함수를 호출하도록 변경되었다.

이전에는 이미 deleteById에서 예외를 던지고 있었다면.

이제는 findById를 사용하여 예외 처리가 필요한 경우 예외 처리를 커스텀 하여 서버 개발자가 원하는 메시지를 클라이언트로 보내줄 수 있다

그럼 언제 delete와 deleteById를 써야 하는가?

- 삭제 대상이 존재하는 경우 삭제하고 없다면 아무 조치를 안 해도 된다 -> `deleteById`
- 삭제 대상이 존재라는 경우 삭제하고, 없다면 예외를 던진다 -> `findById` + `delete`

커스텀 예외 처리를 해야 하는 경우 `deleteById`를 사용해도 무관한 거 아니야? 할 수 있지만, findById를 두 번 호출하는 것이기 때문에 후자가 낫다.
