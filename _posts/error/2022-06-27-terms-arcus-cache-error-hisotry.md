---
title: \[삽질] 약관 테이블 Entity 아커스(캐시)에 적용까지의 오류들 정리

header:
  teaser: https://www.jam2in.com/img/feature-icons/arcus_operator_w.png

categories: error

tags:
- arcus
- error
- jpa
- entity

last_modified_at: 2022-06-27

---

내가 개발을 진행했던 프로젝트 중에서 가장 버그가 많았고, 정상적으로 배포하기까지 품이 많이 들었던 작업이였던 거 같다.  
사실, 정리하고 보니까 별 것 쉬운 것 같지만 오류를 만났을 때 당시는 되게 해결하기 위해 많이 검색하고 고민했었다. 동일한 실수를 하지 않기 위해서 오류를 시간 순서대로 정리해본다.
# 설명에 들어가기 앞서 알아야 할 것들
1. 내가 적용하고자 하는 약관 테이블의 구조는 `약관(N)` ⇆ `약관 그룹(N)` ⇆ `약관 루트(N)` 의 구조를 가지고 각각 N:N 관계이다.
2. N:N을 1:N 연관관계로 만들기 위해서 각 테이블 사이에 maps table을 만들었다.
   1. `약관(1)` ⇆ `약관 그룹 약관 Maps(N)` ⇆ `약관 그룹(1)` ⇆ `약관 루트 & 약관 그룹 Maps(N)` ⇆ `약관 루트(1)`
   2. `약관 그룹` entity에는 `약관 그룹 약관 Maps` 과 `약관 루트 & 약관 그룹 Maps`가 존재한다.
3. 아커스(캐시)에 적재할 때는 객체를 json string으로 변환한다.
4. 아커스(캐시)는 cache miss 할 때 DB에서 조회한 뒤 캐시에 적재한다.
5. 아커스(캐시)는 transaction이 끝나고 캐시에 적재한다.

# 개발 내용: 약관 Repository(DB layer) 앞에 아커스 cahce layer를 둬서 DB 전에 캐시를 먼저 사용하도록 한다.
## 1. DB layer에 아커스 캐시 layer 적용
map 형태로 자주 사용되는 key를 기준으로 캐시에 넣어서 DB가기전에 조회함.  
없으면, DB 조회 후 cache에 적재
## 2. 캐시 적용 후 테스트 코드 작성
양방향 연관 관계(termsGroupTermsMap) 때문에 테스트 코드에서 lazy로딩 오류가 발생함.  

FetchType을 `EAGER`로 추가하던가, Test 코드에 `@Transactional` 어노테이션을 붙이면 됨.  
{: .notice--info}

{% capture content2 %}
persist context의 생명주기가 트랜잭션이 유효할 때까지 이기 때문에 lazy loading 시점에 생명주기 연결이 끊겨서 데이터를 가져올 수가 없기 때문에  @Test 코드 쪽에서 cachedRepository와 repository 테스트를 진행하게 되면, Persist의 생명주기가 트랜잭션이 유효할 때까지 이기 때문에 assert로 비교하려고 하면 오류가 뜬다.

왜냐? lazy loading 하기로 한 Entity가 그대로 연결이 끊겨서 데이터를 가져올 수 없기 때문이다.

