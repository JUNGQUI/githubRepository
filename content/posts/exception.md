---
title: "Exception"
date: 2020-09-14T13:37:52+09:00
draft: true
tags: ["programming", "java"]
---

## Exception?

단어적인 뜻으로는 `예외` 라는 의미를 가지고 있다.

일반적으로 어떠한 약속을 제정할 때 `이럴 경우 특수하니 일반적으로 처리하지 않는다.` 라는 의미로 예외 조항등을 넣기도 한다.
마찬가지로 개발에서 `특수한, 예상치 못한 어떠한 flow 로 흘렀을 경우에 대한 처리` 가 바로 `예외 처리` 가 되겠다.

## 왜 알아야 하나?

개발 중 exception 은 다양하다. I/O 부터 sql, 대표적으로 우릴 괴롭히는 NullPointerException 까지 몹시 많다.

이러한 부분에 대해 모두 제어를 하면서 개발을 할 수 있으면 좋겠지만 사실 이러한 모든 부분들에 대해 exception 처리를 완벽하게 처리하기엔 어렵다.
하물며 business logic 자체가 수시적으로 바뀌는 real world 에서 라면 더더욱 완벽하게 처리하기 어렵다.

그렇기 때문에 흔히 try / catch 와 같은 처리를 통해 `예외일 경우 범용적으로 이렇게 하겠다` 라는 식으로 개발을 하는 경우가 왕왕 있다.

하지만 이렇게 할 경우 하위 모듈에 대한 상위 모듈의 책임감이 전파 및 부담이 증가되므로 사실상 좋은 방법은 아니다.
모든 예외를 정확하게 잡아내기엔 어렵지만, 적어도 이러이러한 예외는 나올 수 있으니 이럴 경우 명확한 error code 등이 명시된 예외를 반환하여야 하고, 그럴 경우
exception 에 대해 잘 알아야 그 protocol 에 맞게 개발 할 수 있다.

## 구성

기본적으로 Object 를 상속받은 Throwable 이 있고, error 및 exception 이 throwable 을 상속받아서 구현하게 되어있다.
또한 exception 내에서 RuntimeException, IOException, SQLException 등으로 나눠진다.

즉,

```
Throwable
|-Exception
  |-RuntimeException
  |-IOException
  |-SQLException
  ...
ㄴ-Error
```

이와 같은 구성으로 이루어져 있다.

여기서 크게 구분을 두는 부분이 바로 CheckedException 과 UncheckedException 으로 구분된다.

- CheckedException

RuntimeException 외 Exception. checked 라는 명칭에서 알 수 있듯이 이미 compile 시점에 `checked` 되어 있어야 하는 Exception 이다.
즉, code 를 짤 때 부터 해당 exception 들에 대해서는 code 상 예외 처리를 해줘야 한다.

> - 예외 처리
>
>말 그대로 예외에 대해 어떻게 처리할 것인가 에 대한 방법 차이이다. `예외 복구`, `예외 처리 회피`, `예외 전환` 이 있다.
>
>**예외 복구**
>
>예외가 발생한 부분에서 정상적인 flow 로 흐를 수 있게 error 를 복구하여 진행하는 방법이다. 정확한 error 를 맞췄다면(?) 복구가 정상적으로 진행되나
>정확한 error 가 아닌 엉뚱한 error 에 대해 처리를 했다면 정상적인 flow 로 처리 될 수 없다.
>
>**예외 처리 회피**
>
>발생한 부분을 호출한 고수준 모듈로 무작정 exception 을 발생시켜 반환하는 방법이다. 개발하는데에 굉장히 편리하게 개발 할 수 있는 장점이 있지만
>application 적으로 보자면 의미없는 무의미한 exception 이 발생하기 때문에 고수준 모듈에서도 처리를 할 방법이 없다.
>
>또한 이렇게 할 경우 error 는 저수준 모듈에서 발생했으나 책임은 고수준 모듈이 지게 되므로 유지보수 측면에서도 굉장히 불편하게 된다.
>
>**예외 전환**
>
>예외 처리 회피와 유사하게 결국 발생한 부분을 호출한 고수준 모듈로 exception 이 발생한다. 하지만 다른 점은 해당 exception 이 어떤 exception 인지, 리
>혹은 같은 exception 이여도 내부에서 정의하기에 다른 error code 로 관리되고 있다면 그 error code 까지 wrapping 하여 전달한다.
>
>이럴 경우 책임 수준은 고수준 모듈에서 지게 되지만 실질적인 error 명시는 저수준 모듈에서 정의된 후 넘어오기 때문에 넘어온 error 명시에 따라 고수준 모듈이 처리하게 되므로
>유지보수 측면에서도 명확한 사유를 확인 할 수 있게 된다. 

