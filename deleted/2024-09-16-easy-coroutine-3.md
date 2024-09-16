---
title: \[코루틴] 우리가 자주 하는 실수들

header:
  teaser: /assets/coroutine_teaser.png

categories: coroutine
   
tags:
   - kotlin
   - coroutine
   - coroutine mistake
   - coroutine pr
   - coroutine review

last_modified_at: 2024-09-16 

---

회사에서 PR 리뷰를 하다 보면 자주 하는 실수들이 있는데요.

기본적으로 코루틴은 간단하게 쓸 수 있지만, 내부는 복잡하게 동작하는 경우가 많습니다.
스터디를 진행하면서 정말 많은 테스트 코드와 문서를 참고하면서 알게된 사실들을 
저 또한 잊지 않기 위해서 정리합니다.

### runBlocking은 자신을 호출한 스레드를 블로킹한다.

실제 운영 코드에서 runBlocking을 사용하는 경우가 종종 있는데, 동작을 이해하지 못하고 사용하시는 경우가 있습니다.
(*가령, 중단 함수에서 launch, async 등을 사용하고자 하는 경우)*

runBlocking을 호출하는 순간, 호출한 스레드는 생성된 코루틴이 종료될 때까지 스레드가 블로킹되어 종료될 때까지 대기합니다.

```kotlin
val dispatcher = newFixedThreadPoolContext(1, "one-thread-pool").executor.asCoroutineDispatcher()
@Test
fun `runBlocking`(): Unit = runBlocking(dispatcher) {
    launch {
        runBlocking(Dispatchers.IO) {
            log.info("runBlocking")
            delay(1000)
        }
    }
    launch {
        log.info("Success")
    }
}
```

