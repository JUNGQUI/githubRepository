---
title: "Oop"
date: 2020-10-12T22:16:55+09:00
draft: true
tags: ["programming"]
---

## OOP?

Object Oriented Programming 의 앞 글자를 따서 만든 준말이다.

해석하자면 그 유명한 `객체지향 개발` 이 된다.

객체가 무엇인지에 대해 먼저 알아봐야 할텐데 말 그대로 `객체` 를 뜻한다. 여기서 객체라 함은 현실세계의 모든 개념으로 볼 수 있다.

대체적으로 class 로 구현을 해서 객체화 (encapsulation) 한다.

## 왜 알아야 하나?

사실 java 로 개발을 하면 자연스럽게 class 로 만들고 하다 보면 자연스럽게 ~~본의든 아니든~~ 객체 지향 개발 을 하게 된다.

엔티티를 class 로 만들고. function 과 method 도 class 로 만들고... 진행하다보면 모든 요소가 class 로 객체화 된다.

그렇게 해서 얻는 이득은 뭐가 있을까?

우선 제일 큰 요소는 모든 (method, entity 등) 요소들이 하나의 class 로 관리가 된다는 것이다. 이렇게 명확하게 code 상의 분리를 통해
이상한 부분에 대해 빠르게 파악 할 수 있고 하나의 business logic 에 온전히 하나의 class 가 부담하게 되므로 집중하여 개발 할 수 있다.

이러한 부분이 높은 응집도와 낮은 결합도를 불러일으키며 이러한 부분들이 유지보수 비용 감소와 원활한 개발 

## 구성

- 객체

현실 세계의 모든 것들이 객체가 될 수 있다. 좁게는 하나의 재료이거나 실존하는 물건이 될 수 있고 넓게는 어떠한 비즈니스 로직이 될 수 있다.

>클래스와 타입 사이의 차이는 꼭 이해해 두어야 합니다. 
>
>객체의 클래스는 그 객체가 어떻게 구현되느냐를 정의합니다. 
>클래스는 객체의 내부 상태와 그 객체의 연산에 대한 구현 방법을 정의합니다. 
>
>반면, 객체의 타입은 그 객체의 인터페이스, 즉 그 객체가 응답할 수 있는 요청의 집합을 정의합니다. 
>하나의 객체가 여러 타입을 가질 수 있고 서로 다른 클래스의 객체들이 동일한 타입을 가질 수 있습니다. 
>
>즉, 객체의 구현은 다를지라도 인터페이스는 같을 수 있다는 의미입니다.
>
> — GoF의 디자인 패턴, p.46

솔직히 무슨 말인지 이해가 안간다. 다만 객체는 class 와 type 으로 나뉜다는 것은 알겠다.

좀 더 자세히 보자.

```java
// class
public class Board {
  private Long id;
  private String content;
}

// type
public interface IBoardService {
  String returnBoard(String content);
}

// type 에 대한 구현체들
public class BoardService implements IBoardService {
  public String returnBoard(String content) {
    return content;
  }
}

public class BoardAService implements IBoardService {
  public String returnBoard(String content) {
    return "A " + content;
  }
}

public class BoardBService implements IBoardService {
  public String returnBoard(String content) {
    return "B " + content;
  }
}
```

이와 같이 객체의 type 은 해당 class 가 변화 할 수 있는 종류'들' 인 셈이고 이러한 부분을 반영하기 위해 interface 로 구현하였다.

이와 같은 부분이 바로 encapsulation 이라 볼 수 있다.

