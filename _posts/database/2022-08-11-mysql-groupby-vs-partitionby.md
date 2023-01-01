---
title: \[DB] Group By와 Partition By의 유사점 및 차이점 

header:
  teaser: https://cdn.imweb.me/thumbnail/20200915/a0c073d00397a.png

categories: database
   
tags:
   - database
   - group by
   - partition by
   - over
   - order by
   - mysql

last_modified_at: 2023-01-01 01:33:00 

---

계정 / 약관 관련 업무를 하다보면 다른 서비스팀으로부터 문의 및 요청이 들어온다. 이번에 복잡하게 꼬여서 데이터 추출을 하기 까다로운 경우가 생겼는데 이 때, group by를 통해 데이터를 추출할 수 있었다. group by를 통해 데이터를 추출할 때 잃어버리는 데이터가 많아 여러번 sub query를 만들어 조회를 했는데, partition by를 이용하니 정말 간단하게 추출할 수 있었다.

비슷하지만 다른 `Group By vs Partition By` 정리를 해본다.

들어가기전에 앞써서 간단하게 요약하자면 둘의 유사점과 차이점은 다음과 같다.

- 유사점은 데이터를 그룹화 한다는 점에서 유사하다.
- 차이점은 `Group By`는 그룹화하는 과정에서 상세 데이터를 상실하게 되고, `Partition By`는 상세데이터가 유지된다는 점이다.

# Group By vs Partition By

## Group By

동일한 값 여러 개를 가지고 있는 열의 이름 혹은 이름들을 Group By 절에 적어줌으로써 그 열(들)이 가지고 있는 값이 동일한 경우끼리 그룹을 지게된다. 이때, 집계 함수를 통해 값을 합치는 과정에서 동일한 값을 하나의 값으로 합치고 그 행들의 값을 계산한다. 집계 함수를 합치는 과정에서 기존의 행들은 사라지게 된다. 집계 함수를 통한 값과 Group By 절에 있던 column들은 볼 수 있어도 기존의 나머지 정보는 볼 수 없다.

물론, 서브 쿼리를 통해 해결할 수 있지만 그렇게 하다보면 쿼리문 자체가 길어지는 단점이 있다.
집계 합수에서 자주 사용되는 함수들은 다음과 같다.

| 집계 함수 | 의미 |
| --- | --- |
| COUNT | 같은 열(들)의 개수를 센다. |
| SUM | 같은 열(들)의 값의 합 |
| AVG | 같은 열(들)의 값의 평균 |
| MIN | 같은 열(들) 중 최소 값 |
| MAX | 같은 열(들) 중 최대 값 |

```sql
SELECT status_cd, count(*) // status_cd 값과 그 집계함수의 결과만 알수 있다.
FROM user
GROUP BY status_cd // status_cd 기준으로 중복없게 row가 생성되고, 그 값들의 counting 값이 출력된다.
having count(*) > 10; // counting 된 값이 10 이상인 것만 출력한다.
```

## Partition By

`Partition By`는 `Group By`보다 복잡한 분석을 할 때 사용하는 SQL 구문이다.

`Partition By`는 `OVER`절과 윈도우 함수와 함께 사용된다. 위에서도 말했지만, `Partition By`는 기존 행의 세세한 정보들이 사라지지 않고 그대로 유지된다. 또한, 모든 집계 함수를 윈도우 함수를 통해 동일하게 사용할 수 있다.

{% capture content %}
💡 집계 함수와 윈도우 함수의 차이점

집계 함수는 여러 행의 수치를 단 1개의 수치로 반환한다.
윈도우 함수는 각 행마다 1개의 값을 반환한다. 윈도우 함수는 기존의 데이터에는 아무런 변화를 주지 않은 상태에서, 새로운 열에 반환할 값을 계산하고자 집계 함수를 함께 사용할 수는 있다. 집계 함수를 써도 되고 그냥 윈도우 함수를 써도 된다.

{% endcapture %}

{% include notice--info title="" content=content %}

