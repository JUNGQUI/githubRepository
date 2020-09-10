---
title: "Stream"
date: 2020-09-08T15:07:43+09:00
draft: true
tags: ["programming", "java"]
---

## Stream?
사전적 의미로는 개울을 의미한다. 그래서 '정보가 개울의 물처럼 연속적으로 흐르는 것' 이라는 느낌으로 data stream 등으로 이름을 지었다고 한다.~~카더라~~

stream 은 주로 연속적인 data 에 대해 제어하는 기능이 담겨져 있고 java 8에서부터 도입된 기능이다.

## 왜 알아야 하나?

java 8 이전에는 Collections 에 대해서 제어를 하기 위해서 for 문을 돌려서 제어를 했어야 했다.

이렇게 할 경우 내부의 제어 logic 이 복잡해진다면 그럴수록 code 가 비대해지고 가독성이 떨어지게 된다. 또한 business logic 자체가 섞이게 될 수 있고 이는 유지보수의 복잡성을 가져온다.

lambda 와 마찬가지로 이 또한 함수형으로 간결하게 그리고 in-line code 형식으로 만들기 위해 stream 이 도입되었다.

## 구성
크게 3가지 단계로 구성되는데 `생성`, `연산`, `반환` 으로 볼 수 있다. (이 부분은 개인적으로 정의한 용어다. 개념이 저렇게 구성된다고 보면 되겠다)

- 생성

우선 실행하기에 앞서 생성이 되어야 하니 생성을 해야 한다.

``` java
// builder pattern
Stream<String> builderStream = Stream.<String>builder()
                .add("A").add("B").add("C")
                .build(); // [A, B, C]

// Collections 로 생성 후
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

List<String> lang = Arrays.asList(
                "Java", "Scala", "Groovy", "Python", "Go", "Swift"
        );

// stream 으로 생성
Stream<Integer> numberStream = numbers.stream();
Stream<String> langStream = lang.stream();

// array 생성 후
String[] arr = new String[]{"a", "b", "c"};
Stream<String> stream = Arrays.stream(arr);

Stream<String> streamOfArrayPart = Arrays.stream(arr, 1, 3); // 1~2 요소 [b, c]
```

이렇게 생성하는 방법이 있는데, 취향에 맞게 생성자로 사용해서 쓰면 되겠다. 사실 후술할 map이나 reduce 등을 이용해서 바로 생성과 동시에 연산 후 반환하는 형식으로 사용하는 것이
stream 과 lambda 를 이용한 취지에 적합한 방식이다.

- 연산

stream 으로 생성이 된 후에 데이터에 대해 연산을 수행하는 부분이다.

연산에 대해서는 다양한 것들이 많다. iterator, map, flatmap, filter, sorting 등이 있다. 

#### filter

말 그대로 filter 를 걸어서 조건에 맞는 element 만 뽑아내어 stream 으로 연산을 하는 작업이다.

``` java
Stream<String> stream = lang.stream()
                            .filter(langName -> langName.contains("a"));
```

이후 이렇게 열린 stream 을 통해 다시 연산을 진행하거나 그대로 값을 collections 로 반환할 수 있다.

#### iterator

``` java
// [15, 17, 19, 21, 23]
Stream<Integer> iteratedStream = Stream.iterate(15, n -> n + 2).limit(5);
```

반복적인 작업으로 stream 을 연산하며 return 해줄 수 있다. 주의해야 할 점은 stream 크기는 기본적으로 infinite 이고 iterate 에는 크기 제한에 대한 조건문이 없기 때문에
limit 를 통해 크기를 제한해줘야 한다.

#### map

map 에서는 비로소 여러가지 직접적인 조작이 가능해진다. 예컨데 받은 값에 추가로 어떤 조작을 가하고 return 해주거나, 특정 조건의 값만 받아서 조작을 가한 후 return 하는 등
주로 쓰이는 기능 ~~(본인은 그렇다)~~ 이다.

``` java
// [1, 2, 3, 4]
IntStream intStream = IntStream.range(1, 5);

IntStream newIntStream = intStream.map((element) -> {
    if (element > 2) {
        return element+1;
    } else {
        return element-1;
    }
});

// 삼항연산자
IntStream newIntStreamThree = intStream
                .map((element)-> element > 2 ? element-1 : element + 1);
```
사실 위의 code 를 돌려보면 삼항연산자 부분은 exception 이 발생한다. 그 이유로는 newIntStream 에서 intStream 으로 이미 값을 받아 해당 stream이 닫혔기 때문이다.

#### sorting

정렬 연산이다. 기본적으로는 오름차순 배열이며 comparator 재정의를 통해 조작도 가능하다.

``` java
List<String> lang = Arrays.asList(
                "Java", "Scala", "Groovy", "Python", "Go", "Swift"
        );

// [Go, Groovy, Java, Python, Scala, Swift]
// 오름차
List<String> comp1 = lang.stream().sorted()
        .collect(Collectors.toList());

// [Swift, Scala, Python, Java, Groovy, Go]순
// 내림차
List<String> comp2 = lang.stream().sorted(
        Comparator.reverseOrder())
        .collect(Collectors.toList());

// [Go, Java, Scala, Swift, Groovy, Python]순
// string 길이 오름차순
List<String> comp3 = lang.stream().sorted(
        Comparator.comparingInt(String::length))
        .collect(Collectors.toList());

// [Groovy, Python, Scala, Swift, Java, Go]
// 재정의, string 길이 내림차
List<String> comp4 = lang.stream().sorted(
        (s1, s2) -> s2.length() - s1.length())
        .collect(Collectors.toList());
```