![Untitled](https://choiseonjae.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fdf7fe4f8-566c-497d-8c37-c5b69bc3f7f5%2F6be50b9b-8029-4d55-9d97-159c7c67bf58%2FUntitled.png?table=block&id=2d65b024-79b9-4c2e-bcfb-7a821ab81888&spaceId=df7fe4f8-566c-497d-8c37-c5b69bc3f7f5&width=1420&userId=&cache=v2)

중단이 되더라도 thread가 blocking되어 1초 뒤에 Success가 출력되는 것을 확인 할 수 있습니다.

```kotlin
val dispatcher = newFixedThreadPoolContext(1, "one-thread-pool").executor.asCoroutineDispatcher()
@Test
fun `runBlocking`(): Unit = runBlocking(dispatcher) {
    launch {
        coroutineScope {
            log.info("runBlocking")
            delay(1000)
        }
    }
    launch {
        log.info("Success")
    }
}
```

![Untitled](https://choiseonjae.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fdf7fe4f8-566c-497d-8c37-c5b69bc3f7f5%2Ff0051d65-9b5b-4e97-adb8-f4275afdb7fa%2FUntitled.png?table=block&id=10375811-dbbc-803f-ab9d-fe07bd9e048d&spaceId=df7fe4f8-566c-497d-8c37-c5b69bc3f7f5&width=1420&userId=&cache=v2)

동일한 상황에서 runBlocking을 coroutineScope으로 수정만 해도, blocking 이 해소되고 중단된 경우 새로운 코루틴에게 스레드가 할당돼 동작합니다.

### async는 exceptionHandler가 동작하지 않는다. (feat. exceptionHandler)

exceptionHandler는 자식 코루틴에서 처리되지 않은 예외를 공통 처리를 할 때 유용합니다. 그러나 async에서 발생한 예외는 자동으로 처리되지 않으므로, await()을 호출하는 시점에서 수동으로 처리해야 합니다.

(*exceptionHandler는 launch 에서 정상적으로 동작합니다*)

```kotlin
val exceptionHandler = CoroutineExceptionHandler { _, exception ->
    println("Caught $exception") // 출력되지 않음
}

val deferred = supervisorScope {
    async(exceptionHandler) { // exceptionHandler이 동작하지 않음.
        throw ArithmeticException("Sample Exception")
    }
}

try {
    deferred.await()
} catch (e: ArithmeticException) {
    println("Handled $e") // 출력됨
}
```

### 코루틴 스코프는 병렬 처리를 위한 함수가 아니다.

`withContext`는 코루틴의 실행 컨텍스트를 변경하는 함수입니다. 이 함수는 코루틴을 현재 스레드 또는 디스패처에서 지정된 새 스레드 또는 디스패처로 이동시키는 역할을 합니다. `withContext`는 특정 코루틴을 다른 스레드 또는 디스패처로 전환하는 것을 돕지만, 이것이 병렬 처리를 수행하는 것은 아닙니다.

```kotlin
@Test
fun withContext(): Unit = runBlocking {
    log.info("1")
    withContext(Dispatchers.IO) {
        log.info("2")
        Thread.sleep(1000)
    }

    withContext(Dispatchers.Default) {
        log.info("3")
    }
}
```

![Untitled](https://choiseonjae.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fdf7fe4f8-566c-497d-8c37-c5b69bc3f7f5%2F302adddb-dfa5-4c49-a214-c592e6c21953%2FUntitled.png?table=block&id=10375811-dbbc-801c-acb1-eec7df6390ee&spaceId=df7fe4f8-566c-497d-8c37-c5b69bc3f7f5&width=1420&userId=&cache=v2)

2 → 3을 넘어갈 때 1초를 대기하고 있고,  `코루틴` 또한 변경되지 않고 `@coroutine#1` 로 그대로 입니다.
withContext는 현재 코루틴의 컨텍스트(스레드도 포함)를 변경해줄 뿐, 병렬 동작을 지원하는 것은 아닙니다.

다만, 코루틴 빌더와 함께 사용하여 병렬 로직을 구현할 수 있습니다.

```kotlin
@Test
fun withContext(): Unit = runBlocking {
    log.info("1")
    launch {
        withContext(Dispatchers.IO) {
            log.info("2")
            Thread.sleep(1000)
        }
    }

    launch {
        withContext(Dispatchers.Default) {
            log.info("3")
        }
    }
}
```

![Untitled](https://choiseonjae.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fdf7fe4f8-566c-497d-8c37-c5b69bc3f7f5%2Fcaeca103-8119-414b-9526-8e971cf9fff5%2FUntitled.png?table=block&id=10375811-dbbc-80a8-8f10-c87d8c574d6a&spaceId=df7fe4f8-566c-497d-8c37-c5b69bc3f7f5&width=1420&userId=&cache=v2)

코루틴 `@coroutine#2`와 `@coroutine#3`가 생성됨과 동시에 서로 다른 스레드를 할당하여 1초 대기 없이 전부 병렬 처리되는 것을 확인할 수 있습니다.

### 중단 함수는 중단 될 수 있는 함수이지 항상 중단되는 것은 아니다.

suspend를 붙이더라도, 실제 중단이 발생하지 않으면 그냥 순차적으로 로직이 수행됩니다.

```kotlin
@Test
fun `중단`(): Unit = runBlocking {
    launch { log.info("1") }
    noSuspend()
    log.info("2")
    yield()
    log.info("3")
}

suspend fun noSuspend() {
    
}
```

![Untitled](https://choiseonjae.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fdf7fe4f8-566c-497d-8c37-c5b69bc3f7f5%2F9ac34243-2514-464c-98ab-b77679d32272%2FUntitled.png?table=block&id=2c262a2b-4bb7-4457-9902-a50af5190a41&spaceId=df7fe4f8-566c-497d-8c37-c5b69bc3f7f5&width=1420&userId=&cache=v2)

2 → 1 → 3 순으로 수행된 이유는 launch로 자식 코루틴을 생성 후 noSuspend() 를 호출 → 중단 없이 함수 종료 → 2 출력 → `yield()` 중단시키는 함수 → queue 있던 자식 코루틴 수행 → 1 출력 → 다시 재개 → 3 출력 

suspend 가 붙은 중단 함수는 항상 중단하는 것은 아니고, 중단될 수 있음을 뜻합니다.

### 코루틴 빌더에서 발생한 오류는 잡아도 계속 전파된다.

코루틴에서의 오류 처리는 코루틴의 계층 구조와 밀접하게 연결되어 있습니다. 일반적으로, 자식 코루틴에서 발생한 예외는 부모 코루틴으로 전파됩니다. 이는 코루틴의 "부모-자식" 관계에서의 예외 전파 메커니즘이 설계된 방식입니다. 

자식 코루틴에서 예외를 캐치하고 처리할 수 있지만, 이것이 예외의 전파를 중지하지는 않습니다. 예외를 캐치하고 로그를 남기거나, 특정 상태를 업데이트하는 등의 처리를 할 수는 있으나, 이후 예외는 여전히 부모 코루틴으로 전달됩니다. 

```kotlin
@Test
fun `예외 전달`(): Unit = runBlocking {
    try {
        launch {
            throw RuntimeException("Sample Exception")
        }
    } catch (e: Exception) {
        log.error("Caught $e")
    }

    yield() // launch 의 실행 시점은 여기 입니다
    log.info("Success")
}
```

![Untitled](https://choiseonjae.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fdf7fe4f8-566c-497d-8c37-c5b69bc3f7f5%2Fe1d68577-0b1a-42bc-8e4b-94a59351cf7d%2FUntitled.png?table=block&id=10375811-dbbc-8078-97a2-da6122081693&spaceId=df7fe4f8-566c-497d-8c37-c5b69bc3f7f5&width=1420&userId=&cache=v2)

오류 전파를 막기 위해서는 두가지 방법이 있습니다.

- supervisorScope 사용하기: supervisorScope는 자식 코루틴에서 발생한 예외가 부모 코루틴으로 전파되지 않도록 합니다. 이 스코프 내에서 자식 코루틴의 실패가 다른 자식 코루틴에 영향을 미치지 않습니다.

```kotlin
@Test
fun `예외 전달`(): Unit = runBlocking {
    try {
        supervisorScope {
            launch {
                throw RuntimeException("Sample Exception")
            }
        }
    } catch (e: Exception) {
        log.error("Caught $e")
    }

    yield()
    log.info("Success")
}
```

![Untitled](https://choiseonjae.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fdf7fe4f8-566c-497d-8c37-c5b69bc3f7f5%2F8ddce0db-8ed2-4fe7-8cf2-b1d22df03f92%2FUntitled.png?table=block&id=10375811-dbbc-8022-ba40-da5c9da576d4&spaceId=df7fe4f8-566c-497d-8c37-c5b69bc3f7f5&width=1420&userId=&cache=v2)

- try-catch 블록 사용하기: 자식 코루틴을 try-catch 블록으로 감싸서 예외를 캐치할 수 있습니다.

```kotlin
@Test
fun `예외 전달`(): Unit = runBlocking {
    launch {
        try {
            throw RuntimeException("Sample Exception")
        } catch (e: Exception) {
            log.error("Caught $e")
        }
    }

    yield()
    log.info("Success")
}
```

![Untitled](https://choiseonjae.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fdf7fe4f8-566c-497d-8c37-c5b69bc3f7f5%2Ff8365c99-e8f0-4583-b014-f3796f6eeaf8%2FUntitled.png?table=block&id=10375811-dbbc-8063-ac57-ee98edbc184e&spaceId=df7fe4f8-566c-497d-8c37-c5b69bc3f7f5&width=1420&userId=&cache=v2)

### GlobalScope 는 요청이 취소되도 계속 진행된다.

GlobalScope는 코틀린 코루틴의 전역 스코프로, 애플리케이션의 생명주기와 독립적으로 존재합니다. 이 스코프 내에서 시작된 코루틴은 애플리케이션 전체에서 공유되며, 특별한 조치를 취하지 않는 한 종료되거나 취소되지 않습니다.

```kotlin
GlobalScope.launch {
    // 병렬 처리
}
```

위와 같이 병렬 로직을 수행하다가 API 도중에 예외가 발생하더라도, GlobalScope는 구조화된 동시성이 깨진 상태이기 때문에 부모-자식 관계가 아닙니다. 결과 여부와 상관없이 계속 병렬로 동작하기 때문에 불필요하게 리소스가 낭비될 수 있습니다.

만약, 현재 코루틴에서 구조화 되지 않고 사용하고 싶다면, 아래와 같이 사용하길 추천드립니다.

```kotlin
val scope = CoroutineScope(SupervisorJob()) // bean으로 설정하여 사용하면 좋습니다.
scope.launch {
		// 병렬 처리
}
```

**공식문서의 안티패턴입니다. `([링크](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-global-scope/))`*

![Untitled](https://choiseonjae.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fdf7fe4f8-566c-497d-8c37-c5b69bc3f7f5%2Fa8659eb6-a8f3-4233-a661-4ef3fd30f75a%2FUntitled.png?table=block&id=10375811-dbbc-802d-95e7-f1347a4aa9af&spaceId=df7fe4f8-566c-497d-8c37-c5b69bc3f7f5&width=1420&userId=&cache=v2)

## 결론

항상 모든 동작에 대한 원리나 내부 코드를 기억하고 있을 필요는 없고,
이렇게 정리해놓은 결론만이라도 최소한 인지하고 있으면 많은 도움이 될 것이라고 생각합니다.

다음 장에서는 Dispatcher에 대한 짧은 동작 원리 설명과 그래서 어떻게 쓰면 되는지 예제를 정리할 예정입니다.
