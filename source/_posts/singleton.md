---
title: "[JAVA] Singleton pattern(싱글턴 패턴)"
categories: java
tags: ["java"]
date: 2018-05-03 17:58:17
---

## Singleton pattern(싱글턴 패턴)
소프트웨어 디자인 패턴에서 싱글턴 패턴(Singleton pattern)을 따르는 클래스는, 생성자가 여러 차례 호출되더라도 실제로 생성되는 객체는 하나이고 최초 생성 이후에 호출된 생성자는 최초의 생성자가 생성한 객체를 리턴한다.
이와 같은 디자인 유형을 싱글턴 패턴이라고 한다. 주로 공통된 객체를 여러개 생성해서 사용하는 DBCP(DataBase Connection Pool)와 같은 상황에서 많이 사용된다.
출처: [위키백과]

[위키백과]: https://ko.wikipedia.org/wiki/싱글턴_패턴 "위키백과"


우선 아래와 같은 클래스 하나를 만들어보자
{% codeblock lang:java %}
public class Wallet {
    ...
}
{% endcodeblock %}

그리고 이를 생성자를 통해 만들어보면
{% codeblock lang:java %}
Wallet myWallet1 = new Wallet();
Wallet myWallet2 = new Wallet();

System.out.println(myWallet1);
System.out.println(myWallet2);
{% endcodeblock %}
출력값은 어떻게 나올까? 
```
Wallet@60e53b93
Wallet@5e2de80c
```
서로 다른 주소값을 가진 객체가 두개 생성되는 것이다.

이를 이제 싱글톤 패턴으로 만들어보려고 한다! ~~내 지갑은 하나뿐이니까..~~

### Lazy Initialization(게으른 초기화)
실제로 객체가 필요하기 전까지는 객체를 만들지 않았다가 필요할 때 생성하므로 메모리 낭비를 줄일 수 있다.

{% codeblock lang:java %}
public class Wallet {
    private static Wallet wallet;
    
    private Wallet(){}

    public static Wallet getInstance() {
        if (wallet == null) {
            wallet = new Wallet();
        }
        
        return wallet;
    }
}
{% endcodeblock %}

우선 생성자를 `private`으로 바꿔줬다. 이를 통해 new로 생성되는 것을 막는다.
그리고 객체가 null인지를 판별해 null일 경우에만 객체를 생성한다.

자 이제 아까와 같이 두개의 지갑을 만들어보자
{% codeblock lang:java %}
Wallet myWallet1 = Wallet.getInstance();
Wallet myWallet2 = Wallet.getInstance();

System.out.println(myWallet1);
System.out.println(myWallet2);
{% endcodeblock %}

```
Wallet@60e53b93
Wallet@60e53b93
```
이제 지갑을 여러개 생성하더라도 모두 같은 주소값을 가리키고 있어 같은 값을 가지게 된다.
Multi-thread(멀티쓰레드)환경에서는 동기화 처리도 필요하다. 하나의 객체를 수정하고 있을 때 다른 곳에서 함께 수정하면 정확하지 않은 데이터가 입력될 수 있기 때문이다.
자바에서 이를 도와주는 키워드가 바로 `synchronized`이다. 이 키워드를 명시해주면 접근권한이 있더라도 이전 작업이 완전히 끝날 때까지 lock을 걸어 데이터를 보호해준다.

{% codeblock lang:java %}
public class Wallet {
    private static Wallet wallet;

    private Wallet(){}

    public static synchronized Wallet getInstance() {
        if (wallet == null) {
            wallet = new Wallet();
        }
        
        return wallet;
    }
}
{% endcodeblock %}

하지만 이렇게 명시하면 호출 시마다 동기화를 진행하므로, 성능 저하가 있을 수 있다.
이를 예방하기 위한 방법을 살펴봐야겠다.

### Double-checked locking(DCL)
객체를 얻을 때마다 동기화하는 overhead를 줄이기 위한 방법으로 고안된 디자인 패턴이다.
{% codeblock lang:java %}
public class Wallet {
    private volatile static Wallet wallet;
    
