---
title: \[트랜잭션\] propagation REQUIRES_NEW 주의 사항
categories: 
   - transactional

tags:
   - jpa
   - transactional
   - propagation
   - REQUIRED
   - REQUIRES_NEW
   

last_modified_at: 2022-09-30 

---

@Transactional에서 기본 값은 `REQUIRED` 이다. propagation 설정이 REQUIREDS_NEW일 경우, 롤백 발생했을 때 어떤 식으로 롤백되는지 알아보자.

# 동작 방식 요약

![Untitled](https://user-images.githubusercontent.com/49507736/193454427-58263b59-0478-47b6-89b7-47fe283cde5b.png)
`REQUIRES_NEW` 를 호출하게 되면, 기존 부모 트랜젝션으로부터 호출되었다고 하더라도 새로운 트랜젝션을 생성해서 로직이 동작한다. (DB 커넥션도 실제로 다르다.)

![img_5.png](https://user-images.githubusercontent.com/49507736/193454503-31165770-1989-47f4-8aa4-bc0a3aefc3cf.png)

(출처: [https://woodcock.tistory.com/40](https://woodcock.tistory.com/40))

간혹, 다른 블로그에서 표현하는 “*완벽하게 독립적이다” 라는 의미와는 조금 다른게 thread 자체는 동일하다.*

코드 상 롤백이 되는 경우를 그림으로 표현하면 다음과 같다.

![img_1.png](https://user-images.githubusercontent.com/49507736/193744022-0d1aab1a-5869-4018-bce8-cb3ded159007.png)

물론, 2번에서 발생한 오류(Unchecked Exception)은 잡지않으면 계속 위로 올라가기 때문에 전체 롤백이 발생한다. 즉, 2번에 오류 발생했고, try-catch로 잡았다고 당연히 가정한다.

결과를 살펴보면, **예외가 1번에서 발생하면 당연히 모든 로직에 대해서 저장되지 않는다.**

2번에서 발생한 경우, **1번과 3번의 로직은 동작하고 2번만 롤백된다.**

3번에서 발생한 경우, **2번은 이미 commit이 동작하고(트랜잭션이 종료되고) 난 시점이기 때문에 1,3번은 롤백되고 2번은 롤백되지 않는다.**

각 위치별 예외시 롤백 적용범위를 잘 알아둬야 의도치 않은 버그가 생기지 않도록 조심할 수 있을듯 하다.

## Rollback mark가 생기지 않는 경우
`@Transactional`의 propagation REQUIRED에서 Exception이 발생해도 rollback 마크가 생기지 않는 경우가 존재한다.  

한 Depth를 내려가지 않고, (== 다른 메소드를 호출하지 않고) 발생한 오류에 대해서 try-catch 했을 경우 AOP 가 인식하지 못하여 rollback mark를 새기지 못한다.
이런 경우, 오류를 잡으면 rollback 이 되지 않기 때문에 `@Transactional` 코드를 작성할 때 고려해서 의도치 않은 버그를 예방하면 좋을 거 같다.
예제 코드를 보면 다음과 같다.
```java
@Transactional
fun test() {
    reposioty.save(Test.builder().testId(1L).build());
    
    try {
        reposioty.save(Test.builder().testId(2L).build());
        throw new RuntimeException();
    } catch (e : Exception) {
        // 같은 method안에서 오류 발생하고 잡은 경우에 대해서는 aop가 인식하지 못하기 때문에 rollback되지 않는다.
    }
}
```


# 코드

```java
@Service
@RequiredArgsConstructor
public class ParentService {

  private final Repository repository;
  private final ChildService childService;


  @Transactional
  public void parent() {
    repository.save(Entity.builder().id(1L).build());
    childService.child();
    repository.save(Entity.builder().id(3L).build());
    throw new RuntimeException(); -> 3번 발생 시점
  }

  @Transactional
  public void parent2() {
    repository.save(Entity.builder().id(1L).build());
    try {
        childService.childThrowException(); -> 2번 발생 시점
    } catch(Exception e) {
        log.info("child fail");
    }
    repository.save(Entity.builder().id(3L).build());
    throw new RuntimeException();
  }
}
```

```java
@Service
@RequiredArgsConstructor
public class ChildService {

    private final Repository repository;
	
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void child() {
        repository.save(Entity.builder().id(2L).build());
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void childThrowException() {
        repository.save(Entity.builder().id(2L).build());
        throw new RuntimeException();
    }
}
```

위와 같이 `parent()`, `parent2()` 를 테스트 해봤을 경우,

`parent()`는 2번 id row가 들어가고,

![img_4.png](https://user-images.githubusercontent.com/49507736/193454491-926a41e8-2652-4048-834c-e1645246dda2.png)

`parent2()` 는 1번, 3번 id row가 insert되었다.

![img_2.png](https://user-images.githubusercontent.com/49507736/193454485-a6bd1296-958e-4149-99ea-0e052597536e.png)

# 원인 파악
이유에 대해서 팀원과 이야기 나눠봤는데, 파악한 이유는 다음과 같다.
```
부모.doBegin() {
  try {
    save1()
    
    자식.doBegin() {
      try {
        save2()
      } catch (e: Exception) {
        rollback()
      }
        commit() // 해당 시점에 이미 save2가 날라가 버리기 때문...
    }
    
    save3()
  } catch (e: Exception) {
    rollback()
  }
  commit()
}
```
_이미 DB로 커밋되어 버렸기 때문에 밑에서 rollback이 되더라도 자식 트랜젝션은 돌릴수가 없다_

# 출처
- https://woodcock.tistory.com/40
- https://lemontia.tistory.com/878
- https://techblog.woowahan.com/2606/
