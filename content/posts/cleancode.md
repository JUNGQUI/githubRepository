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
- 규칙 9: 게터/세터를 쓰지 않는다. 

하나씩 설명 하자면

규칙 1: 쉽게 생각해서 과도한 중복 if 문과 for 문, switch 를 쓰지 말자는 의미이다. 아래의 코드를 보자.

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

규칙 3: class 를 obj 로 만들 때 해당 class 의 모든 요소를 포장해서 사용하라는 의미다.

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

규칙 4: 알아보기 어렵다는 소리다.

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

규칙 5: 코딩 시 몇 번의 타자를 더 쓰기 싫다고 축약하는 것이 이전에는 유행했었다. 하지만 그렇게 하지 말자는 것이다.

```java
public class PW {
  public void pw(String w) {
    if (w.contains(" ")) {
    	return;
    }

    System.out.println(w);
  }
}

public class PrintWord {
  public void printWord(String word) {
    if (word.contains(" ")) {
    	return;
    }

    System.out.println(word);
  }
}
```

위 두 가지 코드를 보자면, 워낙 간단하니 사실 따지고 보면 이해가 안가진 않는다. 다만 왜 `w` 라는 변수에 빈 칸이 포함되면 안되는 것인가 에 대한 의문이 남는다.

하지만 변수 명을 보면 word 즉, 단어 라는 뜻이고 결국 함수명과 비교해 볼때 이 class 는 단어만을 출력하는 class 이구나 라고 볼 수 있다.

상대적으로 첫번째 코드가 단어 자체가 적고 쓰기도 편하지만, 알지 못하는 사람이 외부에서 봤을 시엔 한번에 이해가 되지 않는다는 단점이 있다.

최근에 IDE 에서 자동 완성 기능이 주어지기 때문에 상대적으로 축약이 힘을 잃었고 그렇기에 오히려 축약을 하지 않는 것이 깨끗한 즉, 가독성이 높은 code 가 되는 것이다.

규칙 6: 모든 엔티티에 대해 최대한 도메인이 하는 일과 관련된 부분만 가지게 작게 유지하라는 뜻이다.

```java
@Data
@NoArgumentConstruct
@AllArgumentConstruct
public class SomeObj {
  private String tit;
  private String cont;
  private String zc;
  private String ad1;
  private String ad2;
}
```

SomeObj 는 여러가지 String parameter 가 섞여 있다. 심지어 축약으로 의미는 해석하기 힘들 정도이다. ~~비속어도 끼어있다!~~

```java
@Data
@NoArgumentConstruct
@AllArgumentConstruct
public class SomeObj {
  private Board board;
  private Address address;
}
```

하지만 이렇게 구성할 경우 ~~무슨 상관이 있는지 모르겠지만~~ 게시판 역할을 하는 `Board` 와 주소 정보를 제공하는 `Address` 가 있다는 걸 알 수 있다.
이와 같은 규칙 7번에서도 일맥상통하는데 최대한 다른 class 로 분할하여 3개 이상의 인스턴스 변수를 가지지 않는다면 각각의 역할에 대해 명확히 할 수 있기에 도메인에 책임소재를 강화 할 수 있다.

규칙 8: [일급 컬렉션](https://jungqui.github.io/posts/lambda) 을 써야 한다. 자세한 설명은... 생략하지 않는다.

결국엔 규칙 7번과 관통하는 규칙인데 최대한 도메인에 집중하는 구조를 만들기 위해 단 하나의 요소만 사용하는 일급 클래스를 사용하라는 것이다.

규칙 9: getter/setter 를 쓰지 않는다.

getter setter 는 constructor 를 이용하지 않고 해당 엔티티의 property 를 바꿀 수 있는 기본적인 property 이다. 문제는 이걸 너무 자주 사용 할 경우
data 의 일관성이 보장되지 않는다는 단점이 있다.

아주 기본적인 만큼 아주 손쉽게 변경이 가능하고, 이는 비즈니스 로직 수행 시 이 data 가 내가 원했던 결과를 그대로 들고 왔을까 라는 부분에 대해서
의구심이 들 수 밖에 없다.

예시를 들어 보자.

```java
public enum OrderState {
  READY, PICKUP, DELIVERY, COMPLETE, FAIL
}

public class Item {
  private String itemName;
  private String itemStandard;
  private int qty;
}

public class Order {
  private OrderState orderState;
  private Item item;
}

public class ChangeOrderState {

  public Order makeOrder (String itemName, String itemStandard, int qty) {
    Item item = new Item(itemName, itemStandard, qty);
    return new Order(OrderState.READY, item);
  }

  public void changeOrderState (Order order, String itemStandard) {
    if (order.getOrderState == OrderState.READY && StringUtils.hasText(itemStandard)) {
    	order.setItemStandard(itemStandard);
    }
  }

  public void changeItem (Item item, String itemStandard) {
    item.setItemStandard(itemStandard);
  }
}
```

이런 주문 class 가 있다고 할 때 ~~로직으로는 도저히 이해가 안가지만~~ itemStandard 를 변경하는 부분만 3 곳에 있다.
각 method 를 따로 사용한다면 그나마 괜찮지만 저 method 들이 모든 프로젝트 내에서 조건에 따라 다르게 호출이 된다고 한다면 itemStandard 에 대해 하나의 일관된
값을 기대 할 수 없게 된다.

이렇게 한다면 error 가 발생했을 경우 어디에서 발생했는지 명확하게 알기 힘들 뿐더러 만약 itemStandard 를 migration 해야 한다고 할 때 
쉽게 손대기도 어려운 구조가 된다.

## 그래서?

결국엔 가독성이 좋은 코드가 클린 코드라고 볼 수 있다. 예전에는 기기적인 문제와 (속도와 메모리) IDE 의 지원 한계 (자동 완성 등) 로 인해 
이러한 부분에 집중하기 어려웠고 어쩔 수 없이 일인전승을 하듯이 모든 기업마다 각기의 컨벤션이 존재하였다. 다만 당연하게도, 이렇게 구현 할 경우
새로운 인원에 대한 유입의 장벽이 생기며 변화에 대해서도 민감하게 대응하기 어렵게 되는데 무어의 법칙조차 깨질 정도로 빠른 메모리 등 기기의 발전과
그에 따른 IDE 의 개발을 위한 기능 지원을 통해 매우 쉽게 코딩이 진행되다 보니 개발자 친화적으로 변화하게 되었다고 본다.

앞으로도 발전하면 발전했지 퇴보하진 않을 거기 때문에 사람 중심적인 개발 방법론이 더 지향을 받게 될 것이고, 클린 코드를 실제 적용해보면
확실하게 가독성이 늘어나기 떄문에 본인부터 체감을 하게 되니 실제로 적용하기 힘들더라도 적용하도록 노력하면 나름 점점 더 편하게 개발을 하게 되리라 생각한다.