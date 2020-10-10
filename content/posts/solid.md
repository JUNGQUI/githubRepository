---
title: "Solid"
date: 2020-09-26T18:34:42+09:00
draft: true
tags: ["programming"]
---

## SOLID?

객체지향 프로그래밍 패턴 원칙들의 약자다.

SOLID 는 각각
SRP(단일 책임 원칙), 
OCP(개방-폐쇄 원칙), 
LSP(리스코프 치환 원칙),
ISP(인터페이스 분리 원칙), 
DIP(의존 역전 원칙) 로 이루어져 있고, 앞 단어를 따서 만든 약어이다.

## 왜 알아야 하나?

사실 DDD 라던지 일급클래스, functional interface 와 같은 부분에서 이미 이러한 원칙들이 일부 적용된 method 들이 몇몇 있다.

하지만 그런 기술적인 부분을 먼저 접근하기에 앞서 원칙에 대해 이해를 하고 접근한다면 해당 기술적인 부분들이 `왜 이렇게 만들어졌는가` 에 대한 기반 지식을
쌓고 접근할 수 있기 때문에 이해하기가 훨씬 편하다.

뿐만 아니라 사실 개발을 하면서 모듈화, 응집도를 떨어뜨리는 등의 기본적으로 spring framework 를 공부하면 나오는 원칙들이
사실 이 SOLID 원칙에 모두 반영이 되어 있다.

굳이 따지면 장황하게 설명되어 있는 것들에 대해 명확하게 1줄로 정의를 해준 것이 SOLID 원칙이라고 생각한다.

## 구성

지금부터 각 원칙을 하나씩 파해쳐보자.

### SRP, Single Responsibility Principle (단일 책임 원칙)

여기에서 책임이란, 보통 하나의 logic 을 수행하는데에 수행하는 역할 정도로 분석이 가능하다.

즉 각 class, method, interface 등은 오직 하나의 business logic 을 수행해야 한다는 것인데, 이렇게 하는 이유는 간단하다.
하나의 class 안에 여러 가지 logic 이 겹쳐진 상태로 사용된다면 유지보수 측면에서 error 가 발생 시 어디에서 발생했는지 파악하기 힘든 단점이 있다.

다음은 '계산기' 라는 

```java
public class Calculator {

  public int calculator(int a, int b, String calculatorMode) {
    switch(calculatorMode) {
    	case "plus":
          return a+b;
          break;
        case "minus":
          return a-b;
          break;
        case "time":
          return a*b;
          break;
        case "divide":
          return a/b;
          break;
    }
  }
}
```

이와 같은 로직이 있다고 가정 했을 때 ~~물론 로직이 간단하니 실제론 안 그렇겠지만~~ 어떠한 연산 결과가 나왔을 때 잘못된 값이 나왔다고 가정을 해보자.
그렇다면 실제로 어떤 연산을 통해 그러한 오류 값이 나왔는지 파악하기 어렵다.

사실 이 계산기 로직의 경우는 굉장히 간단하기 때문에 로그를 찍을 수 있다면 calculatorMode 에 따라 code refactoring 을 진행하면 된다.

하지만 하나의 계산이 1000line 정도를 가지고 있다면, 단순히 log 만 가지고는 확인이 불가능할 것이다.

위 code 를 SRP 를 적용한다면 이와 같이 바뀔 수 있을 것이다.

```java
public class Calculator {

  public int calculator(int a, int b, String calculatorMode) {
    switch(calculatorMode) {
    	case "plus":
          return add(a, b);
        case "minus":
          return minus(a, b);
        case "time":
          return time(a, b);
        case "divide":
          return divide(a, b);
        default:
          return 0;
    }
  }
  
  private int add(int a, int b) {
    return a+b;
  }

  private int minus(int a, int b) {
    return a-b;
  }

  private int time(int a, int b) {
    return a*b;
  }

  private int divide(int a, int b) {
    return a/b;
  }

}
```

로직이 간단해서 딱히 나눠야 하나? 싶은 생각이 들텐데, 가장 직관적으로 보여주기엔 로직이 간단해야 하기에 계산기로 표현을 하였다.

사실 각 사칙연산 당 하나의 class 로 만들어 내는게 가장 SRP 에 적합한데 표현상 private 으로 구현하였다. 실제로는 각 부분을 담당하는 로직을 하나의 class 로 구현하여
모듈화를 시키고 그것에 대해 접근해서 사용하는 방법을 사용하는게 SRP 이다.

---

### OCP, Open-Closed Principle (개방-폐쇄 원칙)

method 자체는 개방되어 있으나 안에 구현체(implements)의 경우 외부로부터 closed 시키는 방식의 개발 원칙이다.

쉽게 말하자면 interface 의 다형성이 바로 이 부분에 해당한다.

```java
public interface OCPInterface {
  int polyTest();
}

public class OCPInterface_1 implements OCPInterface {
  public int polyTest() {
    System.out.println("test_1");
  }
}

public class OCPInterface_2 implements OCPInterface {
  public int polyTest() {
    System.out.println("test_1");
  }
}
```

각 class 가 따로 생성되어 있다고 가정 했을 때 interface 하나의 method 를 통해 별도의 로직을 구현 할 수 있다.
이러한 부분이 바로 open (OCPInterface) closed (각 class 의 변경 사항은 각 class 내에서 한정) 원칙이다.

