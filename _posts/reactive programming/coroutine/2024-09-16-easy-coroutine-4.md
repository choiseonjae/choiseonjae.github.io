---
title: \[코루틴] 코루틴 그래서 어떻게 써야할까?

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

내부 동작이나 원리를 아는 것도 중요합니다. 제일 중요한 건 실무에서 실제로 사용하는 것이고, 어느 상황에 어떻게 쓰면 되는지를 정리해 놓는 걸 좋아합니다.

원래는 스터디를 마치고 혼자 보는 노션에만 정리해두는 편이지만 잘 정리해서 블로그에도 올려봅니다.

### Dispatcher 는 어떻게 동작할까?

전 포스팅에서 말했듯이 간단하게 Dispatcher의 동작 원리를 설명해볼 텐데

코루틴에 Thread를 할당해주는 주체를 Dispatcher 입니다.

생성된 코루틴은 globalQueue에 적재되고, 진행되고 있는 코루틴이 중단되면 Dispatcher가 비워진 스레드에 다른 코루틴을 할당합니다. (중단된 코루틴이 resume되면 다시 queue에 적재됩니다)

![Untitled](https://choiseonjae.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fdf7fe4f8-566c-497d-8c37-c5b69bc3f7f5%2Fff7031ae-7324-4fbe-8fa6-1df6e90c5477%2FUntitled.png?table=block&id=1df8116c-caec-46e9-8c64-58353f005d07&spaceId=df7fe4f8-566c-497d-8c37-c5b69bc3f7f5&width=1420&userId=&cache=v2)

![Untitled](https://choiseonjae.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fdf7fe4f8-566c-497d-8c37-c5b69bc3f7f5%2Ff4617370-0d0e-49b2-a020-8d41cbc90867%2FUntitled.png?table=block&id=06ed20a2-20d3-4860-99ff-d0acf59d4230&spaceId=df7fe4f8-566c-497d-8c37-c5b69bc3f7f5&width=1360&userId=&cache=v2)

![Untitled](https://choiseonjae.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fdf7fe4f8-566c-497d-8c37-c5b69bc3f7f5%2F38b2f893-eeb7-41ea-82ac-9aed19eaca0e%2FUntitled.png?table=block&id=10375811-dbbc-8053-ac04-dccd3a79a999&spaceId=df7fe4f8-566c-497d-8c37-c5b69bc3f7f5&width=1360&userId=&cache=v2)

![Untitled](https://choiseonjae.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fdf7fe4f8-566c-497d-8c37-c5b69bc3f7f5%2Fb6eaa53b-f882-4059-8700-3cc2d0a2e290%2FUntitled.png?table=block&id=10375811-dbbc-8086-85a7-d9b4ab8cddcd&spaceId=df7fe4f8-566c-497d-8c37-c5b69bc3f7f5&width=1360&userId=&cache=v2)

![Untitled](https://choiseonjae.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fdf7fe4f8-566c-497d-8c37-c5b69bc3f7f5%2Fa5b28c4c-d745-4868-9f6d-ac5fd7aa745b%2FUntitled.png?table=block&id=10375811-dbbc-80b7-a6dd-c3985e52ab7b&spaceId=df7fe4f8-566c-497d-8c37-c5b69bc3f7f5&width=1360&userId=&cache=v2)

![Untitled](https://choiseonjae.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fdf7fe4f8-566c-497d-8c37-c5b69bc3f7f5%2F8a3327ca-9855-428a-9d01-fd9dacf16b94%2FUntitled.png?table=block&id=10375811-dbbc-800f-9861-f735dec89d1c&spaceId=df7fe4f8-566c-497d-8c37-c5b69bc3f7f5&width=1360&userId=&cache=v2)

![Untitled](https://choiseonjae.notion.site/image/https%3A%2F%2Fprod-files-secure.s3.us-west-2.amazonaws.com%2Fdf7fe4f8-566c-497d-8c37-c5b69bc3f7f5%2Fd3b3208d-edc5-413c-8e7d-cab5c34ed1c6%2FUntitled.png?table=block&id=1c478a1e-bcb7-46d6-8789-52724990686e&spaceId=df7fe4f8-566c-497d-8c37-c5b69bc3f7f5&width=1360&userId=&cache=v2)

## 코루틴 사용 예제

아래는 코루틴에 대한 상황별 사용 예제를 정리했습니다.

기본적인것부터 병렬 처리, 동시성에 대한 내용도 담았습니다.

처음 사용하신다면 보시는 것을 추천드립니다!

### 나는 병렬 로직을 작성하고 싶다.

```kotlin
coroutineScope {
  val job = launch(Dispatchers.IO) { // Dispatchers는 변경 가능
      // 병렬 로직
  }
  
  val defferd = async(Dispatchers.IO) { // Dispatchers는 변경 가능
      // 병렬 로직
  }
  launch { // Dispatchers는 미 설정 시, 동시성 으로 동작 (기존 Thread 반환 시 동작)

  }
}
```

coroutineScope의 모든 코루틴이 종료될 때까지, 현재 코루틴이 중단된 후 수행됩니다.
coroutineScope 안에서는 job과 defferd가 병렬로 수행됩니다.

### 코루틴 스코프 안에서도 자식 코루틴 완료를 기다리고 진행하고 싶다.

예를 들어, 모든 회원에게 메세지를 보내고 싶을 때 chunk 단위로 작업을 진행할 때 chunk 단위의 자식 코루틴을 기다리는 방법입니다.

```kotlin
// 매번 코루틴 스코프를 생성할 때,
while (true) {
  supervisorScope { // 자식 코루틴 간의 예외 무시. chunk 단위도 대기하는 효과는 있지만 매번 스코프 생성
      val users = findAllUsers(pagable) // chunk 1,000
      user.forEach {
          launch {
              // 메세지 전송
          }
      }
  }
}
```

### 코루틴 스코프 안에서 성공한 자식 코루틴의 결과만 모으고 싶다.

예외 처리를 하지 않고 awaitAll()을 하면 한 건이라도 오류 발생 시, 예외가 위로 전파됩니다.

```kotlin
supervisorScope {
  val users = findAll()
  val birthdayList = user.mapNotNull {
      async {
          runCatching { getBirthday(it.payAccountId) }
              .getOrNull()
      }
  }.awaitAll()
}
```

### 원하는 결과 오는대로 빠른 반환을 하고 싶다. (1)

하나라도 동의한 약관이 존재할 경우, 바로 true로 반환하고 싶은 경우

```kotlin
fun main() = runBlocking {
  val terms = flowOf("TERM1", "TERM2", "TERM3", "TERM4", "TERM5")

  val isAgreed = terms
      .flatMapMerge(concurrency = 5) { term ->
          flow { emit(checkAgreement(term)) }.flowOn(Dispatchers.IO)
      }

  val a = isAgreed.firstOrNull { it.second } ?: false

  println("${Thread.currentThread().name} 첫 번째 동의한 약관: $a")
}

suspend fun checkAgreement(term: String): Pair<String, Boolean> {
  val sleep = (1000..5000).random().toLong()
  println("${Thread.currentThread().name} Checking agreement for $term sleep: $sleep")
  callApi(sleep)
  val agreed = listOf(true, false).random()
  println("${Thread.currentThread().name} Agreement for $term: $agreed")
  return term to agreed
}

suspend fun callApi(sleep: Long) = withMDCContext(Dispatchers.IO) {
  delay(sleep)
}
```

**로그** 

```kotlin
DefaultDispatcher-worker-4 Checking agreement for TERM4 sleep: 2759
DefaultDispatcher-worker-1 Checking agreement for TERM1 sleep: 2641
DefaultDispatcher-worker-2 Checking agreement for TERM2 sleep: 3999
DefaultDispatcher-worker-5 Checking agreement for TERM5 sleep: 4348
DefaultDispatcher-worker-3 Checking agreement for TERM3 sleep: 4045
DefaultDispatcher-worker-1 Agreement for TERM1: false
DefaultDispatcher-worker-4 Agreement for TERM4: true
main 첫 번째 동의한 약관: (TERM4, true)
```

### 원하는 결과 오는대로 빠른 반환을 하고 싶다. (2)

위의 상황에서는 사실 중단 함수에서만 flow가 빠져나올 수 있습니다.
우리는 blocking call 많이 하기 때문에 Thread를 할당해주더라도 (뒷 단에서 돌고 있더라도) 반환하고 싶을 수 있습니다.

기본적으로 코루틴에서 제공해주는 blocking 시, 빠져나오는 기능을 제공하지는 않습니다.
(제가 알아본 바로는)

그래서 flow, channel 을 통해 책임을 분리하고 call blocking 되어 있더라도 channel에서 바로 이어서 작업하는 방식으로 구현했습니다.

(바로, channel로 구현하면 만약 emit()이 없으면 무한 대기로 이뤄질 수 있기 때문에 flow → channel의 형태로 구현해야 합니다)

```kotlin
@Test
fun channelForAsync(): Unit = runBlocking {
    val startTime = System.currentTimeMillis()

    val rc = flowOf(1, 2, 3, 4, 5)
        .flatMapMerge(concurrency = 5) {
            flow {
                withContext(Dispatchers.IO) {
                    Thread.sleep(it * 1000L)
                }
                val result = it == 3
                if (result) emit(it)
            }
        }
        .produceIn(CO_SUPERVISOR_SCOPE)

    val result = runCatching { rc.receive() }
        .getOrDefault("TEST")
    rc.cancel()
    log.info("result $result")
    log.info("time: ${(System.currentTimeMillis() - startTime) / 1000.0}")
}
```

위 처럼 코드를 짜면 여러 상황 중에서 내가 원하는 결과가 온 상황에 대해서 빠른 반환을 할 수 있습니다. 가령, 1,2,3,4,5 약관에 대해서 동의 여부 체크하는데 하나라도 동의 안되어있으면 오류 던질 경우, 전체를 체크 하되 미동의 약관이 하나라도 내려오는 경우 모두 기다리지 않고 빠져나와 오류를 던질 수 있습니다.

### 어떤 결과든 빠르게 도착하는 코루틴에 대해서 처리하고 싶다.

결과에 상관없이 먼저 반환 오는 값을 이용하려고 할 때 사용할 수 있는 예제가 있습니다.

select 입니다.

```kotlin
@Test
fun selectTest(): Unit = runBlocking {
    val result = select<String> {
        async { randomDelayTask("작업 A") }.onAwait { it }
        async { randomDelayTask("작업 B") }.onAwait { it }
        async { randomDelayTask("작업 C") }.onAwait { it }
    }
    println("결과: $result")
}

suspend fun randomDelayTask(name: String): String {
    val delay = Random.nextLong(1000, 4000)
    delay(delay)
    return "$name (${delay}ms)"
}
```

위 코드를 실행해보면 각 랜덤 작업 중 먼저 도착한 것먼저 빠른 종료를 하게 됩니다.

## 결론

실무에서 요구사항은 다양하고, 그 때 그 때 맞는 예제를 쓰기 위해서 각 요구사항별 코드를 미리 정리해봤습니다. 저도 실제로 실무에서 사용하고 있습니다.

도움이 되었길 바랍니다!
