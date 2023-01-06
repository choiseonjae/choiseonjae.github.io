---
title: \[Spring Boot\] 스프링 부트가 실행 될 때, 자동으로 데이터 삽입하기 + 데이터 삽입 안되는 오류 해결 (initialize database load 오류)

categories: spring-boot
  
tags:
   - database
   - sql
   - database load error
   

last_modified_at: 2021-05-29 18:31:32.71 

---


스프링 부트가 뜰 때, h2 등을 사용하는 경우에 데이터 베이스에 대한 테이블과 그 데이터를 자동으로 삽입하고 싶을 때가 있다. 매번 /h2-console을 들어가서 삽입할 수는 없으니까! ㅎㅎ

## sql 파일 위치

![](https://camo.githubusercontent.com/e0361eb25c04c9eb964b98db7d41ec248f9e239d07003a528afb624e8ec7d86f/68747470733a2f2f692e6962622e636f2f345a4d6d46737a2f556e7469746c65642e706e67)


당연히 방법이 있을 것 같아서 찾아보던 도중 classpath위치에 (resources 폴더) data.sql 파일을 생성하고 사용하고자 하는 데이터 베이스 시퀀스에 맞춰서 명령어를 넣어 주면 된다. 

create table 같은 ddl은 schema.sql 파일에 넣고, dml 같은 데이터 삽입은 data.sql 파일에 넣어주면 된다.
![](https://camo.githubusercontent.com/3282bed99d951b24098ca66c5704596632a79e9390dfaa9e1539aff55c338888/68747470733a2f2f692e6962622e636f2f594479774a68762f696d6167652e706e67)
![](https://camo.githubusercontent.com/3dea867cc45418771e17bc79c180f115a5ea71dfb11fd85483619a14a045e06a/68747470733a2f2f692e6962622e636f2f4a334a746664732f696d6167652e706e67)

### 쿼리를 모르겠다면 
만약에 시퀀스를 잘 모르겠다면, jpa의 `spring.jap.show-sql: true`로 application.yml에 넣어주면 console에서 날리는 쿼리를 볼 수 있다. 해당 쿼리를 보고 참고해서 원하는 쿼리를 작성하자! ㅎㅎ
![](https://camo.githubusercontent.com/5647287dccd992c7e5592dcf2387a42959556ade49a91a47083c76bfde55befc/68747470733a2f2f692e6962622e636f2f5a4c687438427a2f696d6167652e706e67)

## 주의사항

![](https://camo.githubusercontent.com/f048048d6d5f90680c4e07193a0288a7038de2e129b17019dfc6ef1ad8cc8c28/68747470733a2f2f692e6962622e636f2f567150533479662f556e7469746c65642d312e706e67)

create table 등은 가능한데, data.sql에서의 쿼리가 먹히지 않아서 찾아보니 `ddl-auto: none, initialize: true` 로 설정해야 읽힌다. 추측하건데, ddl-auto는 자동으로 생성하지 않기 위해서 none을 하는 것 같고, initialize를 true로 해야 .sql 파일을 읽는 것 같다!

이상!!
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTYxMjMyODkwNywtMjg1NDIxMTk1LDMyOT
E5OTEwMSwtMzAzNDI3OTYwXX0=
-->