    private Wallet(){}

    public static Wallet getInstance() {
        if (wallet == null) {
            synchronized (Wallet.class) {
                if (wallet == null) {
                    wallet = new Wallet();
                }
            }
        }
        
        return wallet;
    }
}
{% endcodeblock %}
하지만 그닥 좋은 문법은 아니다. 이 패턴을 검색해보면 안티패턴(anti-pattern)의 하나로 언급되고있다.

*안티패턴(anti-pattern): 실제 많이 사용되는 패턴이지만 비효율적이거나 비생산적인 패턴*

Multi-thread(멀티쓰레드)환경에서 오작동할 가능성이 있기 때문이다
- 동시에 객체를 초기화하면 객체가 두개가 생겨버리는 현상이 일어나거나
- 생성자는 생성이 완료되기 전에 메모리 공간을 할당받게 되는데, 미처 생성이 완료 되기 전에 '어 이미 할당된 객체가 있네? 사용해야지!'라고 접근하게 되면 오류가 나게 된다.

이 코드에 `volatile`이라는 키워드는 java버전 1.5이상 사용 시 추천한다. 또한 자세한 설명은 http://kwanseob.blogspot.kr/2012/08/java-volatile.html 여기를 참고해보면 좋을 것 같다.

어쨋든 DCL패턴은 이미 안티패턴으로 규정이 되었고, 다른 방법을 생각해봐야할 것 같다.

### Enum
{% codeblock lang:java %}
public enum Wallet {
    INSTANCE;
    ...
}
{% endcodeblock %}
이렇게 열거형 객체인 enum을 사용하는 것인데 enum에 대해서는 따로 게시글로 다뤄봐야겠다.
enum은 전역변수로도 사용이 가능하고, enum타입의 `INSTANCE`가 생성될 때 멀티쓰레드로부터 안전하다.
하지만 추가된 메소드는 안전하지 않을 수 있다. 그래서 이 방법도 그닥 좋아보이지는 않는다.

### initialization-on-demand holder
늦은 초기화기법과 holder를 함께 쓰는 디자인 패턴이다. 모든 자바버전에서 안전하게 사용이 가능하고, 좋은 성능을 보여준다.

{% codeblock lang:java %}
public class Wallet {
    private Wallet() {}

    private static class LazyHolder {
        static final Wallet INSTANCE = new Wallet();
    }

    public static Wallet getInstance() {
        return LazyHolder.INSTANCE;
    }
}
{% endcodeblock %}
최초로 `getInstance()`를 실행 했을 때 클래스 로더에 의해 객체가 생성되고 `final`로 선언하여 값이 다시 할당되지 않도록 해준다.
이 방법이 가장 많이 사용되는 싱글톤 패턴이라고 한다.

### 싱글톤은 그래서 언제쓰여?
- 위키에 나와있듯이 DBCP처럼 공통된 객체를 여러개 생성해서 사용해야 하는 상황에서 사용된다.
- 객체가 꼭 하나여야만 하는 상황에 사용된다.
- 고정된 메모리 영역을 얻기 때문에 메모리 낭비를 방지할 수 있다. -> 두번째부터 객체 로딩 시간이 줄어든다(개이득!)

### 단점도 있겠지
- 결합도가 높아진다
- 객체지향 설계 원칙에 따라 개발하는 것을 저해하는 요인이 된다
- 생성자를 private으로 구현하기 때문에 상속할 수 없다
- 멀티쓰레드환경에서 동기화처리를 해줘야함(할일++)
- 잘쓰면 좋겠지만 그게 어렵다

***
##### 참고사이트
https://blog.seotory.com/post/2016/03/java-singleton-pattern
http://woowabros.github.io/tools/2017/07/10/java-enum-uses.html
https://medium.com/@joongwon/multi-thread-환경에서의-올바른-singleton-578d9511fd42
http://limkydev.tistory.com/67