```sql
select R.id, R.name, R.birthday
from (
	select *, row_number() over (partition by name order by id) as rownumber from user
) R
where R.rownumber = 1; // name가 같은 그룹 id 기준으로 행번호를 붙인뒤에 행번호가 1번을 추출

------------------------------------------------------------------------

select R.id, R.name, R.birthday
from (
	select *, row_number() over (order by id) as rownumber from user
) R
where R.rownumber = 1; // 전체 그룹을 id 기준으로 행번호를 붙인뒤에 행번호가 1번을 추출

------------------------------------------------------------------------

select R.id, R.name, R.birthday
from (
	select *, rank() over (order by score) as rownumber from user
) R
where R.rownumber = 1; // 전체 그룹을 score(점수) 기준으로 순위를 매긴뒤 1번을 추출. 만약, 1번이 두명이라면 두명이 추출되고, 다음 등수는 3등으로 표기된다.

------------------------------------------------------------------------

select R.id, R.name, R.birthday
from (
	select *, dense_rank() over (order by score) as rownumber from user
) R
where R.rownumber = 1; // 전체 그룹을 score(점수) 기준으로 순위를 매긴뒤 1번을 추출. 만약, 1번이 두명이라면 두명이 추출되고, 다음 등수는 2등으로 표기된다.

------------------------------------------------------------------------

select R.id, R.name, R.birthday
from (
	select *, lead(name) over (order by birthday) as next_person_birthday from user
) R
// 전체그룹에서 birthday 기준으로 정렬되었을 때 현재 열의 다음열의 name 값을 next_person_birthday에 넣는다.

------------------------------------------------------------------------

select R.id, R.name, R.birthday
from (
	select *, lag(name) over (order by birthday) as next_person_birthday from user
) R
// 전체그룹에서 birthday 기준으로 정렬되었을 때 현재 열의 이전 열의 name 값을 next_person_birthday에 넣는다.

------------------------------------------------------------------------

select R.id, R.name, R.birthday
from (
	select *, sum(score) over (order by birthday) as score_total from user
) R
// 전체그룹에서 birthday 기준으로 정렬되었을 때 누적으로 score 점수의 합을 넣는다. (특징은 누적이다)

------------------------------------------------------------------------

select R.id, R.name, R.birthday
from (
	select *, count(score) over (order by birthday) as score_total from user
) R
// 전체그룹에서 birthday 기준으로 정렬되었을 때 누적으로 score 점수의 갯수를 구한다. (특징은 누적이다)
```

| 집계 함수 | 의미                                                                                                                                                                                              |
| --- |-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| row_number() | over(order by [column])이 고정으로 나온다. **order by 기준으로 행의 번호**를 붙인다. over(partition by [column] order by [column]) 처럼 partition by를 붙이면 column에서 같은 값끼리 그룹핑 한 뒤에 행 번호를 붙인다.                             |
| rank | over(order by [column]) 기준으로 순위를 매긴다. **1등이 두명 나오면 다음 등수는 3**등부터 시작이다. over(partition by [column] order by [column]) 처럼 partition by를 붙이면 column에서 같은 값끼리 그룹핑 한 뒤에 순위를 붙인다.                         |
| dense_rank | over(order by [column]) 기준으로 순위를 매긴다. **1등이 두명 나오면 다음 등수는 2**등부터 시작이다. over(partition by [column] order by [column]) 처럼 partition by를 붙이면 column에서 같은 값끼리 그룹핑 한 뒤에 순위를 붙인다.                         |
| lead | lead(column) over (order by [column]) column으로 정렬되었을 때, **정렬 순서로 다음 열(다음 값)**의 값을 가져온다.  over(partition by [column] order by [column]) 처럼 partition by를 붙이면 column에서 같은 값끼리 그룹핑 한 데이터 내에서 가져온다.     |
| lag | lead(column) over (order by [column]) column으로 정렬되었을 때, **정렬 순서로 앞의 열(이전 값)**의 값을 가져온다.  over(partition by [column] order by [column]) 처럼 partition by를 붙이면 column에서 같은 값끼리 그룹핑 한 데이터 내에서 가져온다. |
| sum | sum(column) over (order by [column]) column으로 정렬되었을 때, sum 안에 있는 column 값이 **누적**으로 추출된다.                                                                                                       |
| count | count(column) over (order by [column]) column으로 정렬되었을 때, count 안에 있는 column **값이** 누적으로 counting된다.                                                                                             |

# 정리

group by를 사용한다고 partition by에서 추출할 수 있는 값을 못 얻는건 아니다. 즉, 둘다 원하는 값을 추출할 수 있는 건 동일하다. 하지만, group by의 경우, 기존 데이터를  잃어버리면서 새로운 값을 구하기 때문에 원하는 값을 구하기 위해서 서브 쿼리를 길게 작성하게 될 수 있다.

1. 유사점은 열들의 값을 그룹핑하여 결과물을 가져온다.
2. 차이점은 기존 값을 잃어버리지 않고 구하고 싶으면, partition by를 사용하면 된다.
3. partition by는 윈도우 함수 + 집계 함수를 사용하지만, group by는 집계 함수만 사용한다.

## 출처
- [GROUP BY vs. PARTITION BY: 유사점과 차이점](https://kimsyoung.tistory.com/entry/GROUP-BY-vs-PARTITION-BY-%EC%9C%A0%EC%82%AC%EC%A0%90%EA%B3%BC-%EC%B0%A8%EC%9D%B4%EC%A0%90)