오류 명: failed to lazily initialize a collection of role: ~ could not initialize proxy - no Session 에러    
출처: [https://velog.io/@youns1121/failed-to-lazily-initialize-a-collection-of-role-could-not-initialize-proxy-no-Session](https://velog.io/@youns1121/failed-to-lazily-initialize-a-collection-of-role-could-not-initialize-proxy-no-Session)
{% endcapture %}

{% include notice--green title="영속성 컨텍스트의 생명주기" content=content2 %}

## 3. 모든 Entitiy 연관관계를 fetchType EAGER로 변경   
`@Transactional`과 `FetchType.EAGER` 중에서 cache 적재해서 효율 올릴려면 미리 조회해서 가지고 있는게 좋다고 생각이 되어, 모든 설정을 EAGER로 변경.
{% capture content3 %}
cache에 적재하는 시점이 Transaction이 종료되고 적재하기 때문에 `@Transactional` 어노테이션을 붙이면 정상적으로 적재하는 지 테스트를 할 수 가 없다.
{% endcapture %}

{% include notice--green title="@Transactional 붙이면 안되는 이유" content=content3 %}


## 4. 약관 그룹에 있는 약관 / 약관 루트와의 연관관계 때문에 EAGER로 설정을 불가
{% capture content4 %}
`약관 ⇆ 약관 그룹` Maps 하나 `약관 그룹 ⇆ 약관 루트` 사이에 하나씩 만들어서 약관, 약관 그룹, 약관 루트는 각각 Maps 들과 oneToMany 연관관계를 맺었다.

근데, Terms Group 의 경우 `약관과의 Maps` `약관 루트 간의 Maps` 가 존재한다.  
- **하나의 Entity에 다중 Eager을 사용 시 Hibernate의 과부하를 고려한 것인지 오류가 발생해서 하나는 Lazy로 변경**해야 한다고 하더라.

오류 명: Caused by: org.hibernate.loader.MultipleBagFetchException: cannot simultaneously fetch multiple bags    
출처: [https://eclipse4j.tistory.com/215](https://eclipse4j.tistory.com/215)
{% endcapture %}

{% include notice--green title="EAGER로 설정이 불가능한 이유" content=content4 %}

## 5. fetchType LAZY로 원복 및 Test 코드에 @Transactional을 붙이는 방식으로 변경했다.  
제대로 적재되는 지 두번 호출해서 아커스 hit 되서 가져오는 지는 테스트 코드에서 알 수는 없지만 hit하지 않았을 때 DB에서 정상적으로 가져오는 것까지만 테스트 진행.

## 6. 양방향 연관관계 entity 때문에 [양방향 순환참조 문제](https://dev-coco.tistory.com/133)(**Circular reference*) 발생
{% capture content5 %}
1. **Circular reference*는 **참조하는 대상이 서로 물려 있어 참조할 수 없게 되는 현상
2. 왜 이전에는 발생 안 했냐?
  - 그냥 entity 조회에서 사용하는 경우에는 문제가 없었을 수 있음.
  - **Jackson 라이브러리를 이용하는 경우, Jackson은 entity의 getter를 호출하고, *직렬화를 이용해 JSON 형태로 객체를 변화시키고 전달하는데 getter를 호출하는 과정에서부터 순환 참조가 계속 발생해 stackoverflow가 발생**  
  **직렬화* 객체 → json string → 바이트 단위로 변환하여 네트워크 송수신하기 위함.
오류 명: Infinite recursion (StackOverflowError) (through reference chain:  
출처: [https://pasudo123.tistory.com/350](https://pasudo123.tistory.com/350)
{% endcapture %}

{% include notice--green title="순환참조 문제" content=content5 %}

## 7. 양방향으로 매핑되어있는 객체에 대해서 무한루프로 서로를 참조 문제가 발생하고 스택오버플로우
아커스 적재하는 게 문제라긴보단 아커스 적재 시 ObjectMapper가 Json 형태로 객체를 변환 시키는 과정에서 양방향으로 매핑되어있는 객체에 대해서 JSON 타입에 대한 무한루프문제가 발생하고 스택오버플로우가 발생

{% capture content %}

1. jackson 1.6 버전
   - @JsonManageReference - 참조하는 곳에 붙인다 (1:N에서 1에 해당되는 곳)
   - @JsonBackReference - 참조되는 곳에 붙인다 (1N에서 N에 해당되는 곳)

2. jackson 2.0 버전
   - 해당하는 Entity Class 위에 `@JsonIdentityInfo(generator = ObjectIdGenerators.IntSequenceGenerator.class)` 를 추가한다.
   - 단, 2번 방법으로 진행했을 경우, json에 @id 필드가 생긴다.

3. `@JsonIgnore`을 해당 필드에 붙이면 null로 들어가면서 아예 제외된다.

4. entity 객체를 getter로 조회하지 못하게 다른 class 생성 후 옮겨 담기.    
   주원인은 '양방향 매핑'이기도 하지만, 더 정확하게는 Entity 자체 변환하고자 할 때 발생한 것이기 때문에 필요한 필드만 담은 class를 생성해서 사용한다.

5. 매핑 재설정    
   양방향 매핑이 꼭 필요한지 다시 한번 생각해볼 필요가 있다. 만약 양쪽에서 접근할 필요가 없다면 단방향 매핑을 해줘서 자연스레 순환 참조 문제를 해결

{% endcapture %}

{% capture reference_link_1 %}
> 출처: [양방향 관계에서 infinite recursion 해결법](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=rorean&logNo=221593255071)  
출처: [양방향 순환참조 문제 및 해결방법](https://dev-coco.tistory.com/133)  

{% endcapture %}

{% include notice--info title="양방향 순환참조 문제를 해결하기 위한 방법" content=content %}  

{% include toggle title="양방향 순환참조 해결방법 출처" content=reference_link_1 %}  

## 8. 아커스가 정상적으로 조회되지 않음.
위처럼 양방향 순환참조 문제를 해결 5의 b로 설정한 뒤 문제는 **@EqualsAndHashCode로 실제 terms가 같은 지 조회할 때 객체 주소가 아닌 객체 내용을 보고 같으면 동일한 객체라고 판단하는데 @EqualsAndHashCode에 Maps를 제외시키지 않아서 TermsGroupTermsMap도 내용 비교를 했음.  정상적으로 조회**가 되지 않았다. Maps에 @EqualsAndHashCode가 안 붙어있던거는 아니고 비교 시 null 경우가 있어서 정상적으로 비교되지 않았던거 같음. @EqualsAndHashCode 안의 하위 객체도 전부 @EqualsAndHashCode가 붙어있어야 정상적으로 객체주소가 아닌 필드 값으로 비교를 함.
   1. @EqualsAndHashCode 에서 exclude로 TermsGroupTermMaps 제외 시켰다.  

## 9. 양방향 연관관계가 연결되어 있어 약관 하나에 terms root(약관 루트)까지 조회하여 terms root 밑에 있는 모든 약관을 약관 하나마다 다 중복되게 저장 오류 발생
{% capture content6 %}
1. e.g) terms root a에 약관 1,2,3,4,5 포함 시 약관 1에 1,2,3,4,5 중복해서 저장. 약관 2에 1,2,3,4,5 중복해서 저장 등등 .. 약관 하나도 content데이터가 어마어마한데 이게 중복으로 다 들어갔다.    
2. CPU 사용량이 어마어마게 늘어남    
{% endcapture %}

{% include notice--green title="모든 약관이 중복해서 저장됨" content=content6 %}

## 10. 결론적으로, 위에서 5-b가 아닌 5-e의 방법으로 매핑을 단방향으로 변경하니 정상적으로 해결할 수 있었다.