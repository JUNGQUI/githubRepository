---
title: "ProcessNthread"
date: 2020-08-31T16:32:43+09:00
draft: false
---

# Process 와 thread

## Program? Process? Thread?

### Program
다음은 나무 위키에서의 program 에 대한 정의 중 일부 발췌문이다.
>컴퓨터에서 실행될 때 특정 작업(specific task)을 수행하는 일련의 명령어들의 모음(집합체)이다.

이와 같이 '어떠한 목적' 을 수행하기 위한 명령어들로 이루어진 모음이다.

### Process
Program 이 어떠한 목적을 이루기 위해 여러 task 를 잘게 쪼개놓은 것이다. 사전적 의미로는 이렇지만 실제로 보면 사실상 process 가 program 이지 않을까 한다.

개인적인 생각엔 최근에는 Program 은 사람이 인식하여 실행할 수 있는 단위 (ex : 카카오톡) 으로 인식되고
실제 CPU 등 pc resource 에 대해 소모하는 단위가 process 이지 않나 하는 생각이 든다.

자원에 대해 소모하는 단위라고 말했듯이 하나의 process 가 떠진 상태라면 OS 로부터 해당 작업에 대해 자원을 할당받았다고 할 수 있다.
또한 각기 process 가 독립적인 단위이므로 OS 로부터 독립적인 자원을 받는다.

흔히 말하는 Multi-process 에서는 이때 하나의 program 의 실행에 대해 process가 여러가지 만들어지는데
이 때 CPU core 는 A process 를 실행하던 도중 B process 를 실행 할때 context-switching 이 발생한다.

context-switching 은 사람으로 치자면 신문을 읽다가 TV 를 본다고 할 때 
신문을 읽던 주제에 대해 생각하던 도중 TV에서 방영하는 프로그램~~드라마등~~ 에 적응하는 과정이라고 볼 수 있는데,
이 때 사람의 두뇌는 글자를 읽다가 영상물을 읽게끔 인식의 전환이 일어난다~~고 한다~~. 이를 context-switching 이라 볼 수 있고,
이 때 CPU 는 물리적으로 다른 위치에 access 하여 code 를 읽고 다시 진행 상황에 대해 인지해야 하기 때문에 자원 소모가 발생한다.

당연히 이와 같은 context-switching 이 자주 일어나면 자원 낭비가 심해진다.

### Thread
multi-process 의 context-switching 의 대항마. 사람 입장에서는 process 나 thread 나 하는 일은 동잏하다. 다만 구조가 다른데
process 가 program의 단위라면 thread 는 process 의 단위이다.

추가로 다른 점은 자원에 대한 공유 이다.

process 가 program 을 실행 시 OS 로부터 각기 다른 자원을 할당 받는 반면, thread 의 경우 work 를 위해 process 내의 공유 자원을 이용해서 처리를 하기 때문에
context-switching 이 일어나지 않는다.

## 왜 알아야 하나?
Java 가 ~~(전 Java 진영이기 때문에)~~ 아무리 resource 에 대해 처리를 잘해준다고 하더라도 그게 곧 resource 구성에 대해 `몰라도 된다` 는 아니다.
오히려 그렇기 때문에 더 잘 알아야 하고 어떤 기술이 있는지, 왜 쓰는지에 대해 알아야 적재적소에 기술을 배치 할 수 있다.

개발계에 ~~필자가 미는~~ 명언이 있는데, `개발에 정답은 없어도 최선은 있다` 라는 것이다.

단순 for문만 사용해서, 혹은 iterator 를 사용해서 반복문에 대한 처리는 가능하다.

다만 이를 어떤걸 써야 resource 적인 소모가 적은지, 그렇게 해서 latency 나 performance 를 얼마나 끌어 올릴 수 있을 것인지 에 대해서 인지 해야 기술을 사용할 때 선택을 할 수 있고 이것이 개발자의 능력이라고 생각한다.

## 구성

### Process

process 는 `code`, `data`, `stack`, `heap` 의 영역으로 나뉘어져 있고, 다른 process 에 대해 접근할 경우 pipe, socket 통신 등의 방법을 이용해서
접근해야 해당 자원에 대해 접근 할 수 있다.

