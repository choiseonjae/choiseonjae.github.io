---
title: "\\[JAVA] ConcurrentHashMap 이란? (+ 다른 유사한 자료구조)"
---

오류를 만났다.. 알고있던 오류였지만 이 메소드에서 발생할 수 있다는 점은 놀라웠다.

![error_picture](/assets/image/java/ConcurrentMap/ArrayIndexOutOfBoundsException.png)


정말 많은 양의 데이터를 처리해야 했다.   

조금이라도 성능을 올릴려고 발악해보았다.

여러 쓰레드를 동시에 수행시켰고, 스킵가능한 데이터는 스킵시키기 위해 List를 만들었다.  

먼저, 이슈가 발생한 이유는 여러 개의 thread가 하나의 list에 접근해 값을 추가하려고 할 때 발생했다.

`list.get()`도 아니고 `list.add()`에서 `ArrayIndexOutOfBoundsException` 오류라니 신기했다.

# 일반적인 자료구조
ArrayList를 쓴 이유는 처음에는 생각없이 list로 구현했지만(운영이 아니라 단순 배치여서 대충..) 오류가 발생하고나서야 

자료구조, 동시성에 대해서 정리해봤다.

## List

일단, 생각없이 짠 List로 구현했다면 조회할 때 하나씩 뒤지면서 조회하기 때문에 데이터가 쌓일 수록 지연이 늘어난다.

(성능을 높이려고 한 작업이 더 늦어질수도...)

![list_contains](/assets/image/java/ConcurrentMap/list_contains.png)

## Set

Set은 조회시에 map과 동일한 방법으로 조회한다. 생성 시점에 map을 생성하고, map을 set처럼 관리한다.


![hashSet_constructor](/assets/image/java/ConcurrentMap/HashSet_constructor.png)


![hash_contains](/assets/image/java/ConcurrentMap/set_containsKey.png)

## Map
map은 아래와 같이 key를 hash화 해서 찾고, 같은 hash라면 연결될 같은 Node들을 돌면서 찾는다.

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
Map 이다. 모든 메소드에 `synchroized`가 붙어있다. 위와 같이 느리다는 단점이 있다. 현재로서는 잘 안쓰이는 자료구조이다.
![hash_table_contain](/assets/image/java/ConcurrentMap/HashTable.png)
## ConcurrentHashMap
Hashtable 클래스의 단점을 보완하면서 Multi-Thread 환경에서 사용할 수 있도록 나온 클래스가 바로 `ConcurrentHashMap` 이다.
contains 코드를 보면 `synchroized`가 없다. 그래서 여러 쓰레드가 동시에 접근해서 병목현상이 없고, 빠르게 조회가 가능하다.
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
만약, 16만개의 데이터가 들어간다면 한 버킷당 만개의 데이터가 들어가고 최대 10000번의 `equals()`가 있을 수 있다. 그래서 capacity를 처음부터 잘 조절하는게 중요하다.

물론, load_factor(==데이터 수 / capacity *기본값 0.75*)에 의해서 너무 많은 데이터가 저장되면 자동으로 버킷의 크기를 늘리게 된다. (약 2배씩)
근데, 이때 capacity가 변경되게 되면 index도 달라지고, 다시 넣어야하는 작업이 있기 때문에 엄청난 부하가 발생하니 처음부터 크기를 알 수 있다면 잘 설정해서 넣어주는 것이 좋다.

그렇다고 처음부터 capacity를 왕창 크게 잡는 것도 능사는 아닌게, 빈 공간을 많이 만들테니 메모리가 낭비되게 된다.
게다가 key에 의한 index 접근이 아니라 Iteration(객체의 모든 데이터를 조회해봐야 하는) 상황에서는 성능이 저하된다.
Iteration의 부하는 (capacity + 저장된 데이터 수)와 비례하기 때문이다.
ex) 16의 capacity는 전체 조회 시 16번을 도는데 (그 안의 데이터는 차치하더라도), 16만개의 capacity는 16만번을 조회해야하기 때문이다.

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