대표적으로 IOException, SQLException 이 있는데, 이와 같은 exception 은 file 을 읽고 난 후 혹은 DB connection 이나 정상적인 query call 이
일어난 뒤에 책임 소재가 application 으로 넘어오기 때문에 `연결이 안된 시점에서 DB 의 책임을 어떻게 처리 할 것 인가` 에 대해 명시가 되어야 한다.

그러므로 이러한 Exception 들은 compile 시점에 `확인(checked)` 되어야 하기 때문에 Checked Exception 이라 칭한다.

---

- UncheckedException

Checked Exception 과는 다르게 `하지 않아도 된다.` 라는 느낌의 exception 처리이다. (물론 그렇다고 진짜 처리하지 않으면 안된다.)

Runtime Exception 은 실행 도중 발생하는 유동적인 예외라 볼 수 있다. 다음 code 를 보자.

```java
// 이러한 class 가 있다고 가정 할 때
public class JKNode {
    public int val;
    public List<JKNode> children;

    public JKNode() {}

    public Node(int _val) {
        val = _val;
    }

    public JKNode(int _val, List<JKNode> _children) {
        val = _val;
        children = _children;
    }
}
```

```java
public class main(String[] args) {
    public void test() {
        JKNode node1 = new JKNode();
        JKNode node2 = null;

        // exception 없음
        if (node1.val == 0) {
            System.out.println("val is 0");
        }
        // NPE 발생
        if (node2.val == 0) {
           System.out.println("val is 0");
       }
    }
}
```

이와 같이 동일한 JKNode class 에 대해 null 로 물론 선언했지만, 확정적으로 error 가 발생하지는 않는다. 상황에 따라 유동적으로 발생 할 수 있기 때문에
이러한 exception 을 `확인 못하는 (unchecked) 예외`, unchecked Exception 이라 칭한다.

## 그래서?

일단 예외 처리 하는 방식에 대해 알아봤는데, 일반적으로 아무 생각 없이 code 를 짜다 보면 습관적으로 무의미한 try/catch 에 정확한 error 없이 Exception 을 
catch 하여 로그만 찍고 빠지게 된다.

초기 개발 시라면 당연히 어떠한 error 가 발생할 지 모르니 이렇게 개발 할 수 있지만, 실제 business logic 을 수행하는데 이미 안정적으로 유지 보수 단계에 들어간
application 에 여전히 전체 exception 이 있다면 유지 보수 시 history 를 전혀 모르는 개발자가 추가 투입되었다 했을 때, 혹은 내가 담당한 모듈을 사용하려고 할 때 
의사 소통이 없다면 상대는 사용하는 도중 도대체 무슨 error 가 발생해서 이렇게 된 건지 모르게 될 수 있다.

물론 checked exception 은 compile 시 error 로 marking 이 되지만 unchecked 의 경우 compile 로 잡을 수 없기 때문에 잘 처리해야 하며
처리 방법 또한 명확한 code 로 관리 할 수 있으면 best 겠지만 그렇지 않더라도 명확한 오류 원인을 제공해 고수준 모듈에서 처리 할 수 있게 전달해야 한다.
~~그러려면 잘 알아야지~~