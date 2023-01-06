---
title: \[MYSQL] Access denied for user ~ (using password YES)  

categories: database
tags:
   - mysql
   - error

author_profile: true #작성자 프로필 출력여부
read_time: true # read_time을 출력할지 여부 1min read 같은것!

toc: true #Table Of Contents 목차 보여줌
toc_label: "My Table of Contents" # toc 이름 정의
toc_icon: "cog" # font Awesome아이콘으로 toc 아이콘 설정 
toc_sticky: true # 스크롤 내릴때 같이 내려가는 목차

last_modified_at: 2020-02-17T09:00:00 # 마지막 변경일

---

<!-- intro -->
{% include intro %}

# 해결 방법

mysql 의 데이터베이스 mysql 로 들어가서  User 테이블의 정보를 확인 해 본다.    
User 컬럼의 localhost 와 % 의 비밀 번호 정보가 다르게 입력 되어 있을 수 있다.  
  
다를 경우  
```sql
update user set Password=Password('[비밀 번호]') where User='[아이디]' and Host='%';  
  
commit;  
  
FLUSH PRIVILEGES;  
```
이런 식으로 재등록 해준다.  

update 하고 commit을 해주지 않으면 변경사항이 저장되지 않는다!
{: .notice--success}

{% capture flushprivileges %}
보통은 **INSERT, DELETE, UPDATE를 통해 사용자를 추가, 삭제, 권한 변경 등을 수행하였을 때 이 변경 사항을 반영하기 위하여 사용**한다. 이 떄 FLUSH PRIVILEGES는 **grant 테이블을 reload함으로서 변경 사항을 즉시 반영**하도록 한다.

그런데 만약 INSERT, DELETE, UPDATE와 같은 SQL문을 사용하지 않고 **바로 grant 명령어를 사용하여 작업하였다면 FLUSH PRIVILEGES를 실행할 필요가 없어진다.**

FLUSH PRIVILEGES는 굉장히 성능에 영향을 준다. 특히 **습관적으로 FLUSH PRIVILEGES를 사용**하는 경우가 있는데(모든 명령 이후에 사용하는 경우도 있다) 이는 **엄청난 부하**가 된다.
{% endcapture %}

{% include notice--info title="FLUSH PRIVILEGES" content=flushprivileges %}

단, localhost에서 접속하는 경우 **%**가 아닌 localhost로 Host 컬럼에 저장되어있어야 한다.  
user table을 확인해보고, 내가 현재 로컬에서 접속하는데 **%**로 저장되어있다면 위의 쿼리문을 응용해서  
**% → localhost**로 변경한 뒤에 접속하길 바란다.  


mysql를 잘 몰라서 나도 계속 접속이 안되었다.  
외부에서 접속이 필요한 경우 해당 사용자 아이디를 host name만 다르게 추가하면 됩니다.  
  


# Reference
  
* [정윤재의 정리노트](https://shonm.tistory.com/387) 
* [Code Legacy](https://legacy.tistory.com/114)
* [[삵 (sarc.io)](https://sarc.io/)](https://sarc.io/index.php/mariadb/355-mysql-flush-privileges)
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTYxNjI2NTc2NF19
-->