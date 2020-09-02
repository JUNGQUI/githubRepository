---
title: "Jvm"
date: 2020-08-27T10:59:24+09:00
draft: false
---

# JVM 과 GC

## JVM?
Java Virtual Machine 의 약자로 말 그대로 java 를 돌리기 위해 가상으로 Computer 의 자원을 가진
존재이다.

쉽게 보자면 Linux 이던 window 이던 상관 없이 java 를 사용하는 것도 program - OS 간 가교 역할을 해주는 JVM 의 공로라 할 수 있다.

## 왜 알아야 하나?
후술 할 GC 를 이해하기 위함과 동시에 컴퓨팅 자원에 대해 직접적으로 관여하지 않는 (비교적) java 라 하더라도 대용량 데이터 및 대규모 I/O 작업이 일어날 경우, 
강력한 performance 가 보장되지 않기 때문이다.

따라서 개발자들은 (적어도 많은 data 를 다룬다면) 반드시 `이해` 해야 한다. (주관적 의견입니다)

당연히 GC 또한 이해 없이 사용 시 원래 예상했던 performance 가 나오지 않고 이는 생산성 저하 및 품질 저하로 이어지기 때문에 ~~유지보수 야캐요~~ memory 에 대한 이해는 매우 중요하다.

## 구성

- Class Loader ~~CL~~
- Execution Engine ~~EE~~
- Runtime data area

### Class Loader

javac 가 .java 들을 .class 로 만들어 줄 경우 해당 class 에 대해 올리는 작업(loading) 을 진행한다.
이를 통해 최근에 개발 or 업데이트 된 내역이 반영되는 것이다.

### Execution Engine

Class loader 를 통해 적재된 class 들을 실질적으로 실행하는 engine 이다. 이 engine 에는 두 가지 구동 방식이 있는데,

- Interpreter
 
전통적인 code 실행 방식. 1-line base 로 실행 되기 때문에 코드가 방대해 질수록 실행 속도가 느려진다. lambda 식이 사랑을 받는 이유가 여기에 있다.

- JIT compiler
 
Just-In-Time 의 약자로 어느 기점 전까지 interpreter 방식으로 진행하다가 어느 순간 이후 전체를 한번에 실행하고, 이 실행 결과를 cache 처럼 보관하여
추후 사용 시 다시 interpreter 를 이용하지 않고 cache를 실행한다.

여기서 어느 순간 과 같은 부분은 해당 code 가 재사용성이 매우 높다고 (빈번하게 호출된다고) 판단하는 순간이다.


두 가지 구동 방식이라 칭했지만 실제로는 두 가지 방법 모두 병행하여 사용한다.

- GC

우리가 흔히 알듯이 GC는 더 이상 참조되지 않는 memory를 찾아내어 free 시켜주는 역할을 하는데, 이를 통해 개발자는 일일이 memory 할당 후 free 할 필요 없이 
혹은 걱정 없이) 사용이 가능하다.

다만 모든 tool 에 만능이란 없듯 ~~silver bullet~~ GC 또한 설정과 실제 code 의 style 에 따라 효율에 차이가 발생한다.

### Runtime Data Area

이름만 봐도 알 수 있듯이 실제 구동 시 Data 에 대한 영역이다. Multi-process 환경이라 할 때 이 부분을 process 의 개수만큼 분할하여 사용한다.

### Thread 별 생성

- PC register

Thread 가 생성 될 때 마다 생성되는 공간, JVM - thread 간의 명령에 대한 주소 등 meta data 가 있는 장소이다.

이 부분을 통해 thread 가 자신의 해야 할 일, 자신의 결과에 대한 보고 등을 알 수 있다. 

- JVM stack (stack)

method 내에서 사용된 thread 나 method 에 대한 정보, return value, 연산 결과 등이 저장된다.

또한 method 에 대한 정보를 다루기 때문에 method 내의 지역 변수, 임시 변수, 매개 변수, method 를 호출한 곳의 address 등에 대해서도 이곳에 저장된다.

여기서 중요한 부분이 바로 stack 에 저장되는 변수가 `값으로써의 변수` 가 아닌 `해당 값을 참조하는 주소값` 이라는 점이다. 이 부분에 대해서는 process & thread 부분에서 다룰 예정이다.

- Native method stack

말 그대로 native method 영역으로 java 로 만들어진 연산에 대해 C 등 을 이용해 kernel 에 실행 시 필요한 memory 공간이다.

### 공용 생성

- Method area(static 영역)

class 에 대한 정보, static 변수 및 method, 
뿐만 아니라 변수들의 참조 값과 class 가 interface 인지 아닌지 등도 같이 관리하게 된다.
class 혹은 그와 관련된 정보들을 다룬다. static 변수, Enum 상수 등 memory 에 올라가서 사용 가능한 부분이 바로 이 부분에 해당한다.

사실상 code 의 거의 모든 부분이 올라가기 때문에 흔히 memory 단에서 뭔가를 한다 라고 할 때 이 부분에 대해 말한다.
~~개인적인 생각입니다.~~

- Heap

heap 은 또 eden, survivor 0/1, old, permanent 등으로 구성되어 있다. 또한 대부분의 변수가 저장되면 바로 이곳에 저장된다.

이 부분에 대해서는 GC 가 연관되어 있기 때문에 GC 관련된 글 작성 시 자세히 설명 하려 한다. 우선 간단하게 설명하자면,

~~permanent : 전체 code 에 대한 method 나 class 의 meta data 가 저장되는 공간.~~ 
~~그렇기 때문에 Reflection 이 자주 일어나는 spring 의 경우 이 부분에 대한 관리가 중요하다.~~

Java 8 이후 부터 permanent 의 경우 Native memory 의 meta space 로 대체되었다.

대체된 이유로는 heap 이 아니라 사실상 전체 code 에 대한 meta data 가 주로 이루어져 있었기 때문에 Heap 이 아닌 OS 에 위임하는게 맞다고 생각해서 라고 한다.

또한 이와 같은 방식으로 memory 관리를 할 경우 heap 에 강제되는 memory 가 없어지기 때문에 heap 은 이전보다 더 많은 자원을 할당받아 사용 할 수 있게 되었다.

eden : 변수 생성 시 최초 저장되는 공간

survivor 0/1 : eden 에서 오래된 변수가 저장 되는 공간

old : survivor 에서 오래된 변수가 저장 되는 공간
