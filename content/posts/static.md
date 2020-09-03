---
title: "Static"
date: 2020-09-01T20:47:06+09:00
draft: false
tags: ["programming", "java"]
---

## static?
보통 '정적' 이라는 단어가 붙어 있는 case 가 있는데 정적 variable, 정적 method 을 지칭한다.

정적 이라는 말에서 알 수 있듯이 선언 직후 변하지 않는 값을 표현할 때 static 을 사용한다.
비슷한 의미로 final 이 있는데, final 은 `immutable` 속성을 가지고 있는 것이고 static 은 `고정적인` 의 속성을 가지고 있다고 보면 된다.

참고로 `enum class` 를 확인해보면 ~~java 가 아니라 class~~ 선언된 enum 이 static final 로 선언되어 있는 것을 알 수 있다.

static 의 경우 변경을 가하면 변하지만 final 의 경우 초기화 이후엔 값이 변하지 않기 때문에 두 가지를 조합하여 사용하면 application 이 올라가져 있는 상태에서 독립적으로 존재 (static)
하며 선언 직후 일관된 값(final)을 가진 enum 이 완성되는 것이다.

## 왜 알아야 하나?

~~면접..~~ static method 와 static 변수를 사용할 때 왜 쓰는지와 어떤 영향을 끼치는지에 따라 굳이 사용할 지 안할지를 선택 할 수 있는 지식이 생기는 셈이기 때문에
알아야 한다.

잘 모르고 사용하면 쓸데없는 memory 낭비가 발생 할 수 있다. 예컨데, 간단한 설정을 위해 무분별한 static 변수를 남용한다면 해당 영역은 (stack 에 저장됨) gc가 관여하지 않기 때문에 일정한 memory 가 쌓이게 되므로 낭비가 발생한다.

## 구성

우선 [Memory에 대한 구조](https://jungqui.github.io/posts/jvm) 를 알아야 한다.

JVM 에서 method 가 실행되는 순서를 보면 최초 method 호출 시 method 내의 변수들에 대한 정보가 stack 에 쌓이게 된다.

이후 해당 method 에서 다른 method 를 호출하게 되면 다시 위의 과정을 반복해서 첫번째 stack 위에 새 stack 을 쌓게 된다. 이렇게 쌓이면서 결국 method 가 완료되어 memory return 이 일어나면 해당 stack 은 삭제된다.

헌데, 이러한 방식일 때 `정적으로 관리되어야 할` static 변수를 stack 에 저장하면 언젠가 날아가기 때문에 저장 할 수 없다.

고로 method area 라는 영역에 반영구적으로 저장하여 사용한다. 

보통 static variable (변수) 과 static method 가 있다.

정적으로 선언되어 application 이 종료 전 까지 `공용`으로 접근해서 사용 가능하기 때문에 환경 변수 및 util method 형식으로 사용 가능하다.


## 그래서?

지금까지 이야기 했듯이 해당 값은 반영구적으로 사용이 가능하다. 그 말인 즉슨 항상 application memory 내에 일정한 공간을 차지하고 있다는 말이며, 이 값이 커질수록
application 이 비만이 오기 때문에 performance 측면에서 issue 가 발생할 수 있고, 얘기치 못하게 꼬여서 memory leak 이 발생할 수 있다. (GC 로 관리되지 않는 영역이기 때문에)

잘 사용하면 DB 갈 필요 없이 환경 변수로써 사용할 수 있고 흔히 쓰는 방식으로 util 성 class 를 만들어 bean 으로 만들 필요 없이 간단하게 사용도 가능하지만 항상 편하게 사용하는게 만능은 아니고 자원적 issue 가 발생 할 수 있기 때문에 주의해서 사용하길 바란다.