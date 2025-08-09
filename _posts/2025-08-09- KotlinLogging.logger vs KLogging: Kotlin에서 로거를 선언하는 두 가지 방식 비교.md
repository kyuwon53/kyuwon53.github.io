---
layout: post
categories: 글
tags: kotlin, logger
---


## Kotlin-Logging (MicroUtils / oshai)

* **개요**
  `kotlin-logging`은 `mu.KotlinLogging` 네임스페이스로 사용되며, SLF4J 기반의 *Kotlin 친화적인 래퍼(wrapper)* 라이브러리입니다. (예: `import mu.KotlinLogging`)
  로거를 선언할 때 클래스 이름을 직접 지정하지 않아도 되고, 메시지를 람다 형태로 전달하여 **레이지(lazy) 메시지 평가**를 지원합니다.([GitHub][1], [ProAndroidDev][2])

* **특징**

  * `logger {}`를 통해 간편하게 로거 생성
  * 성능 최적화를 위해 로그 레벨이 활성화된 경우에만 메시지 평가
  * SLF4J 기반이므로 기존 로깅 백엔드(Logback 등)와 호환
  * 멀티플랫폼(MPP) 지원 가능성 있음([GitHub][1])

---

## Klogging

* **개요**
  `Klogging`은 **pure Kotlin** 기반의 로깅 라이브러리로, 구조화된(structured) 로깅, 코루틴 스코프 기반의 컨텍스트 전달, 고해상도 타임스탬프 등을 중심으로 설계되었습니다([GitHub][3], [klogging.io][4]).

* **특징**

  * 메시지를 단순한 문자열이 아닌 **구조화된 이벤트**(structured log event)로 생성
  * Kotlin 코루틴 컨텍스트에 포함된 정보를 자동으로 로그에 포함
  * 최소 마이크로초 단위, 가능하면 나노초까지 지원하는 **고해상도 타임스탬프**
  * 비동기 로깅 설계 (코루틴 채널 등 이용)([GitHub][3], [klogging.io][4])

---

## 요약 정리

| 항목              | kotlin-logging (`mu`)      | Klogging                        |
| --------------- | -------------------------- | ------------------------------- |
| 네임스페이스 / Import | `mu.KotlinLogging`         | (고유 패키지, `klogging` 등)          |
| 기반              | SLF4J 래퍼                   | Pure Kotlin, 자체 구현              |
| 강점              | 간단하고 익숙한 API, 기존 SLF4J와 호환 | 구조화 로깅, 코루틴 컨텍스트 포함, 고해상도 타임스탬프 |
| 용도              | 일반적인 로깅, Kotlin 식 API 선호 시 | 분산/비동기 환경, 구조화된 로깅, 컨텍스트 전달 시   |

---

### 정리하면…

* `mu.KotlinLogging`은 **전통적인 SLF4J 기반**의 Kotlin용 래퍼로서, **간편하게 사용하는 로깅 도구**입니다.
* 반면에 **Klogging**은 **코루틴과 구조화 로깅에 주안점을 둔** 라이브러리로, 보다 **현대적인 구조화된 로그 처리**를 필요로 할 때 선택하면 좋습니다.


[1]: https://github.com/oshai/kotlin-logging "oshai/kotlin-logging: Lightweight Multiplatform logging framework for ..."
[2]: https://proandroiddev.com/kotlin-tips-and-tricks-you-may-not-know-1-kotlin-logging-91b7675b6276 "Kotlin Logger: Idiomatic Logging with Structured Logging and MDC ..."
[3]: https://github.com/klogging/klogging "klogging/klogging: Kotlin logging library with structured ... - GitHub"
[4]: https://klogging.io/docs/about-klogging/ "About Klogging"
