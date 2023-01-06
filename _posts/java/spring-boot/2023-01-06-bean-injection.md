---
title: \[Spring Boot] Bean 주입 순서 (번외. 왜 @Autowired를 쓰지 말아야 할까?)
permalink: /java/spring-boot/bean/injection

categories: spring-boot

tags:
  - bean
  - DI
  - bean injection
  - autowired
  - Constructor Injection

last_modified_at: 2023-01-07 01:16:00

---

우리가 만든 Bean들은 스프링 부트가 자동으로 넣어 주고 있지만, 항상 하나의 객체당 하나의 Bean만 생성되는 것도 아니기 때문에 여러가지 상황에서 스프링 부트가 어떤 우선순위로 bean을 주입해주고 있는지 알아보자.

# Bean의 주입 기준
`@Bean`으로 bean을 생성하게 되면, method name이 bean name으로 생성된다.

스프링에서 bean을 주입하는 코드는 아래처럼 

`DefaultListableBeanFactory.doResolvedDependency()`를 통해서 일어나고 있다.

실제로 코드를 까보고 싶다면 `DefaultListableBeanFactory.java`를 보면 된다.

## 같은 Type의 bean이 1개만 있다면
bean name과 관련없이 bean을 주입해준다.

## 같은 Type의 bean이 여러개 있으면
`@Qualifier`가 없어도 bean name과 field name을 매칭해서 bean을 주입해준다.

필드명과 매핑되는 빈이 없으면 `NoUniqueBeanDefinitionException` 오류가 발생한다.

## @Qualifier가 있으면
그 기준으로 bean을 찾아준다. 해당 이름을 가진 빈이 없으면 오류가 난다.

## @Primary가 있으면
bean name을 무시하고 Type 기반으로 Primary인 Bean을 주입한다.

Primary가 아닌 빈을 넣어주고 싶을 때, `@Qualifier`를 쓰면 된다.

# 번외

## @Autowired가 deprecated 된 이유
필드 injection(`@Aurowired`)같은 경우, 매칭되는 빈이 없으면 `return null`을 반환한다.

```java
Map<String, Object> matchingBeans = findAutowireCandidates(beanName, type, descriptor);
if (matchingBeans.isEmpty()) {
    if (isRequired(descriptor)) {
        raiseNoMatchingBeanFound(type, descriptor.getResolvableType(), descriptor);
    }
    return null;
}
```

그럴 경우, 실제 빈을 사용하는 시점에서 Runtime에 NPE가 발생한다.

그렇기 때문에 요즘엔 `@RequiredArgConstructor`를 활용한 Constructor Injection을 주로 사용한다.

Constructor Injection의 경우, `final`접근자로 인해서 

bean Injection 시점(spring context loading시)에 오류가 발생해서 Runtime에 오류가 발생하는 것을 막을 수 있다.  
