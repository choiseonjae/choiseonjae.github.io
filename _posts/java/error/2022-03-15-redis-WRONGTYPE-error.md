---
title: \[JAVA] redis WRONGTYPE Operation against a key holding the wrong kind of value 오류 원인 및 해결 방법

header:
  teaser: https://media.vlpt.us/images/gingaminga/post/f8b9b993-6264-4313-a1a5-d653fede9ac4/Redis-Logo.wine.png

categories: java

tags:
   - error
   - redis
   - java

last_modified_at: 2022-03-15

---

레디스에는 5가지의 타입이 존재한다. 한번 데이터베이스에 타입이 결정된 상태에서 해당 타입과 상관없는 명령을 수행하려고 할 때 위와 같은 에러 메세지가 출력이 된다.

나같은 경우, 기존에는 String 상태로 저장한 value를 hash 타입으로 바꾸면서 오류가 발생했다.

> 간단한 해결방법은 **기존 키를 지우고 처음부터 다시 시도**하면 된다.  

다만, 실제 운영환경에서는 사용자의 값을 중간에 제거한다는 건 좋은 방법이 아니다.  
어떤 사이드 이펙트를 발생시킬 지 모르기 때문에 나같은 경우에는 지우지 않고 **저장되는 키를 새롭게 바꿔서 추가하는 방법**을 선택했다.

각각의 key에 대해서 duel-write를 진행하고, 동시에 fallback 을 설정하여 기존에 저장된 레디스 키와 새로운 키 둘 다 읽을 수 있는 환경을 먼저 전체 배포를 진행한다.  
그 다음 duel-write와 fallback 코드에서 기존 키에 대한 로직들을 제거하면 서버 여러 대가 오류 없이 배포 진행할 수 있다.

## 출처
- https://m.blog.naver.com/newwodudrj/221376339003
