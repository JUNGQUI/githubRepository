---
title: "Graphql"
date: 2020-09-21T20:58:34+09:00
draft: true
tags: ["DB", "query", "language"]
---

## graphQL?

facebook 에서 개발한 쿼리 언어이다.

그렇다면 진행하기 앞서 쿼리 언어란 무엇인가에 대해 알아봐야 할 것이다.

가장 익숙한 쿼리 언어는 SQL 일 것이다. Structured Query Language 의 약어인 SQL 은 우리가 흔히 쓰는

DB 접근 및 제어 언어이다. 단어 뜻으로 보면 `구조화 된 질의 언어` 라는 건데 결국 주요 포인트는 DB 제어 라는 거다.

많은 RDB, DB 가 SQL 을 표준으로 선택하고 있을 정도로 보편화되어 있다. 그러나 그 대항마로 최근에 뜨기 시작한게 GraphQL (이하 gql) 이다.

## 왜 알아야 하나?

SQL 과 다른 점은 전통적인 controller 방식이 아닌, object 구조와 gql 정의 spec ~~임의 명시~~ 을 통해 controller 역할을 resolver 가 해준다는 점이다.

이러한 부분을 통해 FE - BE 간 협업 또한 쉬워지고 controller 를 일일이 구축하고 controller 에 신경을 쓰는 시간을 business logic 에 더 신경 쓸 수 있게 된다.

개인적인 의견이지만, test 용 controller (mocking controller) 를 만드는 부분이 정통적인 controller 환경보다 훨씬 우수하다고 생각된다.

그 이유는 아래에 서술한다.

## 구성

gql 은 크게 type / input / Query / Mutation 으로 나뉘어진다. 
type 의 경우 gql 내에서의 class 선언이라고 볼 수 있다.
input 의 경우 type 과 동일하지만 Mutation 을 할 경우 사용하는 type 의 일종이라 볼 수 있고, 
Query 의 경우 조건에 맞게 select sql,
Mutation 의 경우 insert, update, delete sql 로 생각하면 된다.  

우선 gradle (혹은 maven) 을 통해 필요한 것들을 import 해야 한다.

(본 글은 gradle 로 진행했기에 예시로 gradle, spring-boot 를 이용했다.)

```java
implementation 'com.graphql-java-kickstart:graphql-java-tools:6.2.0'
implementation 'com.graphql-java-kickstart:graphql-spring-boot-starter:7.1.0'
runtimeOnly 'com.graphql-java-kickstart:graphiql-spring-boot-starter:7.1.0'
testImplementation 'com.graphql-java-kickstart:graphql-spring-boot-starter-test:7.1.0'
```

```graphql
schema {
    query: Query
    mutation: Mutation
}

type Mutation {

}

type Query {

}
```

`schema.graphqls` file 을 통해 query 와 mutation 을 root 로써 정의해준다. 이렇게 root 로 정의해줌으로써 graphQL 이 정상적으로 작동하게 된다.

위 두 가지가 중요한 설정 값이다. 이것만 설정해주면 모두 끝난 것이다. ~~참 쉽죠?~~

이제 필요한 부분이 생겼을 때 graphQL file 을 생성해서 작성해주면 된다. 간단한 예시를 하나 들어보자.

```java
public class GqlUserInformation {
  private Long userId;
  
  private List<String> tag;

  private String name;
  private String password;

  private Intger notInclude;
}
```

이와 같은 object 가 있다고 가정하자. 물론 gql 용 Object 가 별도로 존재하는 상태에서 DAO 는 별도로 존재한다고 가정한다.

```graphql
type GqlUserInformation {
    userId: ID
    tag: [String]
    name: String
    password: String
}
```

gql 로 표현하면 이와 같이 설정 할 수 있다. 물론 gql 내에 포함되지 않은 `notInclude` 변수의 경우 query 를 통해서 가져올 수 없다.

```java
public class QueryGqlUserInformation {
  public GqlUserInformation findById(Long id) {
    ...
    SOME_LOGIC
    ...
  }
}
```
이와 같은 service 에 대해 gql query 를 통해 한다면

```graphql
extend type Query {
    findById(id: ID): GqlUserInformation
}
```

이와 같이 작성하면 된다. 이렇게 쉽게 작성이 가능하고 controller url mapping 없이 object 구조와 gql 정의만으로도 service 접근이 가능하기 때문에
front-end 개발자가 gql 을 관리하고 back-end 개발자는 해당 spec 을 통해 service 를 만들어 제공해주는 식으로 개발을 진행 할 수 있다.

이와 같은 점이 앞서 말했듯이 test 용 controller 를 생성하는 기존 방식보다 훨씬 효과적이라 볼 수 있는게, business logic 을 service 에 다 만들고
난 후에 unit test 를 통해 service 를 검증 한 뒤 controller 에 또 추가 logic (validation 등)을 구성하고 다시 controller test 를 진행해야 하는
전통적인 controller 방식보다 더 개발하기 쉽다고 볼 수 있다.

완성된 Query gql 의 경우 다음과 같다.

```graphql
type GqlUserInformation {
    userId: ID
    tag: [String]
    name: String
    password: String
}

extend type Query {
    findById(id: ID): GqlUserInformation
}
```


## 그래서?

나온지 제법 되었음에도 gql 을 사용하는 거나 gql 을 이용한 open source 는 사실 아직 찾기 힘들다.
다만 최근 상승세나 검색어 트렌드 등을 확인해보면 확실히 gql 이 많이 자리를 잡아가고 있는 것도 사실이고
MSA 와 같은 것들을 보면 확실히 개발 역할군 자체가 분할되어 가는데 이런 경향에 gql 은 확실히 훌륭한 무기로 볼 수 있다.

물론 learning-curve 도 심한 편이고 (기존 패러다임이 깨져버리기 때문에) 많이 사용되지 않았기 때문에 레퍼런스도 많이 없는 편이다.

하지만 gradle import 를 보면 알 수 있듯이 spring 부터 시작해서 node, react 등 많은 framework 에서도 gql 에 대해 잘 쓸 수 있게
좋은 module 들이 늘어나는 추세이다.

한창 module 이 많아지고 나서 시작하기엔 curve 가 확실히 심한 편이니 미리 공부해둬서 migration 을 준비해보는건 어떨까? 