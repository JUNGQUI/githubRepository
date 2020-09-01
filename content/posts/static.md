---
title: "Static"
date: 2020-09-01T20:47:06+09:00
draft: true
---

# static
## static?
보통 '정적' 이라는 단어가 붙어 있는 case 가 있는데 정적 variable, 정적 method 을 지칭한다.

정적 이라는 말에서 알 수 있듯이 선언 직후 변하지 않는 값을 표현할 때 static 을 사용한다.
비슷한 의미로 final 이 있는데, final 은 `immutable` 속성을 가지고 있는 것이고 static 은 `고정적인` 의 속성을 가지고 있다고 보면 된다.

참고로 `enum class` 를 확인해보면 ~~java 가 아니라 class~~ 선언된 enum 이 static final 로 선언되어 있는 것을 알 수 있다.

static 의 경우 변경을 가하면 변하지만 final 의 경우 초기화 이후엔 값이 변하지 않기 때문에 두 가지를 조합하고 static 의 memory 내의 정적으로 존재하는 것을 확인 할 수 있다.

## 왜 알아야 하나?

~~면접..~~static method 와 static 변수를 사용할 때 왜 쓰는지와 어떤 영향을 끼치는지에 따라 굳이 사용할 지 안할지를 선택 할 수 있는 지식이 생기는 셈이기 때문에
알아야 한다.

잘 모르고 사용하면 쓸데없는 memory 낭비가 발생 할 수 있다.

## 구성

## 그래서?
