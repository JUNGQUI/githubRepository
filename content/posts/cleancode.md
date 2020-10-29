---
title: "Cleancode"
date: 2020-10-27T23:53:17+09:00
draft: true
tags: ["Programming"]
---
## Clean Code?

직관적으로 `깔끔한 코드` ~~코드는 외래어...~~ 라는 뜻이다.

직관적으로 보기 좋은 깔끔한 코드라는 의미인데, 로직적으로도 불필요한 연산을 빼버리고 최대한 compact 하게 만들어서 가독성을 늘린 코드를 의미한다.

대표적으로 대원칙이 있는데 이를 `객체지향 생활체조 원칙` 이라고 한다.

## 왜 알아야 하나?

가독성이 높아질수록 유지보수가 높아지는건 당연지사이다. 그 와중에 코드가 깔끔해지면 business logic 이 더욱 눈에 띄고 설계 당시 business logic 에서 꼭
필요하다고 생각했던 부분들이 의외로 필요 없어서 제거하거나, error 발생 시 쉽게 찾아내거나 포커싱하기 쉬워지는 장점이 있다.

단점으로는 이렇게까지 생활화 하기 전까지 체득하기가 너무 어렵다는 점이다.
하지만 본인이 10년 이상 개발로 밥벌어먹고 살려면 이러한 부분들에 대해서 잘 알아야 업무 스트레스가 줄어들지 않을까?

## 구성

위에서 언급했듯이 `객체지향 생활체조 원칙` 이 있는데, 원칙은 다음과 같다.

- 규칙 1: 한 메서드에 오직 한 단계의 들여쓰기(indent)만 한다.
- 규칙 2: else 예약어를 쓰지 않는다.
- 규칙 3: 모든 원시값과 문자열을 포장한다.
- 규칙 4: 한 줄에 점을 하나만 찍는다.
- 규칙 5: 줄여쓰지 않는다(축약 금지).
- 규칙 6: 모든 엔티티를 작게 유지한다.
- 규칙 7: 3개 이상의 인스턴스 변수를 가진 클래스를 쓰지 않는다.
- 규칙 8: 일급 콜렉션을 쓴다.
- 규칙 9: 게터/세터/프로퍼티를 쓰지 않는다. 

하나씩 설명 하자면

1. 쉽게 생각해서 과도한 중복 if 문과 for 문, switch 를 쓰지 말자는 의미이다. 아래의 코드를 보자.

```java
public class JKPrintClass {
  public void printNoCleanCode (String testText) {
    if (testText.contains("a")) {
    	System.out.println("text include 'A'");
        
        if (testText.contains("b")) {
        	System.out.println("text include 'B'");
        } else if (testText.contains("c")) {
        	System.out.println("text include 'C'");
        }
    }
  }

  public void printCleanCode () {
    if (testText.contains("a")) {
        System.out.println("text include 'A'");
    }

    if (testText.contains("b")) {
        System.out.println("text include 'B'");
    } 
    
    if (testText.contains("c")) {
        System.out.println("text include 'C'");
    }
  }
}
```

두 코드의 결과는 동일하다. 하지만 `printNoCleanCode` 를 보면 if 안에 if 가 있고 그게 아닐 경우 (else if) 의 수가 또 존재하기에
만약 `print C` 부분에 에러가 생기거나 변형이 일어나야 할 경우 앞의 a 조건을 만족해야 하며 변형이 이 부분에 대해 영향을 끼치면 안된다.

하지만 `printCleanCode` 의 경우 하나의 if 조건이 하나의 책임만을 지고 있는 상황이기에 변형, 에러 발생으로 인한 수정 등에 대해서 상위의 조건에 영향을 끼치지 않는다.

이러한 부분은 [SOLID 원칙](https://jungqui.github.io/posts/solid/) SRP 에도 위배되는 상황이라고 볼 수 있다. 또한 2번 원칙인 else 도 이와 유사한 이유로
행해서는 안되는 것에 대한 원칙이라 볼 수 있다.

3. class 를 obj 로 만들 때 해당 class 의 모든 요소를 포장해서 사용하라는 의미다.

```java
@Data
@NoArgumentConstruct
@AllArgumentConstruct
public class SomeObj {
  private String title;
  private String content;
  private String zipCode;
  private String address1;
  private String address2;
}
``` 

```java
@Data
@NoArgumentConstruct
@AllArgumentConstruct
public class SomeObj {
  private Board board;
  private Address address;
}

@Data
@NoArgumentConstruct
@AllArgumentConstruct
public class Board {
  private String title;
  private String content;
}

@Data
@NoArgumentConstruct
@AllArgumentConstruct
public class Address {
  private String zipCode;
  private String address1;
  private String address2;
}
```

이 두 가지 class 를 비교해보자. 하나는 address 에 대해 공통적인 부분과 게시판과 관련된 부분이 있는것으로 보이며, 해당 property 에 대해 혼재되어 있다.

하지만 아래의 class 의 경우 명확한 책임소재가 분리되어 있다. Object 를 application layer 에서 호출했을 떄 
Board 가 비어 있으면 책임 소재를 Board 에서만 찾으면 되고,
Address 가 비어 있으면 Address 에서만 책임 소재를 찾을 수 있다. 이럴 경우 어디에서 문제가 생겼는지 확실히 알 수 있고,
(일반적으로) Address 중 일부만 채워서 만들지도 않기 때문에 개발 할 시에 object 의 무결성도 어느 정도 보장 할 수 있다.
~~물론 이렇게 한다고 무결성이 충족 되진 않는다~~

4. 알아보기 어렵다는 소리다.

```java
@Data
@NoArgumentConstruct
@AllArgumentConstruct
public class SomeObj {
  private Board board;
  private Address address;
}

@Data
@NoArgumentConstruct
@AllArgumentConstruct
public class Board {
  private String title;
  private String content;
}

@Data
@NoArgumentConstruct
@AllArgumentConstruct
public class Address {
  private String zipCode;
  private String address1;
  private String address2;
}


public class MainClass {
  public void noCleanCode() {
    SomeObj someObj = new SomeObj(new Board("title", "content"), new Address("00000", "주소1", "주소2"));
  }

  public void cleanCode() {
    SomeObj someObj = new SomeObj(
      new Board("title", "content"), 
      new Address("00000", "주소1", "주소2")
    );
  }
}
```

main class 내의 두 가지 method 를 비교해보자면 결과만 보면 동일한 결과 값이다. 두 가지 method 모두 `SomeObj` 를 만드는데, 동일한 값으로 만들게 한다.

다만 `noCleanCode` 의 경우 new 에 new 에 new 가 있어서 어떤 param 이 어떤 obj 의 param 으로 갔는지 알기 어렵다. ~~사실 간단해서 알기 쉽긴 한데 그렇다고 하자~~

하지만 `cleanCode` 의 경우 확실하게 어떤 string 이 어떤 obj 의 param 으로 들어가는지가 명확하게 파악된다. 저 new 를 . 으로 치환해서 생각해보면 4번 규칙이 이해가 될 것이다.


# TO BE CONTINUE ~~원피스!~~

## 그래서?
