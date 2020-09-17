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
2. build 시가 아닌 application 구동 후 해당 method 호출 시 error 확인 가능

이러한 단점이 있다.

string 으로 만들어야 하는 native query 의 이러한 단점을 커버하기 위해 만들어진게 queryDSL 이다.

## 왜 알아야 하나?

builder pattern 과 비슷하게 query 자체를 method 등을 이용해서 구성을 하기 때문에 가독성 측면에서 유리하며
다른 ORM 을 이용해 query 를 호출했을 때 dialect 등을 이용해 query 자체가 무거워지는 ORM 과는 다르게 실제로 mapping 된 method 에 맞게
query 가 구성되기 때문에 성능 저하 측면이 없다.

다만 목적성 자체가 검색 및 조회에 특화되어 있는 기술이기 때문에 insert 와 update 등으로는 구성되어 있지 않다.

## 구성

계속 집필...

## 그래서?