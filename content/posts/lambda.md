---
title: "Lambda"
date: 2020-09-07T13:38:49+09:00
draft: false
tags: ["programming", "java"]
---

## Lambda?

대표적으로는 그리스어의 람다가 있다. 

그리고 수학적으로는 람다식이라는 것이 있는데 이 부분이 개발에서 쓰이는 lambda expression 과 유사하다고 볼 수 있다.

수학적 람다 식은 [여기](https://ko.wikipedia.org/wiki/람다_대수) 를 보면 알 수 있는데, 도입 - 핵심 개념 부분을 보면 `추상화` 라는 개념이 등장하는데 이 부분이 바로 개발쪽에서 lambda expression 을 쓸 때 interface 와 일맥상통 하는 부분이다. 

개발쪽에선 `익명 함수` 라는 말로 주로 쓰이는데 쉽게 말해 함수 선언 없이(익명) 바로 implement 하여 interface 의 장점을 극대화 시키는 함수형 프로그래밍 기법이다.

## 왜 알아야 하나?
java 8 의 가장 큰 변경점 이 lambda expression 구현이다. 또한 MSA 가 붐이 일면서 interface 를 사용하는 장점인 추상화, 다형성 등을 이용하기 좋은 장점이 있다.

java 8에서 공식적으로 지원하게 되고 전반적인 개발 구조에서 많이들 쓰이며 무엇보다, 문제를 해결하는 또 하나의 총알 하나 정도는 가져가야 추후에 선택지가 다양해지고 더 좋은 개발자가 되지 않겠는가?

## 구성
일단 진행하기에 앞서 일급 컬렉션에 대해 알아야 한다.

> - 일급 컬렉션?
>
>하나의 컬렉션을 랩핑 해서 구현하지만 그 외의 매개 변수는 존재하지 않는 object 를 의미한다.
>
>무슨 의미인지 보자면 특정 entity 용 object (DAO, VO, DTO 등) 를 생성 할 경우 해당 class 내에 name 이란 변수가 선언된다면
>선언된 name 외에는 다른 변수는 선언되지 않은 class 를 말한다.
>
>```
>public class FirstClassCollections {
>   private List<String> names;
>}
>```
>
>이렇게 구현할 경우 장점은 business logic 에 특화되기 때문에 해당 class 와 관련된 모든 logic 이 해당 class 안에 존재하기 때문에 생성 시 business logic 이 service layer 에 나갈 필요가 없어진다.
>
>이는 곧 코드의 간결함 및 가독성을 증대 시킨다.
>
>다만 단점으로는 모든 collections 을 일급 컬렉션으로 만들 경우 logic이 굉장히 복잡해지고 많아질 수 있다.

이러한 일급 컬렉션을 먼저 언급한 이유는 `람다 표현식` 이 함수를 일급 컬렉션으로 사용하지 못하는 상황을 대체하기 위해 등장했기 때문이다.~~두둥등장~~

또한 일급 컬렉션처럼 함수를 사용할 경우 일급 컬렉션의 장점인 side-effect 에 대해 safe 한 개발도 가능하며 iterate 한 부분에서도 simple 하게 구현이 가능하다.

이제 일급 컬렉션과 람다의 등장 이유를 알았으니 사용하는 방법을 알아보자 ~~ARABOZA~~ 먼저 그냥 다짜고짜 쓸 수는 당연히 없다. 우선 functional interface 가 존재해야 한다.
java 에서 함수를 일급 컬렉션으로 쓰고자 하는 목적이였으니 함수를 일급 컬렉션처럼 선언해줘야 한다.

이때 등장하는게 functional interface 인데, interface class 이면서 method 가 하나만 있는 interface 를 functional interface 라고 한다.

``` java
@FunctionalInterface
public interface JKLambda {
    String stringConcat(String s1, String s2);
}
```

이와 같은 interface 를 선언 후 사용 시에는 당연히 구현이 되어있지 않은 interface 이기 때문에 구현 (implement) 해서 사용해야 한다.

lambda 가 없을 경우는 아래와 같은 방식으로 구현해서 사용해야 했다.

``` java
JKLambda jkLambda = new JKLambda() {
    @Override
    public String stringConcat(String s1, String s2) {
        return s1 + " " + s2 + "JK lambda, no lambda";
    }
};
```

그러나 lambda expression 을 사용하면 아래와 같이 심플하게 구현이 가능하다.

``` java
JKLambda jKlambda = (a, b) -> a + " " + b + " JK lambda, use lambda";
```

딱 봐도 한눈에 가독성이 뙇 생기고 코드로 작성하는 line 이나 글자 수도 몹시 작아진다.

또한 iterate 상황에 대해서도 심플하게 구현이 가능하다.

``` java
List<String> list=new ArrayList<>();
list.add("1");
list.add("2");
list.add("3");
list.add("4");
list.add("5");

// no lambda
for(String element : list) {
    System.out.println(element);
}

// lambda
list.forEach(
    System.out::println
);
```

이와 같이 구현할 경우 object base 에 strict 한 구조를 만들 수 있기 때문에 immutable 에도 도움이 된다.

>주의해야 할 점은 **삼항 연산자** 와 **람다 표현식** 은 다르다는 점이다.
> 
>본인은 람다 표현식이 값을 return 하기 때문에, 또 `선언하지 않는 익명 함수` 이기 때문에 별도의 interface 없이 바로 사용 가능하다고 생각했다.
>
>하지만 공부를 하면서, 또 다시 곰곰히 생각해보니 그럴바엔 그냥 util 성 method 를 만들면 되는 부분이였기에 본인이 전혀 다르게 생각하고 접근했었던 것이다.
>
>개인적으로 본인과 동일한 생각으로 접근하는 분들이 있을 수 있다 생각하여 노파심에 작성해둔다.

## 그래서?

개발에 silver bullet 은 없다. 다만 항상 기존의 것과 새로운 것을 조합해서 더 효율적으로 개발하는 것이 우리가 지향해야 하는 점이라고 생각한다.
그러한 부분에 입각해서 새 기술을 쌓는 것에 대해 게을리 해서는 안된다고 생각한다.

지금 이 lambda 식도 사실 `legacy 에 전체 적용` 한다거나 `이게 좋으니 이것만으로 개발해야해!` 라는 접근법을 가질수도 없고 가져서도 안된다.~~본인이 그러했다~~

다만 늘 이야기 했듯이 기술 stack이 쌓여서 조금 더 효율적으로 풀어 나갈 수 있는 branch 하나가 생겨남으로 조금이라도 더 효율적인 개발 방식을 지향하는게 개발자들이
평생 공부해야하는 이유이지 않을까 싶다. 

performance 라는 단어 안에 program 의 속도, 효율성 뿐만 아니라 개발자간 상호작용과 유지보수 또한 포함되지 않는가?