- 반환

[Stream doc](https://docs.oracle.com/javase/8/docs/api/?java/util/stream/Stream.html) 내에 `reduction` 부분을 보면 `전체 stream 에 대해 반복되는 연산을 통해 하나의 결과로 도출 시키는 method` 라
설명하고 있다. 그리고 이러한 reduction 에 포함되는 method 는 reduce 와 collect 로 구성되어 있다.

#### reduce

sum, max, min 등의 연산 작업을 수행하는 method 이다. 초기에 초기 값을 설정할 수 있고 재정의하여 lambda expression 으로 구현할 수 있다.

``` java
// [1, 2, 3, 4]
IntStream intStream = IntStream.range(1, 5);
List<Integer> newintCollections = intStream.boxed()
                                            .collect(Collectors.toList());

// 전체 stream 의 합을 구하는 lambda expression 사용
// 초기값이 없기 때문에 null이 반환 될 수 있고 그렇기에 optional 로 반환해야 한다.
// 결과 값 : 10
Optional<Integer> optionalInt = newintCollections.stream()
                    .reduce((element1, element2) -> element1 + element2);

// 초기 값 10이 주어지고 sum 을 진행
// 결과 값 : 10 + 10(1+2+3+4) = 20
int identityInteger = newintCollections.stream().reduce(
        10,
        Integer::sum,
        (element1, element2) -> {
            System.out.println(element1 + " + " + element2);
            return element1 + element2;
        });
```

#### collect

말 그대로 결과를 수집하는 method 이다. stream 연산이 끝난 값들에 collections 로 wrapping 하여 결과로 반환해준다.

앞서 사용했던 code 를 다시 한번 보자면

``` java
List<String> lang = Arrays.asList(
                "Java", "Scala", "Groovy", "Python", "Go", "Swift"
        );

// [Go, Groovy, Java, Python, Scala, Swift]
// 오름차
List<String> comp1 = lang.stream().sorted()
        .collect(Collectors.toList());

// [Swift, Scala, Python, Java, Groovy, Go]순
// 내림차
List<String> comp2 = lang.stream().sorted(
        Comparator.reverseOrder())
        .collect(Collectors.toList());

// [Go, Java, Scala, Swift, Groovy, Python]순
// string 길이 오름차순
List<String> comp3 = lang.stream().sorted(
        Comparator.comparingInt(String::length))
        .collect(Collectors.toList());

// [Groovy, Python, Scala, Swift, Java, Go]
// 재정의, string 길이 내림차
List<String> comp4 = lang.stream().sorted(
        (s1, s2) -> s2.length() - s1.length())
        .collect(Collectors.toList());
```

이와 같이 .sorted() 로 연산이 끝난 후에 .collect(Collectors...) 와 같은 형식으로 반환할 타입을 지정해준다.
java 7에서 diamond operator 가 생겼기 때문에 list 안의 type 에 대해서는 반환 될 list 내에서 정의가 되어 자동으로 값들을 해당 type 에 맞게 wrapping 해준다.

## 특징

지금까지 method 에 대해 살펴봤는데, 중간 중간에 `이래서 안됩니다.` 와 같은 부분이 몇 있었다.~~눈치 못채셨다면 죄송..~~
그러한 부분에 대해 추가적인 stream 의 특징에 대해 알아보자.~~ARABOJA~~

- 일회성

가장 중요한 특징이라 할 수 있는데, stream 은 data 를 담아두고 재사용하는 목적으로 개발된 API 가 **아니다.** 그렇기 때문에 한번이라도 연산이 끝났다면 다시 사용할 수 없다.
연산이 종료되는 순간 해당 stream 이 닫히기 때문이다.

``` java
// 각 요소 반복 시 print
intStream.forEach(System.out::println);
// 위에서 반복 후 stream close 되었기에 접근 시 InvocationTargetException 발생
intStream.sorted();
```

- null-safe

`reduce` 에서 초기 값이 주어지지 않았을 경우 Optional 로 지정해줘야 한다고 했던 부분이 있다. 이와 같이 NPE 가 발생할 수 있을 상황(reduce sum 시 초기 값 없을 시와 같은)에서는
method 자체가 return Optional<T> 로 지정이 되어있다. 이와 같은 경우에 대해 method 가 지정되어 있기 때문에 null 에 대해 safe 하게 구현 할 수 있다.

>물론 그럼에도 불구하고 비어있는 stream 에 대해 .max() 연산을 할 경우엔 NPE 가 발생 할 수 있다.

## 그래서?

기존에 사용하던 복잡한 for 문 구조와 단순한 로직에 비해 방대한 코드양에 질렸다면 stream 을 도입해보자.
stream 자체를 그냥 보기에는 가독성이 좋지 않다고 말할 수 있겠으나 적응이 되고 난 후에 stream 으로 짜여진 코드 전체를 본다면
오히려 이전 대비 가독성이 높아졌을 것이다.

또한 이러한 stream 으로 refactoring 하는 와중에 불필요했던 부분에 대해 정리도 되는 계기가 되지 않을까 싶다.

확실한건 앞으로도 stream 과 collection 은 적어도 알고리즘 test 에서는 아주 요긴한 무기가 될 것이니 알아둬서 나쁘지 아니하지 겠는가?

참고 글 : https://futurecreator.github.io/2018/08/26/java-8-streams/

> source code 이자 test code 는 [여기](https://github.com/JUNGQUI/spring/blob/master/src/test/java/com/jk/spring/JKTestNotePad.java) 에 있다.