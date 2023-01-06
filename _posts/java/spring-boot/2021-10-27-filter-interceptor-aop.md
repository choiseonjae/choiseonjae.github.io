---
title: \[Spring Boot\] Filter vs Interceptor vs AOP 의 차이 초간단 정리
permalink: /java/spring-boot/fileter-interceptor-aop

categories: spring-boot

tags:
   - filter
   - interceptor
   - aop

last_modified_at: 2021-10-27 19:21:32.71 

---
# Filter vs Interceptor vs AOP

각각의 기능들은 어떤 부가적인 작업에 대해서 공통적으로 처리해준다는 특징을 가지고 있다. 좀더 구체적으로 어떤 차이가 존재하는지 알아본다.

# Filter

J2EE의 표준 스펙 기능으로 Client가 Dispatcher Servlet으로 가기 전 / 후 로 url 패턴에 맞게 부가작업을 처리한다.

web context 안에서 동작한다. 

spring context가 아닌 톰캣과 같은 웹 컨테이너에 의해 관리된다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbZQx9K%2Fbtq9zEBsJ75%2FdEAKj1HEymcKyZGZNOiA80%2Fimg.png)

# Interceptor

Spring이 제공하는 기능. 즉, Spring Context에서 동작한다.

Dispatcher Servlet이 적절한 Controller에게 요청하기 위해 Handler Mapping으로 Controller를 찾아달라고 요청하는데 이때의 반환값이 HandlerExceptionChain(실행체인)이다.

HandlerExceptionChain(실행체인)은 1개 이상의 Interceptor가 등록되어 있으면 순차적으로 실행 후에 Controller를 수행한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FSz6DV%2Fbtq9zjRpUGv%2F68Fw4fZtDwaNCZiCFx57oK%2Fimg.png)

# AOP

위의 Filter와 Interceptor는 Servlet 단위로 동작한다면 AOP는 method 앞에서 proxy되어 동작한다.

그렇기 때문에 3개중에서 제일 늦게 실행된다.

실행 순서로는 Filter → Interceptor → AOP → Interceptor → Filter 순이다.

또한, Filter와 Interceptor는 url 패턴으로 구분되어 대상을 타겟한다면 AOP는 url 패턴, 파라미터, 어노테이션 등 다양한 방법으로 대상을 타겟한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F1bEhb%2FbtqH8cRq0sY%2FdQVkF7pbrdOTVnILW7bmzK%2Fimg.png)


# 차이점과 용도

1. Filter는 web-context / Interceptor, AOP는 spring-context에서 동작한다.
2. Filter만 ServletRequest / ServletResponse 객체를 조작할 수 있다.
3. 용도
    1. Filter는 보안, 로깅, 문자열 인코딩 등 Spring과 무관하게 전역에서 처리해야하는 작업을 할 때 사용한다.
    2. Interceptor는 인증 / 인가, Controller에 넘겨주는 정보 가공, API 호출 로깅등의 작업을 할 때 사용한다.
        1. 세션 쿠키 등을 체크하는 Http 프로토콜 단위로 처리하는 작업에 용이하다.