---

### LSP, Liskov Substitution Principle (리스코프 치환 원칙)

MIT 의 리스코프라는 교수분이 제안한 원칙이다. 간단히 얘기하자면 class A 와 class B 중 A 가 B를 상속받아 재구현을 했다 가정 했을 때
A class 에서 정의된 method 또한 B 에서도 로직 손상 없이 사용할 수 있어야 한다는 것이다.

다음 예시를 보자.

```java
public class A {
  public void aMethod() {
    System.out.println("this is A method");
  }
}

public class B extends A {
  public void bMethod() {
    System.out.println("this is B method");
  }
}
```

이러한 class 가 있을 때 실제 class 의 명칭을 호출하는 로직이 비즈니스 로직이라 가정을 하자.

이럴 경우 당연하게도 B.aMethod() 는 호출 가능하다. compile 적으로도 이상이 없지만 문제가 하나 있다.

앞서 이야기했듯이 이 class 들의 비즈니스 로직은 나를 호출하는 class 명을 출력하는 것이다. 그러나 현재 구조를 보면 B.aMethod 를 호출할 경우 A 가 호출되었다 가 출력된다.
이런 경우 비즈니스 로직을 위배하게 되었고 OOP 에서 객체 지향의 객체라는 선을 넘어버리는 설계인 것이다.

이러한 부분들을 지양하자는 것이 리스코프 치환 원칙이다.

---

### ISP, Interface Segregation Principle (인터페이스 분리 원칙)

이 원칙은 큰 하나의 interface 내의 대량의 method 를 구현하는 방식보단 하나의 interface 내에 하나의 method 형식으로 잘개 쪼개는 원칙을 의미한다.

어찌 보면 SRP 가 적용되는 것으로 보일 수 있는데 대표적인 ISP 가 적용된 부분은 [functional interface](https://jungqui.github.io/posts/lambda/) 라고 볼 수 있다.

가령 위의 계산기 code 를 다시 보자면 private 형식으로 하나의 class 에 녹여낸 부분을 각 사칙연산별로 4개의 interface 로 구성하고 각 로직의 책임은 각 interface가 지는 식으로
구성할 경우 ISP 원칙이 적용되었다 볼 수 있다.

---

### DIP, Dependency Inversion Principle (의존 역전 원칙)

이 원칙은 OCP 와 비슷한 개념으로 볼 수 있다. 다음 예제 코드를 한번 보자.

```java
public class MainClass {
  public void main(String[] args) {
    NonDIPClass nonDIPClass = new NonDIPClass();
    nonDIPClass.methodA();
  }
}


public class NonDIPClass {
  public void methodA() {
    System.out.println("method A called");
  }
}
```

이와 같은 로직이 있다고 가정할 때 methodA 로직이 변경되어 그 때 그 때 다른 문자열을 출력하게 수정되어야 한다고 가정한다면
methodA 를 수정해야 한다. 하지만 interface 를 통해 DIP 를 적용한다면

```java
public class MainClass {
  @Autowired
  @Qualifier("contextMethodA")
  private DIPInterface dipInterfaceA;
  
  @Autowired
  @Qualifier("contextMethodB")
  private DIPInterface dipInterfaceB;


  public void main(String[] args) {
    dipInterfaceA.methodA();
    dipInterfaceB.methodA();
  }
}


public interface DIPInterface {
  void methodA();
}


public class contextMethodA implements DIPInterface {
  public void methodA() {
    System.out.println("method A called");
  }
}

public class contextMethodB implements DIPInterface {
  public void methodA() {
    System.out.println("changed business logic method called");
  }
}
```

이와 같이 DIP 를 적용하면 기존 로직에 대한 변화 없이 새로운 로직을 덧씌워 사용이 가능하다. 이를 고수준 모듈이 저수준 모듈에 의존하는게 아닌, 저수준 모듈에서 고수준 모듈에 대해 의존성을 주입한다 하야
DI(Dependency Injection) 이라고도 한다.

IoC, DI, DIP 모두 비슷한 개념으로 봐도 무방하다.

일부로 @Autowired 를 쓴 이유는 Autowired 가 가장 대표적인 DIP 가 적용된 훌륭한 case 이기 때문이다.

## 그래서?

꾸준히 보면서 느꼈겠지만 SOLID 내의 모든 원칙들이 상호 보완적으로 이루어져 있다. 예컨데, OCP 와 DIP 가 밀접하게 연관되어 있기에 하나를 추구하다 보면
자연스럽게 하나가 적용이 되는 식이다.

그 이유는 모든 SOLID 가 결국 OOP 라는 큰 테두리 안에 있고 결과적으로 모든 SOLID 원칙이 OOP 를 추구하고 있기 때문이다.

실제 실 개발의 모든 case 에 적용하기엔 사실 무리가 있을 것이다. 예컨데 도저히 business logic 이 OOP 를 추구할 수 없는 상황이 올 수 있고
잘 알고 있다는 것과 잘 사용하는 것 또한 다르기 때문에 적응하기도 어려울 수 있다.

다만 실제로 적용하고 나면 유지보수 측면에서 확실한 pay-back 이 있을 것이다.