---
layout: post
categories: 공부 정리
---
[GitHub Docs - About pull request merges](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/incorporating-changes-from-a-pull-request/about-pull-request-merges)

`pull request`를 `merge`는 브랜치의 모든 커밋을 유지하거나 모든 커밋을 단일 커밋으로 스쿼시하거나 브랜치의 개별 커밋을 브랜치로 리베이스 할 수 있다. 

## Merge pull request 
기본 `Merge pull request`은 branch의 모든 커밋이 `merge commit`의 base branch에 추가된다.    
=> 작업 커밋 + merge 커밋 => base branch에 추가 

![Merge commit: D + E added to Main via F](https://docs.github.com/assets/cb-5407/images/help/pull_requests/standard-merge-commit-diagram.png)

풀 리퀘스트는 [--no-ff](https://git-scm.com/docs/git-merge#_fast_forward_merge) 옵션으로 `merge`된다. 

## Squash and merge

`Squash and merge` 옵션은 pull request commit이 단일 커밋으로 묶여서 base branch에 merge된다. 기여자의 개별 커밋 대신 커밋이 하나의 커밋으로 결합되고 기본 브랜치로 병합된다. [fast-forward](https://git-scm.com/docs/git-merge#_fast_forward_merge) 옵션을 사용해서 merge된다.

![Squash and merge: D + E into F in Main](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/incorporating-changes-from-a-pull-request/about-pull-request-merges)

* `squash and merge`는 저장소에 쓰기 권한이 있어야 하고 레파지토리에서 `squash and merge`을 허용해야 한다.

### Merge message for Squash merge 

|커밋 수| 요약 | 설명 |
| :---: | :---: | :---: |
|하나의 커밋|커밋 메시지의 제목과 풀 리퀘스트 번호| 단일 커밋에 대한 커밋 메시지의 본문|
| 하나 이상의 커밋 | 풀 리퀘스트 제목과 풀 리퀘스트 번호 | 모든 스쿼시된 커밋에 대한 커밋 메세지 목록(날짜 순서)|  

* 커밋이 한개일 때는 해당 커밋 메시지의 제목과 풀 리퀘스트 번호가 merge message 요약이 된다. 
* 커밋이 여러개일 때는 풀 리퀘스트 제목과 번호가 merge message 요약이 된다. 

* 풀 리퀘스트가 병합된 후 해당 풀 리퀘스트에서 `HEAD` 브랜치에 대한  작업을 계속할 계획이면 스쿼시 병합하지 않는 것이 좋다. 
  * 수행한 모든 변경 사항을 포함하는 커밋을 기본 브랜치에 생성하는데 이 커밋이 헤드 브랜치가 아닌 기본 브랜치에만 있기 때문에 두 브랜치의 공통 조상이 변경되지 않는다. 따라서 병합 충돌이 발생할 가능성이 높아진다

## Rebase and merge
`Rebase and merge`는 branch의 모든 커밋이 merge commit 없이 개별적으로 base branch에 추가된다.    
`fast-forward`merge와 유사하다.    
리베이스는 새 커밋으로 기본 브랜치의 커밋 히스토리를 다시 작성한다는 점이 다르다. 
[git rebase 참조](https://git-scm.com/book/en/v2/Git-Branching-Rebasing)