- code
>말 그대로 code. 이 부분은 개발자가 직접 작성한 code 외에도 기계어로 이루어진 code 들 또한 들어가 있다.

- data
>data 영역, 이 부분은 global, static 변수 등이 포함되어 있으며, 주의해야 할 점은 local variable 은 이곳에 저장되지 않는다는 점이다.
>
>그 이유는[...](https://www.google.com/search?q=%EC%82%AC%EB%9E%8C%EC%9D%84+%ED%99%94%EB%82%98%EA%B2%8C+%ED%95%98%EB%8A%94+%EB%B0%A9%EB%B2%95+%EB%91%90%EA%B0%80%EC%A7%80&source=lmns&bih=922&biw=1680&client=safari&hl=ko&sa=X&ved=2ahUKEwjMlYLXg8XrAhVZx4sBHcohBcgQ_AUoAHoECAEQAA)

- stack
>loacl 변수에 대한 저장소이다. 위 data 에서 local 에 대해 저장하지 않는다고 이야기한 이유가 여기에 있다. 
>
>global, static 변수와 다르게 관리하는 곳이 있는 이유는
>local 의 경우 해당 method 가 끝나고 나서는 다시는 쓰이지 않을 ~~다시 쓰여도 초기화되서 쓰일~~ 변수들이기 때문에 memory를 별도의 동적인 영역으로 만들어 유동적으로 사용하기 위함이다.

- heap
>size 가 정해지지 않은 list 와 같이 동적인 memory 가 저장되는 곳이다. 이 또한 stack 과 마찬가지로 동적으로 사용하기 위해 별도로 구성되었는데,
>stack 과도 따로 관리가 되는 이유는
>
>stack 은 휘발성으로 한번 사용하고 끝내는 정적 data 이지만 list 와 같은 경우 포인터의 개념으로 주소지에 접근하기 때문에
>memory 가 동적으로 관리되어야 했고 이를 위해 별도로 공간을 만들되, 더 이상 참조되지 않는 순간 주기적으로 GC 가 날리는 형식으로 발전하게 되었다.

### Thread

이미 앞서 말했듯이 thread 는 process 내의 자원을 공유한다. 고로 구성 자체는 비슷하다. 다만 thread 만 독자적으로 가지는 영역이 있는데 그게 바로 stack 이다.

간단히 생각해보자. code, data(global, static variable), heap(memory address) 의 경우 process 내에서 꾸준히 호출되는 (혹은 호출될 가능성이 있는)
뇨속들 뿐이다. 이 값들에 대해 각 thread 별로 별도로 가져간다면 context-switching 때문에 multi-thread 구성을 가져가야 할 이유가 1도 없다.

따라서 이 부분들에 대해서는 각 thread 별로 process의 자원을 공유하고, 이후 각 thread 가 실행하는 method 가 각기 다를 수 있기 때문에 local 변수를 의미하는 stack의 경우만이 thread 별로 별도로 생성된다.

## 그래서?

이와 같이 memory 구조가 이루어져 있기 때문에 개발 시 주의해야 할 점이 있다.

간단한 예로 List 는 편하게 동적으로 사용 할 수 있지만, 항상 동일한 size 가 생성되야 한다면 배열을 통해 memory 이점을 가져 갈 수 있다.

그런거 생각할 바에 빠르게 개발하는게 낫지 않냐 라는 시각이 있을 수 있다. 실제로 이러한 부분들은 엄청난 대용량이 아니면 경험하지 못하기 때문에 
항상 신경써야 할 정도로 중요하다고 볼 수 없다.~~본인도 그렇다~~

다만 습관을 이렇게 들이면 좀 더 compact 한 개발을 할 수 있지 않을까? 노트북은 가벼운걸 좋아하면서 정작 code 가 무겁다면 IDE 가 소모하는 배터리도 무지막지 
하고 결국 내가 들고 다니는 무게가 늘어나게 될 것이라 생각하면 당장 혹사 당하는 내 어깨를 위해서라도 내 두뇌를 훈련시켜야 하지 않을까.