---
title: Singleton pattern(싱글턴 패턴)
categories: java
tags: ["java"]
date: 2018-05-03 17:58:17
---


## Singleton pattern(싱글턴 패턴)
> 소프트웨어 디자인 패턴에서 싱글턴 패턴(Singleton pattern)을 따르는 클래스는, 생성자가 여러 차례 호출되더라도 실제로 생성되는 객체는 하나이고 최초 생성 이후에 호출된 생성자는 최초의 생성자가 생성한 객체를 리턴한다.
이와 같은 디자인 유형을 싱글턴 패턴이라고 한다. 주로 공통된 객체를 여러개 생성해서 사용하는 DBCP(DataBase Connection Pool)와 같은 상황에서 많이 사용된다.
출처: [위키백과]

[위키백과]: https://ko.wikipedia.org/wiki/싱글턴_패턴 "위키백과"


우선 아래와 같은 클래스 하나를 만들어보자
```
public class Wallet {
    ...
}
```

그리고 이를 생성자를 통해 만들어보면
```
Wallet myWallet1 = new Wallet();
Wallet myWallet2 = new Wallet();

System.out.println(myWallet1);
System.out.println(myWallet2);
```
출력값은 어떻게 나올까? 
```
Wallet@60e53b93
Wallet@5e2de80c
```
서로 다른 주소값을 가진 객체가 두개 생성되는 것이다.

이를 이제 싱글톤 패턴으로 만들어보려고 한다! ~~내 지갑은 하나뿐이니까..~~
아까 만들었던 지갑클래스를 다시 보자
```
public class Wallet {
    private static Wallet wallet;
    
    private Wallet(){}

    public static Wallet getWallet() {
        if (wallet == null) {
            wallet = new Wallet();
            return wallet;
        }else {
            return wallet;
        }
    }
}

```
생성자를 `private`으로 바꿔줬다. 이를 통해 new로 생성되는 것을 막는다.
또 이 코드는 **Lazy Initialization(게으른 초기화)** 가 적용되었다. **게으른 초기화** 란 실제로 객체가 필요하기 전까지는 객체를 만들지 않았다가 필요할 때 생성하므로 메모리 낭비를 줄일 수 있다.

자 이제 아까와 같이 두개의 지갑을 만들어보자
```
Wallet myWallet1 = Wallet.getWallet();
Wallet myWallet2 = Wallet.getWallet();

System.out.println(myWallet1);
System.out.println(myWallet2);
```

```
Wallet@60e53b93
Wallet@60e53b93
```
이제 내가 지갑을 아무리 여러개 가지고 있어도 내 자산은 한곳에서 관리됨을 볼 수 있다..(~~지갑이 많다고 돈이 많은게 아니야ㅠㅠ~~)

#### Double-checked locking
DCL(Double-checked locking)
이전까지는 호출 시마다 동기화를 진행하므로, DCL기능을 이용해 객체가 생성되어 있지 않을 때만 동기화하여 성능저하를 완화해 보자!
```
public class Wallet {
    private volatile static Wallet wallet;
    
    private Wallet(){}

    public static Wallet getWallet() {
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
```
자 이제 완벽한 코드가 되었따!!!라고 생각하면 오산이다.
DCL또한 안티패턴(anti-pattern)의 하나다.

*안티패턴(anti-pattern): 실제 많이 사용되는 패턴이지만 비효율적이거나 비생산적인 패턴*

Multi-thread(멀티쓰레드)환경에서 동시에 객체를 초기화하면 객체가 두개가 생겨버리는 현상이 일어날 수도 있기 때문이다.
우리가 원하던 결과는 하나의 객체를 공유하는 거였는데, 동시에 `new Wallet()`을 호출하면 객체가 두개!!

검색해보니 Effect Java책에서도 추천했다는 방법이 있다.
#### Enum
```
public enum Wallet {
	WALLET;
	
	public WALLET getInstance() {
		return WALLET;
	}
}
```
이렇게 열거형 객체인 enum을 사용하는 것인데 enum에 대해서는 따로 게시글로 다뤄봐야겠다.
enum은 전역변수로도 사용이 가능하고, enum타입의 `WALLET`이 생성될 때 멀티쓰레드로부터 안전하다.

### 싱글톤은 그래서 언제쓰여?
- 위키에 나와있듯이 DBCP처럼 공통된 객체를 여러개 생성해서 사용해야 하는 상황에서 사용된다.
- 객체가 꼭 하나여야만 하는 상황에 사용된다.
- 고정된 메모리 영역을 얻기 때문에 메모리 낭비를 방지할 수 있다. -> 두번째부터 객체 로딩 시간이 줄어든다(개이득!)

### 단점도 있겠지
- 하나의 객체로 여기저기 쓰인다 == 결합도가 높아진다
- 결합도가 높아지면 한 곳에서의 변경이 다른데에 영향을 끼쳐 나비효과가 될지도...
- 멀티쓰레드환경에서 동기화처리를 해줘야함(할일++)
- 잘쓰면 좋겠지만 그게 어렵다


공부할 수록 감자밭에서 감자캐는 느낌이라 오늘은 여기까지!(~~난 하나만 캐려고 했는데 자꾸 따라 나와...~~)
궁금한게 너무 많지만 하나하나씩 더 공부해봐야겠다ㅠㅠ

***
##### 참고사이트
https://blog.seotory.com/post/2016/03/java-singleton-pattern
http://woowabros.github.io/tools/2017/07/10/java-enum-uses.html
https://medium.com/@joongwon/multi-thread-환경에서의-올바른-singleton-578d9511fd42