---
title: "\\[JAVA] ConcurrentHashMap 이란? (+ 다른 유사한 자료구조)"
---

오류를 만났다.. 알고있던 오류였지만 이 메소드에서 발생할 수 있다는 점은 놀라웠다.

![error_picture](/assets/image/java/ConcurrentMap/ArrayIndexOutOfBoundsException.png)


정말 많은 양의 데이터를 처리해야 했다.   

조금이라도 성능을 올릴려고 발악해보았다.

여러 쓰레드를 동시에 수행시켰고, 스킵가능한 데이터는 스킵시키기 위해 List를 만들었다.

되게 당연한 오류였지만.. 생각없이 구현하다 실수했고, 발생한 김에 정리해본다.

먼저, 이슈가 발생한 이유는 여러 개의 thread가 thread-safe한 자료구조가 아니라 ArrayList를 사용해 동시 접근으로 인해 오류가 발생했다.

`list.get()`도 아니고 `list.add()`에서 `ArrayIndexOutOfBoundsException` 오류라니 신기했다.

# 일반적인 자료구조

thread-safe한 자료구조를 알아보기 전에 간략하게 일반적인 자료구조의 동작방식에 대해 정리해보자.

## List

내가 위해서 말헀던, 한번 발생했던 id에 대해서 스킵하려고 했다면 List를 썼으면 안됬다.

조회할 때 하나씩 조회하기 때문에 데이터가 쌓일 수록 지연이 늘어난다.

(성능을 높이려고 한 작업이 더 늦어진다..)

![list_contains](/assets/image/java/ConcurrentMap/list_contains.png)

## Set

Set은 조회시에 map과 동일한 방법으로 조회한다. 

생성 시점에 map을 생성하고, map을 set처럼 관리한다.


![hashSet_constructor](/assets/image/java/ConcurrentMap/HashSet_constructor.png)

![hash_contains](/assets/image/java/ConcurrentMap/set_containsKey.png)

## Map

map은 아래와 같이 key를 hash화 해서 찾고, 

같은 hash라면 연결될 같은 Node들을 돌면서 찾는다.

![map_contain](/assets/image/java/ConcurrentMap/map_containsKey.png)


![getNode](/assets/image/java/ConcurrentMap/map.png)


여기까지가 기본적인 ArrayList, HashSet, HashMap이다.

# Synchronized 자료구조
## Collections.synchronizedSet()
![Collections.synchronizedSet_contains](/assets/image/java/ConcurrentMap/Collections_synchronizedSet.png)

contains 메소드 자체에 `synchroized`가 걸려있는 걸 볼 수 있다. 이런 경우, 메소드가 임계 구역으로 설정되 thread가 점유하고 있을 때, 다른 쓰레드가 접근하려고 하면 Lock이 걸려 병목 현상이 발생한다.

## Collections.synchronizedMap()

![Collections.synchronizedMap()contains](/assets/image/java/ConcurrentMap/concurrentHashMap_containsKey.png)

이 자료구조 또한 `Collections.synchronizedSet()`과 동일하게 병목 현상이 발생하여, 성능에 하락이 있을 수 있다.

## HashTable

Map 이다. 모든 메소드에 `synchroized`가 붙어있다. 

위와 같이 느리다는 단점이 있다. 

현재로서는 잘 안쓰이는 자료구조이다.

![hash_table_contain](/assets/image/java/ConcurrentMap/HashTable.png)

## ConcurrentHashMap
Hashtable 클래스의 단점을 보완하면서 Multi-Thread 환경에서 사용할 수 있도록 나온 클래스가 바로 `ConcurrentHashMap` 이다.

contains 코드를 보면 `synchroized`가 없다. 

그래서 여러 쓰레드가 동시에 접근해서 병목현상이 없고, 빠르게 조회가 가능하다.

![](/assets/image/java/ConcurrentMap/concurrentHashMap_containsKey.png)

![](/assets/image/java/ConcurrentMap/concurrentMap_get.png)

