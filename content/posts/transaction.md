---
title: "Transaction"
date: 2020-09-10T11:18:19+09:00
tags: ["DB", "spring"]
---

## Transaction?

위키피디아의 정의에서 transaction은 `데이터베이스 트랜잭션(Database Transaction)은 데이터베이스 관리 시스템 또는 유사한 시스템에서 상호작용의 단위이다.` 라고 정의되어 있다.

이와 같이 시스템 내에서의 작업의 단위로 볼 수 있는데, DB에서는 DB I/O 간의 1회 connection 시의 작업 단위이며, 이 작업 단위 상에서는 ACID가 지켜져야 한다.

>ACID ~~산성~~ (원자성(Atomicity), 일관성(Consistency), 독립성(Isolation), 영구성(Durability) 의 첫 글자를 따온 약어)
>
>- Atomicity
>
>transaction 도중 중간에 작업의 단위를 끊고 일부만 진행하는 것은 존재할 수 없다.
>
>ex) `계좌 A 와 계좌 B 간의 이체` 를 하나의 작업으로 간주 시, A에서 출금 후 작업 종료 (금액 증발) 과 같은 경우는 있어선 안된다.
>
>- Consistency
>
>transaction 은 시스템 내에서 정의된 제약 조건에 대해서는 어떠한 작업이 오더라도 지켜져야 한다.
>
>ex) `현금 카드 사용 시 금액 차감` 이 하나의 작업이며 `현금 카드 잔액은 항상 0 이상` 이 제약 조건일 경우 현금 카드는 실제 결제를 하지 않고 실패 처리
>
>- Isolation
>
>모든 작업 도중에는 transaction 은 해당 transaction 만 존재 할 수 있다. 하지만 상황에 따라 transaction 이 확장 될 수 있다.
>
>ex) `이체` 와 `입금` 작업이 동시에 진행 될 경우 먼저 요청받은 순서대로 해당 작업을 진행하고 완료 이전 까지 다음 작업은 lock 상태에서 대기한다. 
>
>- Durability
>
>모든 transaction 은 성공적으로 작업 될 경우 DB에 영구적으로 반영되어야 한다. 이를 통해 transaction 을 작업의 단위로 두고 DB lock 등을 통해 data 의 일관성을 유지 할 수 있다.
>
>ex) query 작업 완료 이후 commit 을 진행하기 전까지 (즉, transaction 이 완전히 끝나기 전까지) application 내에서는 작업이 반영되어 있지만 DB 에는 여전히 작업이 되어 있지 않다.
>이후 commit 을 통해 반영을 한 뒤 DB에 비로소 작업한 내역이 반영된다.

## 왜 알아야 하나?

대다수의 application 에서는 DB 작업이 빠질 수 없는데, 이 DB 작업 시 transaction 은 선택이 아닌 필수다. 사실 본인도 이러한 개념만 알고 사용하고 있었는데 이 외에
위의 ACID 등 원칙을 지키면서 추가 option 이 여러가지 있는데, 상황에 따라 이러한 부분을 선택해서 사용하면 굉장히 유용히 쓸 수 있을 것이다.

예를 들어 후술할 transaction  옵션 중 propagation 이라는 것이 있는데 여기에서 REQUIRED_NEW 와 같이 사용하면 작업 진행 중 별도의 작업으로 새로 분리 할 수 있다.
전체 flow(이하 A) 에서 필수는 아니지만 개발자 편의상 DB 작업을 해야 하는 table 이 있다면(이하 B) 이와 같은 option 으로 작업 중 부드럽게 transaction  분리 및 별도 작업이 가능하다. 
이를 통해 A 작업과 B 작업을 분리하여 만약 B 작업이 실패하여도 A 작업에 영향을 끼치지 않게 개발 할 수 있다.

## 구성

transaction 은 위에서 언급했듯이 하나의 논리적 작업의 단위이다. 보통 개발에서는 이 부분에 대해 DB I/O 작업 단위로 구성되어 있다.

transaction 에는 5가지 상태가 존재하는데, 활동(Active), 실패(Failed), 철회(Aborted), 부분 완료(Partially Committed), 완료(Committed) 단계로 나뉘어져 있다.

각 단계는 `활동 - task == success ? : 부분 완료 - 완료 : 실패 - 철회` 로 나뉘어져 상태가 변경되고 `완료 || 실패` 에 도달할 경우 transaction 이 종료된다.

- 활동(Active)

말 그대로 활성화 된 다음부터 작업이 진행되는 도중의 상태이다. 이 상태일때는 query 로 볼 시

``` bigquery
BEGIN;

SELECT * FROM SOME_TABE;

COMMIT;

```

`SELECT` 부분이 활성 상태라고 볼 수 있다.

## 그래서?