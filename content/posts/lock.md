---
title: "Lock"
date: 2020-09-03T09:54:12+09:00
draft: false
tags: ["programming", "java"]
---

## Lock?

사전적 용어로 '잠금' 을 뜻한다. 그렇다면 왜 개발에서 lock 이라는 개념이 나왔을까.

우선 설명하기에 앞서 동시성 관련된 부분을 알아야 한다. [이전](https://jungqui.github.io/posts/processnthread/) log 를 보면 thread 기반으로 구성되어 있으며
해당 thread 는 stack 을 제외한 나머지 자원을 공유한다고 설명한 바 있다. 공유하는 이유는 context-switching 에 따른 over-head 를 줄이기 위함이고
이러한 방식을 취해 좀 더 빠른 실행을 할 수 있다고 설명한 바 있다.

그렇다면 이 부분에서 의구심이 들텐데, `공유 자원에 대한 동시 수정` 은 어떻게 되는가 라는 점이 남는다.

이러한 부분에 대한 해결책으로 lock 이라는 개념이 도입되었다.

## 왜 알아야 하나?
상식적으로 따져봤을 때 하나의 `update` 가 끝났다면 그 결과가 최종 반영이 되어야 한다.
하지만 동시에 접근 가능하게 허용 한 뒤 `update` 작업을 하게 한다면 일단 동시에 들어온 것 자체가 문제가 된다.

가장 많이 쓰이는 예가 바로 은행 입/출금인데, 100원에서 1원을 뺌과 동시에 1원을 더하면 값은 99 이거나 101 이 될 수 있다.

서비스가 커짐에 따라 performance 향상에 thread 를 도입했는데 동시성에 대한 부분을 제어하지 못해 전체 서비스가 망가지고 roll-back이 되어 ~~회사가 망하고 개발자가 실직하는~~ 안타까운 상황에 빠지기 전에 
이러한 부분에 대해 알고 써야 효율적인 thread 환경을 만들 수 있다.

## 구성
크게 synchronized 를 이용해 method 자체를 lock 을 거는 방법과 Reentrantlcok 두 가지 방법이 있다.

- synchronized 방식

```
public synchronized void checkSync(String threadId) {
    System.out.println("JK Thread in (synchronized) : " + threadId);
}
```

- Reentrant Lock 방식

```
ReentrantLock reentrantLock = new ReentrantLock();

public void checkReentrantLock(String threadId) {
    reentrantLock.lock();
    System.out.println("JK Thread in (Reentrant) : " + threadId);
    reentrantLock.unlock();
}
```

> source code 는 [여기](https://github.com/JUNGQUI/spring/blob/master/src/main/java/com/jk/spring/lock/JKLock.java) 에 있고
> test code 는 [저기](https://github.com/JUNGQUI/spring/blob/master/src/test/java/com/jk/spring/JKTestNotePad.java) 에 있다.

두 가지 모두 thread 의 접근 시 lock 을 이용해 타 thread 의 공유 자원의 침범에 대해 막는 역할을 한다.
(여담이지만, 이러한 공유 자원 영역을 critical section 이라 한다) 하지만 몇 가지 차이점이 있는데

- 공정/불공정

thread 는 lock 을 얻기 위해 순차적으로 critical section 에 접근한다. A, B, C thread 가 있다고 가정 할 때 A 가 선정 후 B는 해당 lock 이 점유되어 있기 때문에
받아가지 못하고 대기하는 사이 C 가 다음 순서로 들어올 때 A 가 마침 끝나게 되면 C가 접근 후 수행 하면서 lock 을 점유하게 되고 이는 B 의 기아상태를 야기한다.

이러한 부분에 대해 불공정하다, fairness 가 없다 라고 하는데 반면 ReentrantLock 의 경우 생성자로 생성 시 boolean 을 통해 fairness 를 true 로 주어 lock 을 얻기 위해
대기하고 있던 시간이 긴 thread 에 우선적으로 lock 을 제공해준다.

- tryLock() / lockInterruptibly()

두 기능 모두 ReentrantLock 에만 구현이 되어 있는데, 위의 개념과 동일하다. tryLock 의 경우 lock 을 얻을 수 있는지 확인 후 얻을 수 있으면 lock 을 실행, 아니면 다른 작업을 할 수 있게 해주는 method 이다.

정의를 보면 tryLock 의 경우 fairness 를 true 로 설정해도 nofair 하게 뚫고 들어가서 lock 을 얻기 때문에 사용에 주의를 해야 한다.
만약 fair 는 지키면서 사용하고자 한다면 tryLock(time, TimeUnit.second) 와 같은 형태로 사용하되 시간을 0으로 설정하면 되는데, 이 때 0초 동안 try 하기 때문에 최초 이후 실패 시 lock fairness 가 보장된다.

lockInterruptibly() 의 경우 3가지 case 가 있는데,
 
- lock 가능 시 즉각 hold count 를 1로 만들고 본인이 lock 을 획득한다. 
- 본인이 lock 을 점유 시 hold count++ 
- 타 thread 가 lock 점유 시 잠재적 sleep 상태 진입 후
    1. 해당 thread 가 lock 획득 시 깨어남
    2. 다른 thread 가 해당 thread 를 인터럽트 했을 시 깨어남

으로 나뉜다.

이렇게만 작성하면 synchronized 가 불편하기만 하고 장점이 없는데요? 라고 할 수 있는데 일단 ReentrantLock 의 경우
exception handling 이 필요하기 때문에 try / final 로 구문이 늘어지게 되어 business logic 이 묻히게 되는 단점이 있다.

또한 가장 큰 단점은 human error 인데, 예를 들어 실수로 unlock 을 하지 않을 경우 엄청난 사태가 벌어질 수 있다.

> [참고 사이트](https://javarevisited.blogspot.com/2013/03/reentrantlock-example-in-java-synchronized-difference-vs-lock.html) 의 도움을 많이 받았다.

## 그래서?

장황하게 썼는데 결국 silver bullet 이란 없기 때문에 두 가지 모두 고루 쓸 수 있어야겠다. 당장 봐도 익숙한 synchronized 가 아무래도 매력적으로 보이는데, 
외국에서는 4개 이상의 thread 가 접근 하는 경우 ReentrantLock 의 성능이 뛰아나다는 평이 있다. 또한 java 1.5 이후 ReentrantLock 이 점점 더 발전해가는 것을 보면
추후에는 synchronized 가 밀릴 수도 있을 것이다.

물론 그렇다고 아예 안쓰고 deprecated 될 일은 없겠지만 thread 성능상의 issue 가 생겼을 때 이 기능을 보고 생각보다 쉽게 풀어나갈 수 있지 않겠는가?

