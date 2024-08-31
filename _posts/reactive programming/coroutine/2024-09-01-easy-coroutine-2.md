---
title: \[코루틴] 코루틴 빌더와 코루틴 스코프

header:
  teaser: /assets/coroutine_teaser.png

categories: coroutine
   
tags:
   - kotlin
   - coroutine
   - coroutine scope

last_modified_at: 2024-09-01 

---
코루틴 빌더와 코루틴 스코프는 비슷해보이지만, 정말 다르다.


## 코루틴 빌더와 코루틴 스코프

|  | 코루틴 빌더 | 코루틴 스코프 |
| --- | --- | --- |
| 키워드 | runBlocking, launch, async … | coroutineScope, withContext, supervisorScope … |
| 정의 | 코루틴을 생성하는 함수 | 코루틴이 실행되는 범위를 제한하는 개념 |
| 예외 | try-catch 블록으로 잡히지 않음 | try-catch 블록으로 예외 처리 가능 |

## 코루틴 빌더
코루틴 빌더는 그 말 그대로 코루틴을 생성한다. 그래서 로그를 보면
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTY0NjU2MzIzMl19
-->