---
title: "[JAVA] mutable, immutable"
date: 2018-05-10 13:30:52
tags: 
    - java
    - immutable
categories: java
---

> 자바에서 String은 mutable이야? immutable이야?

친구의 질문으로 시작된 게시글!

> mutable: 변할 수 있는; 잘 변하는
> immutable: 변경할 수 없는, 불변의

사전적 정의로 보면 이런 뜻을 가지고 있다.
그리고 나는 단순하게 이렇게 생각했다.

{% codeblock lang:java %}
String userName = "홍길동";
System.out.println(userName);

userName = "둘리";
System.out.println(userName);

// result
홍길동
둘리
{% endcodeblock %}
값이 변하는데? 그럼 mutable아니야?
응 아니야.

여기서 변할 수 있다, 없다의 기준은 우리가 눈으로 보는 String 값이 아니다.
우리가 위와 같은 행동을 했을 때는 참조하고 있던 값을 변경하는 것이 아니라 새로운 객체를 재할당 받는 것이다.
{% post_link java-string-concat-append %}
마침 바로 이전글을 참고해보면 알 수 있는 정보가 있다!

### immutable한 String, mutable한 StringBuilder
String 문자열을 이어붙일 때 concat을 쓰면 새 주소값이 생기는 것을 확인했었다.
{% codeblock lang:java %}
String result = "Hello";
result = result.concat("World");
{% endcodeblock %}

| 주소 |     값     |
|:----:|:----------:|
| 1000 | Hello      |
| 2000 | HelloWorld |

이유는 concat할때마다 `new String()`으로 새로운 객체를 만들어 할당받기 때문!

굳이 문자열 붙이는 작업이 아니라 첫번째 언급했던 코드 대로 본다면 
새로운 주소값을 가지고 있는 객체를 `userName`변수에 재할당 하는 것이다.

| 주소 |   값   |
|:----:|:------:|
| 1000 | 홍길동 |
| 2000 | 둘리   |

 1. userName에 홍길동 값을 가지고 있는 1000번 주소를 할당
 2. 값이 변경되면 1000번을 가리키고 있던 것을 새로운 객체가 있는 2000번으로 변경!

반면에 StringBuilder는 append로 객체를 이어붙여 주소값이 변경되지 않는다.
{% codeblock lang:java %}
StringBuilder result = new StringBuilder();
result.append("Hello");
result.append("World");
{% endcodeblock %}

| 주소 |     값     |
|:----:|:----------:|
| 1000 | Hello      |
| 1000 | HelloWorld |

### immutable한 클래스는 어떤게 있나요?
String, Boolean, Integer, Float, Long 등이 있습니다.

### immutable한 클래스를 만들어 봅시다
immutable은 값을 변경할 수 없는 클래스를 뜻한다. 즉 set 메소드가 없다!
또한 final 키워드를 사용해 변수 초기화 이후 바뀌지 않도록 막는다.
{% codeblock lang:java %}
public class ImmutableTest {
     
    private final String userName;
     
    ImmutableTest(String userName){
        this.userName = userName;
    }
     
    @Override
    public String toString(){
        return this.userName;
    }
 
}
{% endcodeblock %}

{% codeblock lang:java %}
String userName = "홍길동";
ImmutableTest immutableString = new ImmutableTest(userName);
userName = new String("둘리");
System.out.println(immutableString);
{% endcodeblock %}

결과 값은 immutable로 만들었던 userName(홍길동)은 변하지 않고, 새로운 객체 둘리가 userName에 할당된다.

***
##### 참고사이트
http://hashcode.co.kr/questions/727/자바에서-immutable이-뭔가요
http://limkydev.tistory.com/68