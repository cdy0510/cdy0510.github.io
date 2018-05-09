---
title: "[JAVA] 문자열 붙이는 방식의 차이(concat, +, StringBuilder)"
date: 2018-05-09 15:40:10
categories: java
tags: ["java"]
---

자바에서 String타입을 붙일 때 사용하는 방법은 다양하다.
기본 연산자인 `+`를 비롯하여 `String Builder`, `concat` 모두 들어보거나 써본 용어일 것이다.
근데 동작 방식에 어떤 차이가 있을까?

먼저 결과값으로만 비교해보자.
{% codeblock lang:java %}
public class Main {
    public static void main(String[] args) {
        String strSample1 = "Hello";
        String strSample2 = "World";
        String result1 = strSample1 + strSample2;
        String result2 = strSample1.concat(strSample2);
        StringBuilder result3 = new StringBuilder();
        result3.append(strSample1);
        result3.append(strSample2);

        System.out.println(result1);
        System.out.println(result2);
        System.out.println(result3);
    }
}
{% endcodeblock %}
출력값은 아래와 같다.
```
HelloWorld
HelloWorld
HelloWorld
```
모두 동일하게 나온다!!
대체 뭔차이지? 그냥 아무거나 골라쓸랭.. 왜 여러개를 만든걸까 ~~는 모두 내생각 이었다~~
결정적으로 결과값은 모두 동일하지만 성능에 차이가 있다. 또한 초기값에 따라 오류가 날 수 있다.


### concat
`concat`명령어를 쓸 경우 초기값이 null이면 안된다.
{% codeblock lang:java %}
String strSample1 = null;
String result = strSample1.concat("Hi");
System.out.println(result);

// 결과
java.lang.NullPointerException
{% endcodeblock %}
`concat`명령어는 더하려는 값을 new String()으로 새로 만든다.
따라서 문자열을 계속해서 붙인다고 가정하면, 붙일 때마다 주소값을 할당받게 되는 것이다.

| 주소 |     값     |
|:----:|:----------:|
| 1000 | Hello      |
| 2000 | HelloWorld |


### StringBuilder
내가 `StringBuilder`를 쓸 때 편리했던 점은 초기화를 안해도 된다는 것이었다.
{% codeblock lang:java %}
StringBuilder result = new StringBuilder();
result.append("Hi");

System.out.println(result);

// 결과
Hi
{% endcodeblock %}
`StringBuilder`는 append를 통해서 문자열을 붙여준다. 아까 `concat`과의 차이는 문자열을 계속 붙여도 주소값이 변하지 않는다는 것이다.

| 주소 |     값     |
|:----:|:----------:|
| 1000 | Hello      |
| 1000 | HelloWorld |

그리고 재밌는 사실 하나가 있다. 만약 null과 문자열을 더하면 결과가 어떻게 나올까?
{% codeblock lang:java %}
String strSample1 = null;
String strSample2 = "World";

StringBuilder result = new StringBuilder();
result.append(strSample1);
result.append(strSample2);

System.out.println(result);

// 결과
nullWorld
{% endcodeblock %}
오류는 나지 않고 nullWorld라는 문자열이 출력된다. 이는 상속받은 클래스 AbstractStringBuilder 내부를 보면 알 수 있다.
{% codeblock lang:java %}
public AbstractStringBuilder append(String str) {
    if (str == null)
        return appendNull();
    ...
}

private AbstractStringBuilder appendNull() {
    int c = count;
    ensureCapacityInternal(c + 4);
    final char[] value = this.value;
    value[c++] = 'n';
    value[c++] = 'u';
    value[c++] = 'l';
    value[c++] = 'l';
    count = c;
    return this;
}
{% endcodeblock %}
`appendNull()`을 통해 문자열을 하나씩 붙여주고 있었던 것이다.


### +연산자
자바 버전에 따라 다른데, 1.5 이전에는 `concat`을 이용하는 방식과 같고
1.5 이후에는 `StringBuilder`를 이용하는 방식과 같다.
이 내용은 아래 주소에서 읽다가 알게된 사실이다.
https://www.slipp.net/questions/271


### 마무리
결과값이 같아 아무거나 써도 될 것 같지만 구조적으로 다르기 때문에 사실 결과값이 완전히 같다고 할수는 없다.
new를 통한 메모리를 낭비하고 싶지 않다면 `StringBuilder`를 사용하는 것이 좋고 JDK1.5이상 버전을 쓴다면 `+`를 사용해도 무관할 것 같다.


***
##### 참고사이트
https://programmers.co.kr/learn/questions/571
https://www.slipp.net/questions/271