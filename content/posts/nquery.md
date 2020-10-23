---
title: "Nquery"
date: 2020-10-23T21:58:01+09:00
draft: true
tags: ["DB", "JPA"]
---

## N+1 query?

문자로만 보면 N 번의 쿼리를 날릴 때 +1 회 쿼리가 발생한다는 거고, 그게 문제라는 거다.

기본적으로 JPA 환경에서 lazy fetch 를 사용하여 sub relation 이 있는 object 를 가져올때 문제가 된다.

## 왜 알아야 하나?

join 은 개발하면서 피할 수 없는데, 제대로 알지 못하고 막무가내로 쓰면 performance 에 영향이 있다.

최근 들어 발견한(?) 이슈인데, co-op 을 하다 보니 다른 개발자가 만든 obj 를 쓸 일이 있었는데, join 에 lazy 가 있는 부분에 대해서
미처 파악하지 못하고 fetch 를 걸지 않은 부분이 있었다.

혹여나 이 글을 읽는 분들은 이런 일이 벌어지지 않기를 바라며, 당연히(!) 알고 있겠지만 상기하시길 바란다.

## 구성

위에서 언급했듯이 lazy 로 걸고 fetch 없이 단순 join 만 걸 경우 n+1 문제가 발생한다.

> - lazy?
>
>JPA 는 persistence 를 통해 data 를 관리하는데, relation 이 걸려있는 부분에 fetch mode 를 걸 수 있다.
>
>기본적으로 관계에서는 eager 로 걸려 있고 별도로 fetch = FetchType.LAZY 와 같은 방식으로 걸 수 있다.
>
>eager 일 경우 relation 을 전부 가져와서 영속성으로 관리하고, lazy 일 경우 relation 이 걸려 있는 object 를 실제로
>사용하려고 할 때 query 를 보내 object 를 가져온다.
>
>```java
>public class A {
>  private String name;
>  private String content;
>}
>```
>```java
>public class B {
>  private String name;
>  private String content;
>
>  @ManyToOne(fetch = FecthType.LAZY)
>  private A a;
>}
>```
>
>이와 같은 class 가 있을 경우 B를 사용하되, A 를 실제로 code 내에서 호출하지 않으면 A 는 실제로 join 하여 결과를 가져오지 않는다.
>
>다만 만약 B.getA() 와 같이 사용하게 될 경우 그 시점에 query 를 통해 data 를 가져온다.

이와 같은 특성이 있어서 n+1 query 가 발생하곤 한다.

이럴 경우 fetch join 을 사용하여 해당 문제를 방지 할 수 있다.

```java
@Query(value = "SELECT * FROM B FETCH JOIN A ...")

...
public void queryDSL() {
  queryFactory.selectFrom(QB.b)
    .join(QB.b.a, QA.a)
    .fetchJoin()
    .fetch()
    ...
}
```

위와 같이 queryDSL 일 경우 `fetchJoin()` 이라는 method 가 있고 @Query 를 사용 할 경우 `FETCH JOIN` 을 이용 할 수 있다.

## 그래서?

join 은 생각보다 많이 쓰이면서 생각보다 많은 부분에 performance 에 영향을 끼친다. 제일 큰 문제는, 문제가 되기 전까진 이 부분들이
문제가 안된다는 것이다.

초기 개발 시기나 서비스 초기에는 많은 data 가 없으니 문제가 없는 것처럼 보이지만 실제로 조금만 data 가 쌓이기 시작하면 갑자기 모든 performance 에 문제가 생기는데,
이쯤되면 이미 서비스 중인 product 에 개보수가 어렵기 때문에 초기 설계 및 개발 시 부터 글로벌 페치 전략이나 각 relation 에 대해 조심스럽게 사용해야 하고
특히 JPA 를 이용한다면 더더욱 설계 및 relation 에 조심히 개발해야 한다.

참고는 [여기](https://jojoldu.tistory.com/165) [저기](https://blog.leocat.kr/notes/2019/06/01/querydsl-avoid-n-plus-one-issue-with-fetch-join) 참고 했다.