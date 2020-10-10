---
title: "Gc2"
date: 2020-10-10T13:22:01+09:00
draft: true
tags: ["programming"]
---

## GC?

다시 돌아온 GC 설명이다. Garbage Collector 의 줄임말이다.

## 왜 알아야 하나?

사실 이전의 GC 는 일반적인 개념만 설명한 부분이 있었다. 그래서 serial gc 나 parallel gc 와 같은 경우를 
설명하지 않았는데 추가적으로 해당 내역에 대해서 공부 할 겸 정리한다.

## 구성

#### Serial GC?

문자 그대로 순차(Serial) 하게 GC 가 작동함을 의미한다. 일반적으로 별도 설정 없이 실행되는 GC 이다.

여기서 Mark-Sweep-Compaction 알고리즘이 나온다.

> - Mark-Sweep-Compaction?
>
>사용하지 않는 object 를 지정 (mark) 하고, 이후 해당 지정된 object 를 삭제 (sweep), 삭제 한 뒤
>eden 및 survivor 1/2 를 정렬 (compaction) 하는 알고리즘이다.

왜 순차로 진행하였냐면 이전에 하나의 GC 에 대해선 단일 thread 로 진행이 되었기 때문에 GC 자체가 순차적으로 진행이 될 수 밖에 없었다.

#### Parallel GC?

말 그대로 병렬 (parallel) GC 처리 방법이다. GC 알고리즘이나 기타 모든 사항은 serial 과 동일하지만
serial 은 GC thread 가 하나인 반면 parallel 은 GC thread 가 다수로 이루어져 있다.

serial 보다 Stop-The-World 시간이 단축 된다.

#### Parallel Old GC?

parallel GC 와 동일하지만 Major GC 부분도 parallel 방식으로 진행된다는 부분과 알고리즘이 Mark-Sweep-Compaction
대신 Mark-Summary-Compaction 으로 되어 있다는 것이 parallel GC 와 다른점이다.
그 외에는 모두 동일하게 구성되어 있다.

> - Mark-Summary-Compaction?
>
>나머지는 Mark-Sweep-Compaction 과 동일하지만 Sweep 대신 Summary 가 다른데, region 이라는 개념을 도입해서
>Region 을 Sweep 대상의 단위로 계산한다. 하나 하나의 object 에 대한 참조를 확인하는게 아니라
>object 참조에 대한 집합을 단위로 하고 전체적으로 가장 높은 값을 가지는 단위를 선택하는 점에 있어서
>STW 시간을 줄인다. 다만 설명에서도 보다시피 그저 단위만 변경하는 것이기 때문에 획기적인 변화가 이루어지진
>않는다.

#### CMS GC?

Concurrent-Mark-Sweep 의 약자로 알고리즘은 Initial Mark -> Concurrent Mark -> Remark -> Concurrent Sweep 순서로 사용된다.

- Initial Mark : GC root 에서 참조 tree 중 root 만 보고 GC 대상일지 아닐지 mark 한다. STW 가 일어나지만 root 만 보기 때문에 아주 짧게 발생한다.
- Concurrent Mark : 최초 mark 후 참조 tree 에서 타고 내려가면서 GC 대상인지 판단한다. STW 가 일어나지 않는다.
- Remark : Concurrent Mark 이후 GC 대상이 될만한 놈이 있는지 다시 확인한다. STW 가 일어나기 때문에 멀티 쓰레드로 작동한다.
- Concurrent Sweep : ~~조진다~~ GC 대상에 대해 Sweep 을 진행한다.

보면 최초 root 들만 보고 전체를 GC 로 볼지 안볼지를 아주 얕게 탐색 후 tree 를 타고 내려가는 부분에선 STW 가 일어나지 않으니 가체점 하듯이 체킹한다.
이러한 방식을 통해 최대한 STW 가 일어나지 않으며 전체적으로 GC 대상을 명확히 파악할 수 있는 부분에 있어서
Mark-Summary-Compaction 보다 정확하고 성능이 좋다.

#### G1 GC?

CMS 를 개선하는 방향으로 이루어졌으며 더 이상 물리적인 PC 의 자원이 무의미해져감에 따라 만들어진 GC 이다.
물리적 PC resource 를 생각안하고 만들었다고 소개했듯이 4g 정도의 메모리가 필요하다고 한다.

전체 Heap 을 region 으로 나눠 (이전 Mark-Summary-Compaction 에서의 region 과 동일) 계산을 한다.
(다만, 그렇다고 Eden, Old, Survivor 등을 없앤다는 소리는 아니다.)

그 이후에 아래와 같은 순서로 알고리즘이 진행된다.

- Initial Mark : CMS 와 동일

- Root Region Scan : Initial Mark 에서 체크된 구역 스캔, 이후 살아있는 객체가 없는 구역일 경우 제거한다. 
그렇기 때문에 일부 STW 가 발생한다.

- Remark : CMS 와 동일

- Cleanup : GC 대상이 많은 구역을 우선 제거한다. 이후 빈 구역을 FreeList 에 추가한다. 일부 STW 가 발생한다.

- Copy : GC 대상 구역에서 살아있는 객체를 새로운 구역에 복사해 모은다. STW 가 발생한다.

## 그래서?

Java 의 경우 VMOption 을 통해 손쉽게 GC 를 정할 수 있고, 이러한 정책을 통해 GC performance 를 가져갈 수 있다.

다만 확실히 현재 spec 과 상황에 맞게 GC 를 선택해야 하며 무조건적으로 G1 GC 가 좋은게 아니기 때문에 GC 에 대해서 잘 이해하고 진행해야 한다.

참고는 [여기](https://www.slipp.net/wiki/pages/viewpage.action?pageId=30770388) [저기](https://mirinae312.github.io/develop/2018/06/04/jvm_gc.html) 를 참고했다.