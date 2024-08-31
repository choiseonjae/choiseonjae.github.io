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
코루틴 빌더는 그 말 그대로 코루틴을 생성한다. 그래서 로그를 보면 #1, #2로 코루틴에 번호가 매겨지는 걸 볼 수있다. ![](https://www.notion.so/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fdf7fe4f8-566c-497d-8c37-c5b69bc3f7f5%2F112a77d0-37bb-444b-9dba-6294a209be0b%2FUntitled.png?table=block&id=36f2cd80-ea72-491e-89d3-6a5eaf81647e&spaceId=df7fe4f8-566c-497d-8c37-c5b69bc3f7f5&width=2000&userId=452a463d-5ff9-4cc6-9f05-886a6f46ae8a&cache=v2)
{% capture content %}
우리가 생각하는 동시성 혹은 병렬성은 새로운 코루틴에서 발생하는 것이다.
{% endcapture %}

{% include notice--info title="코루틴 빌더의 핵심" content=content %}


즉, 같은 코루틴에서 중단해봤다 pool thread가 돌아가는 거고, 자식 코루틴을 생성해야 (launch, async) thread가 중단되었을 때 자식 코루틴이 수행되는 것이다.
(물론, Dispatchers를 설정해준다면 새로운 thread가 할당돼 중단되지 않아도 동작하겠지만)


<!--stackedit_data:
eyJoaXN0b3J5IjpbNTIxODY1NTFdfQ==
-->