thread-safe를 위해서 lock 걸린 부분을 살펴보면, 아래 같은 버킷에 대해서만 `synchronized` 예약어가 걸려있는게 보인다.

![](/assets/image/java/ConcurrentMap/concurrentMap_put.png)

put 조차도 key의 해시가 다르면 lock이 걸리지 않고, 같은 해시를 가진 객체에 대해서만 lock 걸리게 된다.

위와 같은 이유로, put / get이 같은 키에서 동시에 발생하면, get은 가장 최근에 업데이트 된 데이터를 반환한다.

```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {

    private static final int DEFAULT_CAPACITY = 16;

    // 동시에 업데이트를 수행하는 쓰레드 수
    private static final int DEFAULT_CONCURRENCY_LEVEL = 16;
}
```

`ConcurrentHashMap` 생성 시 아무런 설정도 하지 않는다면 디폴트로 값이 채워지게 되는데, 여기서 capacity의 값이 중요하다.  

capacity == 버킷의 수 == 동시에 쓰기 작업이 가능한 최대 쓰레드의 수 이기 때문이다.  

capacity이 16인 기준으로 예시를 들어보자.  

key는 key.hashCode()로 int 값을 받아온다.  

![](/assets/image/java/ConcurrentMap/hashcode.png)

그럼 이 값을 capacity의 나머지를 계산해서 hashCode(int) % capacity(16) ==> index가 된다.  

중복하는 index는 Node에 이어서 LinkList처럼 연결되게 된다.  

그래서 같은 index를 많이 가지게 되면 Node를 타고 타고 equal() 함수를 호출한다.

한 공간에 들어가있는 데이터 수가 1 이하면 `get()`메소드를 실행해도 `equals()`를 호출할 필요가 없으니 성능이 빠를 것이다.

만약, 16만개의 데이터가 들어간다면 한 버킷당 만개의 데이터가 들어가고 최대 10000번의 `equals()`가 있을 수 있다. 

그래서 capacity를 처음부터 잘 조절하는게 중요하다.

물론, load_factor(==데이터 수 / capacity *기본값 0.75*)에 의해서 너무 많은 데이터가 저장되면 

자동으로 버킷의 크기를 늘리게 된다. (약 2배씩)

근데, 이때 capacity가 변경되게 되면 index도 달라지고, 다시 넣어야하는 작업이 있기 때문에 엄청난 부하가 발생하니 

처음부터 크기를 알 수 있다면 잘 설정해서 넣어주는 것이 좋다.

그렇다고 처음부터 capacity를 왕창 크게 잡는 것도 능사는 아닌게, 빈 공간을 많이 만들테니 메모리가 낭비되게 된다.

게다가 key에 의한 index 접근이 아니라 Iteration(객체의 모든 데이터를 조회해봐야 하는) 상황에서는 성능이 저하된다.

Iteration의 부하는 (capacity + 저장된 데이터 수)와 비례하기 때문이다.

ex) 16의 capacity는 전체 조회 시 16번을 도는데 (그 안의 데이터는 차치하더라도), 16만개의 capacity는 16만번을 조회해야하기 때문이다.

### 어떻게 동시성 보장을 보장할까?

사실 동시성을 보장하는 방법 중 가장 쉬운 방법은

`synchronized`를 쓰면 안전하게 동시성을 보장할 수 있다.

하지만, 사용하기에 따라서 성능 이슈가 있습니다. 특히 위에서 언급한 `HashTable` 에서 얘기됐던 성능이 가장 큰 문제가 있다.

이를 ConcurrentHashMap은 정말 멋지고 어-썸한 여러가지 방법으로 개-선을 했는데 크게 두 가지 방법들로 해결한다.

그 중에 하나는 위에서 말한 put 중에서도 버킷에 노드가 존재할 때만 `syncronized`를 거는 방법이고,

