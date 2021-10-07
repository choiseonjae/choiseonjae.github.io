---
title: \[Kotlin] ObjectMapper enum의 name이 아닌 원하는 값으로 enum to json / json to enum 하기

categories: kotlin

tags:
   - kotlin
   - ObjectMapper
   - JsonValue
   - jackson

last_modified_at: 2021-10-07

---

다른 서비스, 서버랑 통신하다보면 미리 합의한 전문으로 통신하게 된다. 간혹, result code가 정의되어 있는 경우가 많은데 숫자로 result code를 반환할 경우, enum으로 정의하기 힘들다.

나같은 경우에는 숫자에 prefix로 _를 붙이는 방식으로 enum class 를 생성했다.

```kotlin
enum ResultCode (val description) {
	_0000("성공"),
	_9999("실패");
}
```

이런 [ResultCode.name](http://ResultCode.name) 과 실제 result code의 값이 달라지는 경우, ObjectMapper를 통해서 transform하기 쉽지 않다.

찾아본 결과 메소드 하나만 만들어주면 직렬화 / 역직렬화를 쉽게 할 수 있다. 예시로 알아보자

---

# @JsonValue 를 이용해서 ObjectMapper Custom 하기

```java
enum ResultCode (val description) {
	_0000("성공"),
	_9999("실패");

	@JsonValue
	override fun toString(): String {
	    return when(this) {
					_0000 -> "0000"
	        _9999 -> "9999" 
	        else -> this.name
	    }
	}
}
```

`@JsonValue` 어노테이션을 입력한 뒤에 메소드를 생성해주면 ObjectMapper가 read / write할 때 해당 메소드를 참조해서 동작한다. 

`override`를 붙인 이유는 `toString`을 `override`했기 때문이고, 굳이 `toString`으로 메소드명을 정의하지 않아도 되고, 그럴 경우 `override`를 붙여주지 않아도 된다.

위의 예제처럼 `enum`에 정의해놓은 경우에 결과를 살펴보면

- `println(ResultCode._9999)` → **9999** // 그냥 enum만 호출해도 toString()이 찍히기 때문에 9999로 나온다.
- `println(ResultCode._9999.toString())` → **9999**
- `println(ResultCode._9999.name())` → **_9999** // enum 자체의 코드를 호출하기 때문에 `_`가 붙어서 나온다.

실제 test code를 작성해서 확인해봤다.

```groovy
class ObjectMapperConfigTest extends Specification {

    ObjectMapper mapper

    def setup() {
        mapper = new ObjectMapper()
    }

    def "objectMapper transform test about 9999 : success"() {
        given:
        def json = "{\"code\": \"9999\"}"

        when:
        def actual = mapper.readValue(json, ResultCodeTestClass.class)

        then: // test 결과는 성공이다.
            actual.resultCode == ResultCode._9999
            actual.resultCode.toString() == "9999"
            actual.resultCode.name() == "_9999"
    }
}

class ResultCodeTestClass {
    @JsonProperty("code")
    public ResultCode resultCode; // 위에서 만든 enum class를 그대로 사용
}
```

# 마치며

@JsonValue 하나만 사용하면 쉽게 바뀌는 것을 보면, 참 사용하기 좋은 라이브러리 라는 걸 새삼느끼는 것 같다. 나중에 날 잡고 ObjectMapper의 다양한 기능을 살펴봐야겠다.
