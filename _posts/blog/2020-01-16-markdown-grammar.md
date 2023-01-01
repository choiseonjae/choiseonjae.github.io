---
title: markdown(마크다운)블로그에서 활용할 수 있는 문법 정리

categories: blog
tags:
   - markdown
   - kramdown
   - jekyll
   
author_profile: true <!-- 작성자 프로필 출력여부 -->
read_time: true <!-- read_time을 출력할지 여부 1min read 같은것! -->

toc: true
toc_label: 목차
toc_icon: "cog"
toc_sticky: true 

last_modified_at: 2020-03-26T13:29:00

---

해당 포스팅은 지속적으로 업데이트 될 수 있습니다.
{: .notice--success}

# 들어가며
본 포스트는 실제 블로그를 운영하면서 사용할 혹은 사용했던 문법들을 정리하여 나중에 쉽게 참고하기 위해서 작성한다.

# 기본적인 마크다운 문법
## 헤더

# # H1
{:.no_toc}

## ## H2
{:.no_toc}

### ### H3
{:.no_toc}

#### #### H4
{:.no_toc}

##### ##### H5
{:.no_toc}

###### ###### H6
{:.no_toc}

헤더의 종류는 1 ~ 6까지 존재하며, **순서대로 크기가 작아**진다.
**1과 2**만 `수평선`이 그려지고, **3 ~ 6**까지는 `수평선`이 없다.
{: .notice--info}

## 인용

> 내용
>> 내용
>>> 내용
>>>> 내용

``` markdown
> 내용
>> 내용
>>> 내용
>>>> 내용
```

인용 문구를 사용할 때 `>`를 붙인다. <br>
`>` 개수를 늘리면 인용 문구 안에서 또 인용 문구를 계속해서 사용할 수 있다.
{: .notice--info}

## 리스트

- 항목 1
+ 항목 2
	* 요소 1
	- 요소 2
* 항목 3
	* 요소 1
	* 요소 2
		* 요소 1
			1.	 내용 1
			2.	 내용 2
			3.	 내용 3

```markdown
- 항목 1
+ 항목 2
	* 요소 1
	- 요소 2
* 항목 3
	* 요소 1
	* 요소 2
		* 요소 1
			1.	 내용 1
			2.	 내용 2
			3.	 내용 3
```

{% capture list %}
`-`,  `*` `+` 중 어떤 것을 사용해도 동일하다.

`-`,  `*` `+`는 **원, 빈 원, 네모**... 순으로 늘어난다.

항목 밑에 요소를 추가하고 싶으면 같은 라인이 아닌 탭(tap)을 한번 한 뒤에 작성하면 된다.

숫자로 리스트를 만들고 싶으면 탭한뒤 `1.`을 하면 된다.
{% endcapture %}

{% include notice--info title="설명" content=list %}


## 코드 강조
``` python
def hello():
	print("hello world")
```

## 강조

****Bold****

__**Bold2**__

~~취소선~~

__기울임__

*_기울임2_*

## 링크

