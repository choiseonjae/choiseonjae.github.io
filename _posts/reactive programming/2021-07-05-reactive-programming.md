---
title: \[Reative Programming] Reactive Programming란? Reactive Programming 간단하게 정리하기

categories: reactive programming
   
tags:
   - reactive programming
   - reactive
   - callback
   - observer
   - observer design pattern
   - observer pattern
   - observation
   - observe
   - 리엑티브
   - 리엑티브 프로그래밍
   - 콜백
   - 옵저버
   - 옵저버 디자인 패턴
   - 옵저버 패턴

last_modified_at: 2021-07-05 

---

Reactive Programming을 한 줄로 설명하면 다음과 같습니다. 
> Reactive programming is programming with asynchronous data streams.

즉, 비동기적 데이터 흐름을 처리하는 프로그래밍입니다. Reactive Programming의 **핵심은 모든 것을 비동기적인 데이터의 Stream으로 간주**하고, **Observer 디자인 패턴을 활용해서 이러한 비동기 이벤트를 처리**하는 것에 있습니다.

설명하기에 앞써 비동기를 사용하는 가장 큰 이유는 **속도**입니다. 

![Reactive%20Programming%E1%84%85%E1%85%A1%E1%86%AB%20Reactive%20Programming%20%E1%84%80%E1%85%A1%E1%86%AB%E1%84%83%E1%85%A1%2068ea60523b354987bece35b05a2ec385/Untitled%201.png](https://github.com/choiseonjae/choiseonjae.github.io/blob/master/assets/reactive%20programming/synchronouse%20vs%20asynchronous.png?raw=true)


# 실시간화

개발자 입장에서는 비동기적인 데이터 처리라고 볼 수 있었습니다. 사용자 입장에서 살펴보면, Reactive Programming이란 **실시간으로 반응을 하는 프로그래밍**을 말합니다. 예를 들면, 네이버 검색창에 단어를 하나씩 입력하는 그 순간에 관련 검색어들이 자동 완성으로 바로바로 제시되는 것이나, 페이스북 글을 보고 있을 때 다른 사람이 '좋아요' 버튼을 누르면 새로고침이 없어도 실시간으로 좋아요 수가 올라가는 것을 말합니다.

![Reactive%20Programming%E1%84%85%E1%85%A1%E1%86%AB%20Reactive%20Programming%20%E1%84%80%E1%85%A1%E1%86%AB%E1%84%83%E1%85%A1%2068ea60523b354987bece35b05a2ec385/Untitled.png](https://github.com/choiseonjae/choiseonjae.github.io/blob/master/assets/reactive%20programming/naver%20example.png?raw=true)

# Asynchronous와 Observer

처음에 언급했듯, **Reactive Programming의 핵심은 비동기 이벤트와 Observer 디자인 패턴**입니다.  

비동기 이벤트란 프로그램에서 실행흐름이 하나씩 진행되는 동기적인 흐름이 아닌, 동시 다발적으로 작업들이 수행되는 이벤트를 말합니다. 유저의 입장에서 대기하는 시간이 없도록 (= 유저의 인터페이스를 방해하지 않도록) 뒷단에서 데이터를 가져오는 작업이 비동기적인 작업입니다. 

Reactive Programming에서 유저가 입력을 할 때마다 즉각적으로 반응하기 위해서는 프로그램은 지속적으로 값을 관찰(Observe)하고, 값에 변화를 감지하여 연산을 수행해야 합니다. 이러한 관찰 패턴을 Observer Pattern이라고 하며, 이는 비동기 이벤트를 처리하는 Reactive Programming의 근간이 됩니다.

관찰을 한다는 것은 지속적으로 listen 하고 있다는 말과 같습니다. Reactive Programming 상에서 해당 스트림에 가입(subscribe)한 것이라고 표현하기도 합니다. (관찰하고 있는 데이터의 스트림을 구독하여 데이터를 대기하고 있는 모습이라서 그런 것 같습니다.) 이는 라디오를 듣기 위해 주파수를 맞추고 기다리고 있거나, 웹 사이트의 소식을 알기위해 이메일 스트림에 가입하는 것과 비슷한 개념입니다.

# Callback 보단 Observer

비동기 이벤트를 처리하는 방법으로 Observation 방식도 있지만, Callback을 사용할 수도 있습니다. 결론적으로 Callback은 상당히 복잡한 구조를 가질 수도 있기 때문에 Observation이 비동기 작업을 처리하는데 더 효율적일 수 있습니다. 

Callback 방식으로 비동기 작업을 알아보면, 다음과 같습니다.

```jsx
function test(e){
	value = "a"
	e(value)
}

test(function(value){
	console.log(value)
})
```

위처럼 특정 함수의 메소드로 함수를 넣어서 특정 작업 후 진행되는 함수를 넣어줄 수 있습니다. 

Main Thread에서 Thread가 나와서 비동기 처리를 할 때, 비동기 처리를 하고 완료 된 뒤 Main Thread에서 받아서 일을 진행한다고 하지만 사실상 코드상에서 중간에 어떻게 처리를 할 수가 없습니다. callback 함수를 통해서 Main Thread의 함수를 넣어줘서 별도의 Thread에서 얻은 결과값을 이어서 처리할 수 있는 것입니다.

![Reactive%20Programming%E1%84%85%E1%85%A1%E1%86%AB%20Reactive%20Programming%20%E1%84%80%E1%85%A1%E1%86%AB%E1%84%83%E1%85%A1%2068ea60523b354987bece35b05a2ec385/Untitled%202.png](https://github.com/choiseonjae/choiseonjae.github.io/blob/master/assets/reactive%20programming/Main%20Thread%20example.png?raw=true)

보통 위 이미지 처럼 표현이되어 비동기 처리 후 알람 받으면 Main Thread가 그 처리에 대한 작업을 하는 것처럼 표현합니다. 하지만, 사실상 Main Thread가 알림을 받고 다시 처리를 하기는 어렵습니다. 코드상으로는 Main Thread의 함수를 Callback으로 넘겨줘서 별도의 Thread에서 자신의 작업을 한 뒤 Main Thread에서 받은 함수로 작업하는 것을 Main Thread에 알려준다. 라고 표현하는 걸로 보입니다.

# 출처

- [Reactive Programming과 Rx](https://m.blog.naver.com/jdub7138/220983291803)
- [Callback 함수](https://velog.io/@hyksmine/call-back..-i4k1xple94)
- [콜백(Callback) 패턴을 사용한 비동기 방식의 원리와 사용법](https://codevang.tistory.com/187)
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTMxMzkyMjQ3NiwtMTYxNTY2NDE5MF19
-->