---
title: "Querydsl"
date: 2020-09-16T22:10:56+09:00
draft: true
tags: ["java", "spring", "ORM", "JPA", "DB"]
---

## queryDSL?

배경은 ORM 에서 시작했다. 

기본적으로 ORM 은 Object-Relation Mapping 이다. DB 를 query 로 접근하는 전통적인 방법이
java 의 OOP 와는 맞지 않아 최대한 Query 를 자제하고 결과를 Object 로 반환 받아 사용하는 기법(?) 이다.

java 개발자들에겐 복잡한 query 를 생각하지 않고 진행하면 된다는 점에서 한 줄기 빛과 같았는데 아주 치명적인 단점이 있었으니
복잡한 query 이거나 data 가 급격하게 많아질수록 엄청난 performance 저하가 발생한다.

그래서 나온 것이 HQL 과 같은 것들인데, 실제로 HQL 을 할 경우 일반 query 처럼 사용하여 특수한 경우에만 performance 저하 없이 사용 할 수 있었다.

다만 이럼에도 불구하고 이렇게 사용할 경우 String 을 통해 query 를 만들기 때문에

1. 가독성 저하, co-op 어려움
2. type-safe 하지 않음

> #### Type-safe?
>
>비논리적 계산이 `합리적으로 계산이 가능 할 때` 계산을 할 때 Type-safe 하지 **않** 다고 한다.
>예를 들어 보자.
>
>```javascript
>console.log('print 4 time ' * 4)
>>NaN
>```
>javascript 의 경우 이와 같이 NaN 이 출력될지언정 입력 시엔 (compile) 아무런 반응이 없다.
>
>반면, java 의 경우
>
>```java
>System.out.println("It Must Print 4 time " * 4);
>```
>
>단번에 compile error 가 발생한다.
>
>이처럼 runtime 시 error 가 발생하는 경우 type-safe 하지 않고
>compile 시점 시 error 가 발생하는 경우 type-safe 하다고 판단한다.
>
>비슷한 개념으로 checked Exception 과 unchecked exception 이 있다.

이러한 단점이 있다.

string 으로 만들어야 하는 native query 의 이러한 단점을 커버하기 위해 만들어진게 queryDSL 이다.

## 왜 알아야 하나?

builder pattern 과 비슷하게 query 자체를 method 등을 이용해서 구성을 하기 때문에 가독성 측면에서 유리하며
다른 ORM 을 이용해 query 를 호출했을 때 dialect 등을 이용해 query 자체가 무거워지는 ORM 과는 다르게 실제로 mapping 된 method 에 맞게
query 가 구성되기 때문에 성능 저하 측면이 없다.

다만 목적성 자체가 검색 및 조회에 특화되어 있는 기술이기 때문에 insert 와 update 등으로는 구성되어 있지 않다.

## 구성

중요한 구성은 총 2가지가 있는데, 첫째로 JPAQueryFactory 를 활성화 해서 criteria sesssion 처럼 사용하는게 중요하고,
두번째로 중요한 점은 실제 DB table 과 mapping 하기 위한 QObject 가 중요하다.

- JPAQueryFactory

```java
public class QuertDSLTest {
    ...
    // JPAQueryFactory 에 entityManager 주입
    // entityManager 를 받은 상태로 bean 으로 생성해서 주입받는 식으로 사용하기도 한다.
    JPAQueryFactory queryFactory = new JPAQueryFactory(em);
    
    // Member 의 Query Object 인 QMember 를 생성
    QMember member = QMember.member;
    
    // Member 를 찾을 때 queryFactory 에서 QMember 에서 (from) 가져오게 설정
    // 이후 where 조건 등 다양하게 사용
    Member foundMember = 
        queryFactory.selectFrom(member) // select + from
        .where(customer.username.eq("joont"))
        .fetchOne();
    ...
}
```

기본 구성은 이렇게 구성하고 query 자체는 query 를 method 에 mapping 하였다 생각하고 query 작성 하듯이
builder pattern 처럼 사용하면 된다.

결과적으로 query result 가 object 에 mapping 이 되기 때문에 사용하기에도 좋고 OOP 의 의도에 맞게 query 를 사용 할 수 있다.

무엇보다 이렇게 구성 할 경우 native 에 대비 business logic 에 더 집중 할 수 있어 생산성 증대에 도움이 된다~~고 한다~~.

- QObject

위의 QMember 와 같이 Query에 mapping 하기 위한 Object 이다. 최근엔 gradle 을 통해서 QObject 를 resource 처럼 자동으로 생성 할 수 있기 때문에
별도의 작업 없이 설정이 가능하다.

`queryFactory.selectFrom(member)` 처럼 QObject 를 queryFactory 에 mapping 한 뒤 이후 condition(where, join 등) 을 method 처럼 추가하여
사용이 가능하다.

criteria builder 에서 getCurrentSession 을 통해 object 를 설정하는 것처럼 queryDSL 에서는 QObject 를 통해 mapping 을 진행한다. 

## 그래서?

JPA, ORM 구조에서는 오직 ORM 만 가지고는 작업을 할 수 없다. 그 이유로는 **굳이 비유하자면** query 를 code 를 통해 자동으로 시스템이 유추하는 식이기 때문에
projection 등을 설정하지 않을 경우 불필요한 join 등이 붙을 수 있다. native query 로 최적화를 하기 힘든 구조이기 때문에 아무리 최적화를 하더라도 시스템적 한계가 있을 수 밖에 없다.

그렇다고 일부 복잡한 call 을 native 로 하자니 ORM 을 추구한 것에 정반대의 개념을 적용한 것이기 때문에 모순적인 부분이라 볼 수 있다.

하지만 queryDSL 을 이용할 경우 OOP 에 ~~크게~~ 위반되지 않는 선에서 합리적인 Native performance 를 가져 올 수 있는 이점이 있다.
사실 비슷한 개념으로 @Query 가 있긴 하다. 하지만 그렇게 할 경우 별도의 mapping 을 해야 하는 단점도 있고, 유지보수 측면에서 OOP 에 적합하다고 볼 수 없기 때문에
queryDSL 을 익숙하게 사용 할 줄 알아야 한다.

