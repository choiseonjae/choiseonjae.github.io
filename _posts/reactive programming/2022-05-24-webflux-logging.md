---
title: \[Webflux] Webflux MDC transactionId(traceId, requestId) logging 

header:
  teaser: https://images.velog.io/images/neity16/post/5679215a-8961-4529-9469-077f768c66d2/webflux.jpeg

categories: webflux

tags:
  - kotlin
  - webflux
  - reactor
  - reactive
  - reactive programming
  - logging
  - MDC

last_modified_at: 2022-05-24

---


우리 서비스들을 모니터링 하다보면, 생각 외로 오류를 추적하기가 쉽지 않은 편이였다.  
requestId(traceId)를 요청 시점에 넣어주고는 있지만, webflux 환경이기 때문에 계속 변경되기 도 하고 user 식별자 또한 msg에 직접 넣는 형식으로 기록되기 때문에 식별자 또한 누락되는 경우가 많았다.  

그래서 항상 오류에 대한 문의가 들어왔을 때 빠르게 파악하기가 쉽지는 않았던 거 같다. (실력은 차치하더라도)  
그러던 와중에 회사에서는 하우스키핑데이라는 날이 존재해서 평소에는 우선순위가 밀려 하기 힘들었던 업무를 하는 날을 제공해주었고, 나는 서비스의 로그를 고도화하는 걸 주제로 잡았다.

# Webflux 로깅하기
주제는 정해졌지만
 막상 시작하기에는 webflux의 경우, Spring MVC와는 다르게 참고할 수 있는 자료들이 거의 없었다.

webflux에서는 잦는 thread 변경이 발생하기 때문에 그냥 MDC에 저장하면 requestId가 중간에 사라지기 때문에 Hook을 통해 변경 시점 전에 context에 MDC 내용을 저장하여 MDC에 copy하는 작업이 필요하다.

```kotlin
package com.kakao.pay.account.web.config

import com.kakao.pay.account.web.filter.MdcLoggingFilter
import com.kakao.pay.account.web.kotlinx.utils.xAuth
import com.kakao.pay.account.web.util.UuidGenerator
import org.apache.logging.log4j.LogManager
import org.reactivestreams.Subscription
import org.slf4j.MDC
import org.springframework.context.annotation.Configuration
import reactor.core.CoreSubscriber
import reactor.core.publisher.Hooks
import reactor.core.publisher.Operators
import reactor.util.context.Context
import java.util.stream.Collectors
import javax.annotation.PostConstruct
import javax.annotation.PreDestroy

@Configuration
class MdcContextLifterConfiguration {

    companion object {
        val MDC_CONTEXT_REACTOR_KEY: String = MdcContextLifterConfiguration::class.java.name
    }

    @PostConstruct
    fun contextOperatorHook() {
        Hooks.onEachOperator(MDC_CONTEXT_REACTOR_KEY, Operators.lift { _, subscriber -> MdcContextLifter(subscriber) })
    }

    @PreDestroy
    fun cleanupHook() {
        Hooks.resetOnEachOperator(MDC_CONTEXT_REACTOR_KEY)
    }

}

/**
 * Helper that copies the state of Reactor [Context] to MDC on the #onNext function.
 */
class MdcContextLifter<T>(private val coreSubscriber: CoreSubscriber<T>) : CoreSubscriber<T> {

    override fun onNext(t: T) {
        coreSubscriber.currentContext().copyToMdc()
        coreSubscriber.onNext(t)
    }

    override fun onSubscribe(subscription: Subscription) {
        coreSubscriber.onSubscribe(subscription)
    }

    override fun onComplete() {
        coreSubscriber.onComplete()
    }

    override fun onError(throwable: Throwable?) {
        coreSubscriber.onError(throwable)
    }

    override fun currentContext(): Context {
        return coreSubscriber.currentContext()
    }
}

/**
 * Extension function for the Reactor [Context]. Copies the current context to the MDC, if context is empty clears the MDC.
 * State of the MDC after calling this method should be same as Reactor [Context] state.
 * One thread-local access only.
 */
private fun Context.copyToMdc() {
    if (!this.isEmpty) {
        val map: Map<String, String> = this.stream()
            .collect(Collectors.toMap({ e -> e.key.toString() }, { e -> e.value.toString() }))
        MDC.setContextMap(map)
    } else {
        MDC.clear()
    }
}

private fun Context.toMap(): Map<Any, Any> = this.stream()
    .collect(Collectors.toMap({ e -> e.key.toString() }, { e -> e.value.toString() }))

fun Context.createPutWithMDC(key: Any, value: Any): Context {
    val mapOfContext = this.toMap().toMutableMap()
    mapOfContext.put(key, value)
    MDC.put(key.toString(), value.toString())
    return Context.of(mapOfContext)
}
```

위 코드를 추가하면 이제 Context에 저장하면 onNext()가 호출되는 시점에 Context에 저장되어있던 값을 MDC에 넣어주게 된다. 
나같은 경우에는 그냥 Context에 put을 하게되면 Context가 Immutable해서 그런지 값이 들어가지 않았고, Map을 만들어서 거기에 내가 넣고 싶은 값과 기존의 Context 값을 같이 넣어서 새롭게 Context.of(map) 의 형태로 넘겼다.   

요청받는 시점에 바로 requestId 추가하기 위해서 webfliter를 추가해 chain.filter(exchange).subscriberContext { } 안에서 작업해줬다.
아래 코드를 좀 보면서 분석해보면 쉽게 이해가 될 것이다.

```kotlin
import org.apache.logging.log4j.LogManager
import org.springframework.core.annotation.Order
import org.springframework.stereotype.Component
import org.springframework.web.server.ServerWebExchange
import org.springframework.web.server.WebFilter
import org.springframework.web.server.WebFilterChain
import reactor.core.publisher.Mono

@Component
@Order(0)
class MdcLoggingFilter(): WebFilter {

    companion object {
        private val log = LogManager.getLogger()!!
    }

    override fun filter(exchange: ServerWebExchange, chain: WebFilterChain): Mono<Void> {
        return chain.filter(exchange).subscriberContext {
           it.createPutWithMDC("reqeustId" to UuidGenerator.generate())
        };
    }
}
```

# 출처
- [https://mjin1220.tistory.com/52](https://mjin1220.tistory.com/52)
- [https://lucas-k.tistory.com/9](https://lucas-k.tistory.com/9)
- [https://techblog.woowahan.com/2667](https://techblog.woowahan.com/2667/)