두번째 방법은 [CAS 알고리즘 사용](https://howtodoinjava.com/java/multi-threading/compare-and-swap-cas-algorithm/)이다.

{% capture content %}

링크에 있는 내용을 번역기로 돌리면 다음과 같다.

메모리 위치의 내용을 주어진 값과 비교하고 동일한 경우에만 해당 메모리 위치의 내용을 주어진 새 값으로 수정합니다. 이것은 단일 원자 연산으로 수행됩니다. 원자성은 최신 정보를 기반으로 새 값이 계산되도록 보장합니다. 그 동안 다른 스레드에서 값을 업데이트한 경우 쓰기가 실패합니다. 작업 결과는 대체를 수행했는지 여부를 나타내야 합니다. 이는 간단한 부울 응답(이 변형은 종종 비교 및 ​​설정이라고 함)을 사용하거나 메모리 위치에서 읽은 값(기록된 값이 아님)을 반환하여 수행할 수 있습니다.

CAS 작업에는 3가지 매개변수가 있습니다.

1. 값을 대체해야 하는 메모리 위치 V  
2. 스레드가 마지막으로 읽은 이전 값 A  
3. V 위에 쓰여져야 하는 새로운 값 B  

즉, 정리하자면 

V 위치에 있는 값 A를 기록해두고, A 값을 수정하여 쓰고자하는 B를 만든다.
V에 덮어쓰기전 V의 값이 A라면 수정하고, A가 아닌 새로운 값이 다른 thread에 의해 수정되었다면 (실패 감지) 다시 처음부터 시작하는 한다.

{% endcapture %}

{% include notice--info title="CAS 알고리즘이란?" content=content %}

- [Unsafe](https://rangken.github.io/blog/2015/sun.misc.unSafe/) 에 대한 내용

![](/assets/image/java/ConcurrentMap/cas.png)

위 밑 줄을 보면, 새로운 Node를 삽입하기 위해, tabAt()을 통해 해당 bucket을 가져오고 bucket == null로 비어 있는지 확인한다.

그리고, 버킷 전체가 들어있는 tab과 그 넣을 위치의 i (== bucket) 이 비어 있을 때, cas(전체 bucket, 위치 (i), null (비어져있었으니까), new Node) 를 호출하게 된다.

bucket이 비어 있을 경우 `casTabAt`을 통해 node를 새로 생성하는데 생성에 실패시에 다시 재시도 한다.

즉, 정확한 위치에 (버킷이 존재할 때만) `synchronized` 와 CAS 알고리즘으로 concurrentHashMap은 원자성과 성능을 챙겼다.


# 오늘의 결론
multi-thread에서 thread-safe하게 사용할 땐, ConcurrentMap을 사용하는 게 좋을 수 있다.

HashMap에 저장될 데이터의 수가 짐작 가능하다면,될 수 있으면 객체를 생성할 때 capacity를 그 값에 맞게 설정해주는게 좋다.

만약 들어올 데이터 수는 16만개인데 초기 capacity를 기본값 그대로 16을 쓴다면, capacity 증가 작업이 매우 많이 일어날 거고, 이게 버킷 크기만 증가 시키는 게 아니라 기존에 저장되어 있던 모든 데이터의 hashCode() % capacity를 다시 계산하고 그 값에 따른 공간 배치를 새로 다 하기 때문에 (== rehashing) 부하가 엄청나다.

그래서 초기 capacity 설정을 잘하자.

# 출처
- https://devlog-wjdrbs96.tistory.com/269
- https://onsil-thegreenhouse.github.io/programming/java/2018/02/22/java_tutorial_HashMap_bucket/
- https://pplenty.tistory.com/17
- https://ktko.tistory.com/entry/%EC%9E%90%EB%B0%94-synchronized%EC%97%90-%EB%8C%80%ED%95%98%EC%97%AC
- https://dgle.dev/ConcurrentHashMap/
- https://howtodoinjava.com/java/multi-threading/compare-and-swap-cas-algorithm/
- https://rangken.github.io/blog/2015/sun.misc.unSafe/