`<http://facebook.com/>`  [http://facebook.com/](https://facebook.com/)

`[googlelink](https://google.com)`  [googlelink](https://google.com/)

## 이미지

### 일반적으로 이미지를 가져오는 방법

`![alt text](/assets/img/fromis.PNG)`![alt text](https://imreplay.com/assets/img/fromis.PNG)

###  내 Github에 사진 저장 후 가져오는 방법

 1. 나의 repository로 이동한다.
![enter image description here](https://user-images.githubusercontent.com/49507736/72503445-11de3a00-387f-11ea-8031-1b700746fc61.png)
 2. 상단의 Issues를 클릭한다.
![enter image description here](https://user-images.githubusercontent.com/49507736/72503446-1276d080-387f-11ea-9989-3ed3af8446fd.png)
 3.  New Issue를 클릭한다.
![enter image description here](https://user-images.githubusercontent.com/49507736/72503447-1276d080-387f-11ea-9589-6f8e64f62703.png)
 4. 자신이 구별할 제목을 넣고, 사진들을 드래그 해서 넣은 뒤, `Submit new issue`를 클릭한다.
![enter image description here](https://user-images.githubusercontent.com/49507736/72503448-1276d080-387f-11ea-9895-4b4519b68a2e.png)
 5. 내가 쓴 issue에 들어간 뒤 자신이 사용하고 싶은 사진 위치에 **우 클릭 후 링크 주소 복사**를 클린다.
![enter image description here](https://user-images.githubusercontent.com/49507736/72503443-11de3a00-387f-11ea-92d4-eb39c06a67d5.png)
 6.  자신이 원하는 곳에 ![일반적으로 이미지 가져오는 방법](###)과 동일하게 사용한다.


## 표

``` markdown
| Header1 | Header2 | Header3 |
|:--------|:-------:|--------:|
| cell1   | cell2   | cell3   |
| cell4   | cell5   | cell6   |
```

| Header1 | Header2 | Header3 |
|:--------|:-------:|--------:|
| cell1   | cell2   | cell3   |
| cell4   | cell5   | cell6   |



# 테마에 적용될 수 있는 문법
## 버튼
``` markdown
[Default Button Text](#link){: .btn}
[Primary Button Text](#link){: .btn .btn--primary}
[Success Button Text](#link){: .btn .btn--success}
[Warning Button Text](#link){: .btn .btn--warning}
[Danger Button Text](#link){: .btn .btn--danger}
[Info Button Text](#link){: .btn .btn--info}
[Inverse Button](#link){: .btn .btn--inverse}
[Light Outline Button](#link){: .btn .btn--light-outline}
``` 
[Default Button Text](#link){: .btn}
[Primary Button Text](#link){: .btn .btn--primary}
[Success Button Text](#link){: .btn .btn--success}
[Warning Button Text](#link){: .btn .btn--warning}
[Danger Button Text](#link){: .btn .btn--danger}
[Info Button Text](#link){: .btn .btn--info}
[Inverse Button](#link){: .btn .btn--inverse}
[Light Outline Button](#link){: .btn .btn--light-outline}
```
[X-Large Button](#link){: .btn .btn--primary .btn--x-large}
[Large Button](#link){: .btn .btn--primary .btn--large}
[Default Button](#link){: .btn .btn--primary }
[Small Button](#link){: .btn .btn--primary .btn--small}
```
[X-Large Button](#link){: .btn .btn--primary .btn--x-large}
[Large Button](#link){: .btn .btn--primary .btn--large}
[Default Button](#link){: .btn .btn--primary }
[Small Button](#link){: .btn .btn--primary .btn--small}

{% capture button %}
`{: .btn}`기본 형태에서 `.btn--primary`, `.btn--info` 등을 붙이면 색상이 변하게 된다.<br>
기본적으로 색에 대한 의미가 있다.<br>
`중요`, `성공`, `주의`, `위험`, `정보` 등 상황에 맞춰서 사용해도 되고, 자기가 사용하고 싶은 색상을 사용해도 된다.
{% endcapture %}

{% include notice--info title="설명" content=button %}

## notice (배경 색상)

```markdown

**Watch out!** This paragraph of text `{: .notice--primary}` class.
{: .notice--primary}

**Watch out!** This paragraph of text `{: .notice--info}` class.
{: .notice--info}

**Watch out!** This paragraph of text `{: .notice--warning}` class.
{: .notice--warning}

**Watch out!** This paragraph of text `{: .notice--success}` class.
{: .notice--success}

**Watch out!** This paragraph of text `{: .notice--danger}` class.
{: .notice--danger}


{% raw %}{% capture notice-text %} {% endraw %}
You can also add the .notice class to a div element.

* Bullet point 1
* Bullet point 2
{% raw %}{% endcapture %}{% endraw %}

<div class="notice--info">
  <h4>Notice Headline:</h4>
  {% raw %}{{ notice-text | markdownify }} {% endraw %}
</div>
```

**Watch out!** This paragraph of text `{: .notice}` class.
{: .notice}

**Watch out!** This paragraph of text `{: .notice--primary}` class.
{: .notice--primary}

**Watch out!** This paragraph of text `{: .notice--info}` class.
{: .notice--info}

**Watch out!** This paragraph of text `{: .notice--warning}` class.
{: .notice--warning}

**Watch out!** This paragraph of text `{: .notice--success}` class.
{: .notice--success}

**Watch out!** This paragraph of text `{: .notice--danger}` class.
{: .notice--danger}

{% capture notice-text %}
You can also add the `.notice` class to a `<div>` element.

* Bullet point 1
* Bullet point 2
{% endcapture %}

<div class="notice--info">
  <b>Notice Headline:</b><br>
  {{ notice-text | markdownify }}
</div>

## 각주
페이지 최하단에 표시되어집니다.

각주1 [^1], 각주2 [^2]
 
[^1]: 각주 내용 1
[^2]: 각주 내용 2
```
각주1 [^1], 각주2 [^2]
 
[^1]: 각주 내용 1
[^2]: 각주 내용 2
```

## reference
- [imreplay.com](https://imreplay.com/blogging/Elements/#)
- [gjchoi.github.io/env/Kramdown](http://gjchoi.github.io/env/Kramdown(%EB%A7%88%ED%81%AC%EB%8B%A4%EC%9A%B4)-%EC%82%AC%EC%9A%A9%EB%B2%95/)

{% include outro %}
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTg4MTE5OTE5OSwyMDY0ODczODQsLTQyMj
AzMjgwOSwtMTcwMzQyODAyMCwyMTUyMzAxNzMsNTgwMTc4NTMs
NzQ3MjMxOTM1LC05NzI2Mjg1NDMsMTgxNDY1MzQyMV19
-->