---
title: \[JPA] @Transactional(readOnly = true)
permalink: /java/spring-boot/transactional/readOnly

categories: jpa

tags:
   - annotation
   - readOnly
   - transactional
   

last_modified_at: 2021-12-14 15:21:32.71 

---

스프링 부트에서 트랜젝션을 읽기 전용으로 설정할 수 있다.

```java
@Transactional(readOnly = true)
```

`class` 명 위에 해당 어노테이션을 작성하면, Spring Boot가 Hibernate Session flush mode를 `Manual`로 변경한다. 강제로(수동으로) flush를 호출하지 않는 한 flush가 일어나지 않는다.

트랜젝션을 commit 하더라도 영속성 컨텍스트가 flush를 일으키지 않아서, 엔티티의 등록, 수정, 삭제가 동작하지 않는다. 또한, 읽기 전용으로 영속성 컨텍스트의 변경 감지를 위한 snapshot을 보관하지 않으므로 성능이 향상된다.

# 3줄 요약

1. 트랜젝션이 끝나도 flush가 일어나지 않는다.
2. 엔티티 변경을 해도 동작하지 않는다. (강제로 flush하지 않는한)
3. 영속성 컨텍스트 변경감지를 위한 snapshot이 필요없어지므로 성능이 향상된다.