> - encapsulation?
>
>직역하자면 `캡슐화` 인데 말 그대로 알약의 캡슐처럼 내부 정보를 숨기는 것이다.
>
>이렇게 해서 얻을 수 있는 장점이 몇 개 있는데
>
>1. 다양한 방법(implements)를 만들어 낼 수 있다.
>
>위의 interface 를 implements 한 것은 총 3개가 있다. 이와 같이 필요에 따라 다른 method 를 현재 사용중인 interface 에
>영향을 주지 않고 새로운 method 를 만들 수 있다. 이를 또한 interface 의 다형성이라 한다.
>
>2. 불필요한 정보를 외부로 유출 하지 않고 사용 할 수 있다.
>
>보통은 interface 를 사용하여 결과값과 parameter 만 제공함으로써 내부의 로직에 대해 사용자가 신경을 쓸 필요가 없게 구현을 하게 된다.
>
>이렇게 할 경우 사용자는 그저 black-box 처럼 안의 로직은 모르지만 결과는 받을 수 있게 되는데, 이 때 사용자를 interface 를 사용하는 class라고 한다면
>관심사의 분리가 일어나기 때문에 높은 응집도와 낮은 결합도를 가져갈 수 있다.

- 응집도

하나의 class 에 얼마나 깊은 비즈니스 로직이 포함되어 있는지를 판단하는 지표다.

예를 들어, 계산을 담당하는 class 에 계산뿐 아니라 입력, 출력, 이모티콘을 그리는 기능까지 담당한다면 `계산`이라는 class의 행위에 대해
응집도가 떨어지게 구현하였다 볼 수 있다.

응집도는 높을수록 하나의 책임에만 집중하기에 담당하는 업무가 명확하게 구분되어 있고 그렇기에 유지보수에 이득을 가져가게 된다.

- 결합도

각 class 가 얼마나 서로를 참조하는 것에 대한 지표이다. 위의 계산기를 예로 들어보자.

응집도를 높히라는 것에 대해 이해를 하여 다음과 같이 나눴다.

```java
public class Calculator {

  public void calculator(int a, int b, String sign) {
    int result = 0;
    switch (sign) {
      case "+":
        result = a+b;
      case "-":
        result = a-b;
      case "*":
        result = a*b;
      case "/":
        result = a/b;
   }
   print(result);
  }
  
  public void print(int result) {
    System.out.println("결과 : " + result);
  }
}
```

출력은 출력을, 계산은 계산을 담당한다. 하지만 결과를 우리가 봐야 하기 때문에 출력에게 출력을 요청하고 계산기가 전부 받아서 진행하다.

응집도는 완벽(...)한 것 같다. 하지만 뭔가 이상하다.

응집도는 나름 나눈답시고 나눴는데 만약 출력해야 하는게 API 형식으로 결과 자체를 해야 한다면 어떻게 될까? 결과는 print를 result 로 출력하게 바꾸던가, 계산기 부분에서 결과 값을
그대로 return 하게 `수정` 해야 한다. 이 수정해야 한다는 부분이 문제가 된다.

이러한식으로 서로가 서로에 대해 깊게 참조가 되어 있는 경우에 대해 `결합도가 높다` 라고 표현한다.

이렇게 결합도가 높다면 비즈니스 로직이 바뀌는 매 순간마다 전체적인 code 변경이 일어나게 되고 품질 하향을 일으킨다.


## 그래서?

사실 개발년생 초기에는 그저 interface 로 감싸서 만들면 된다. ~~끗~~ 라는 식으로 개발을 진행 했었는데, 이후 조금 연차가 쌓이고 code 를 살펴보니
말만 interface 로 되어 있고 실상 interface 의 장점을 전혀 살리지 못한 부분들이 많았다. 그러한 부분들로 인해 비즈니스 로직이 수시로 바뀜에 따라 큰 유지보수적인 손해를 받았고
공부하지 않고 경험을 먼저해서 와닿는 부분들에 대해 정리를 하였다.

필자처럼 경험하고 고생한다음에 깨닫지 말고 미리 알고 경험에 적용하여 훌륭한 개발 이력을 추가하기를 바란다.

참고는 [여기](https://woowabros.github.io/study/2016/07/07/think_object_oriented.html) 를 잘 봤다.

P.S. OOP 는 더럽다(?) 와 같은 이론이 한때 유행한 적이 있었다. 필자는 잘 쓰면 나쁜 기술은 없다고 (세나기) 생각하는 위주이기에
적용했다고 했는데 좋은 점을 느끼지 못한다면 더더욱 노력해야 한다고 생각한다.