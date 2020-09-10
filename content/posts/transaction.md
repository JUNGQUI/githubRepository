---
title: "Transaction"
date: 2020-09-10T11:18:19+09:00
draft: true
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

각 단계는 `ACTIVE.result() == success ? : PARTIALLY_COMMITTED : FAILED` 로 나뉘어져 상태가 변경되고 이후 `부분 완료 -> 완료` / `실패 -> 철회` 로 이어지며, `완료 || 철회` 에 도달할 경우 transaction 이 종료된다.

- 활동(Active)

말 그대로 활성화 된 다음부터 작업이 진행되는 도중의 상태이다. 이 상태일때는 query 로 볼 시

``` bigquery
1: BEGIN;
2:
3: SELECT * FROM SOME_TABE;
4:
5: COMMIT;
```

`SELECT` 부분이 `Active` 상태라고 볼 수 있다. 
4번 line 까지 진행되었을 경우 해당 line 이 `Partailly Committed` 부분인데, 
만약 `SELECT` 도중 timeout 이 발생해서 결과를 가져오지 못했다면 
이 때의 상태가 `Failed` 상태가 된다.

``` bigquery
1: BEGIN;
2: UPDATE SET SOME_COLUMN = 'SOME_VALUE' where SOME_ID = '1'; -- 정상 처리 --
3: SELECT * FROM SOME_TABE; -- DB read timeout 발생 --
4:
5: COMMIT;
```

만약 위 query 에서 `UPDATE` 이후 `SELECT` 에서 DB timeout 이 발생했다면 
실제로 `COMMIT` 이 진행되지 않았기 때문에 transaction 상태는 `Failed` 상태로 변경되고
이후 DB 의 설정에 따라 진행되게 된다. 보통 기본 값은 transaction 작업 내 모든 작업 Roll-back 이 기본 정책이므로
`UPDATE` 내역은 roll-back 되어 반영되지 않는다.
이 상태가 `Aborted` 상태이다.

상태에 대해 알아보았으니 spring 내의 spring-tx 에 대해 알아보자~~ARABOJA~~

spring-tx서(이하 transaction 으로 통일) 는 spring 에서 제공하는 transaction 관련 package 인데 단순히 annotation 만 적어주면 application 에서 해당 작업 수행 시 transation begin
으로 시작하여 작업 종료 후 commit 및 roll-back 을 관리해준다. 이를 이용하면 개발자가 별도로 신경 쓸 필요 없이 
transaction 에 대해 자유롭게 사용 할 수 있다.

transaction 에는 propagation ~~propaganda(아닙니다)~~ 과 isolation 이라는 두 가지 option 이 있는데, 이 기능을 잘 활용하면 ACID 를 지키면서
다채롭게(?) transaction 을 이용 할 수 있다.

- Propagation

위키피디아의 단어 뜻을 찾아보면 번식(...) 이라고 되어 있는데 다른 뜻으로는 `전파` 라는 뜻도 있다. 이 전파라는 개념이 
transaction 내의 propagation 과 일치한다. 

위에서 언급했듯이 transaction 은 작업의 단위인데 이 `단위` 에 대한 범위 설정을 propagation 을 통해 설정해 줄 수 있다.
Enum class 로 정의되어 있으며 종류로는
REQUIRED, REQUIRES_NEW, SUPPORTS, NOT_SUPPORTED, NEVER, NESTED, MANDATORY 
가 있다.

- REQUIRED : transaction 이 존재하지 않다면 transaction 으로 만들고, 있다면 기존 transaction 단위에 병합한다.
- REQUIRES_NEW : REQUIRED 와 동일하게 작업하는데, 다른 점은 기존 transaction 이 있더라도 단위를 병합하지 않고 새 transaction 을 생성한다.
- SUPPORTS : 기존 transaction 단위에 통합하지만, transaction 이 없을 경우 transaction 으로 작업하지 않는다.
- NOT_SUPPORTED : 기존 transaction 이 있더라도 transaction 으로 작업하지 않는다.
- NEVER : transaction 으로 작업하지 않는다. transaction 이 존재한다면 exception 을 발생시킨다.
- NESTED : 기존 transaction 이 있을 경우 REQUIRES_NEW 처럼 작동하고, 없을 경우 REQUIRED 처럼 작동한다.
- MANDATORY : 기존 transaction 이 있을 경우 기존 transaction 과 병합되고 없을 경우 exception 을 발생 시킨다.

다른 option 은 이해가 되는데 REQUIRES_NEW 와 NESTED 는 헷갈리는 개념인데 ~~저는 그랬습니다..~~
[여기](https://stackoverflow.com/questions/12390888/differences-between-requires-new-and-nested-propagation-in-spring-transactions)
를 보면 REQUIRES_NEW 의 경우 무조건 새로운 transaction 으로 작업하는 반면, NESTED 의 경우 기존 transaction 이 있다면 시작하는 부분을 savepoint 로 지정하고
이후 새로운 transaction 처럼 작동한다. 이렇게 작업할 경우 exception 발생 시 지정한 savepoint A 로 이동한다.

다만 transaction 이 없을 경우 transaction 이 없기 때문에 REQUIRED 처럼 새로운 transaction 으로 작업하게 된다.

기본적으로 transaction annotation 사용 시 별도 설정이 없다면 REQUIRED 가 기본 값으로 정해져 있다.

- Isolation

뜻으로는 고립, 독립으로 볼 수 있는데 고립이라는 단어가 더 알맞다고 볼 수 있다.

그 이유로는 Isolation option 은 `transaction 작업 시 해당 transaction 에 대한 외부 접근에 대해 차단하는 수준`을 정하는 option 이기 때문이다.

Propagation 과 마찬가지로 Isolation 도 enum class 로 관리되고 있으며 종류는 
DEFAULT, READ_COMMITTED, READ_UNCOMMITTED, REPEATABLE_READ, SERIALIZABLE 이 있다.

우선 isolation level 에 대해 알아보기 전에 DB transaction 오류에 대해 알아보자.

이후 모든 예시는 해당 table 로 진행한다고 가정한다.

또한, **편의상 한번의 query 를 수행했지만 (A), (B) 를 통해 서로 다른 thread 가 접근하여 수행** 했다고 가정한다.

>| SOME_ID | SOME_COLUMN |
>| --- | :--- |
>| 1 | value1 |
>| 2 | value2 |

---
- Dirty Read : un-commit 된 data 에 접근하여 읽어들이는 경우. data 일관성이 깨져 logic 수행 시 엉뚱한 결과로 저장 될 수 있다.
select 시 lock 의 부재로 인한 문제이다.
``` bigquery
BEGIN;
UPDATE SOME_TABLE SET SOME_COLUMN = 'SOME_VALUE' WHERE SOME_ID = '1'; --(A)--
-- value1 -> SOME_VALUE 로 변경 --
```

이러한 작업이 수행되었을 때,

``` bigquery
BEGIN;
SELECT * from SOME_TABLE; --(B)--
-- id 1, 2 와 SOME_VALUE, value2 가 출력, application 이 작업 수행 --
```
이 작업이 수행되면 분명 commit 이 수행되지 않았지만 이미 (A) 에서 update 를 수행한 부분이 반영되어 select 된다.
이와 같이 시점이 다를 때 마다 값이 다르면 application 에서 logic 수행 시 항상 결과가 다르게 출력 될 것이고 이는 error 를 유발한다.

이를 Dirty read 라 한다.

---

- Non Repeatable Read : 하나의 작업 내 반복된 read 시 다른 transaction 작업으로 인해 동일한 column 결과가 나오지 않는 경우. 마찬가지로 일관성이 깨진다.
``` bigquery
BEGIN;
SELECT * from SOME_TABLE; --(A)--
-- id 1, 2 와 value1, value2 가 출력, application 이 작업 수행 --
SELECT * from SOME_TABLE; --(A)--
-- id 1, 2 와 SOME_VALUE, value2 가 출력, application 이 작업 수행, 일관성 깨짐 --
COMMIT;
```
위의 query 와 동시에 수행되고, 두번째 select 가 수행되기 전 아래의 query 가 실행되었다고 가정해보자.
``` bigquery
BEGIN;
UPDATE SOME_TABLE SET SOME_COLUMN = 'SOME_VALUE' WHERE SOME_ID = '1'; --(B)--
-- value1 -> SOME_VALUE 로 변경 --
COMMIT;
```
이 부분이 수행된다면 위와 같이 두번째 select 에서 결과가 다르게 도출된다.

언뜻 보면 문제될 것이 없어 보인다. 내가 원하는 바는 어쨌든 SOME_TABLE 의 결과 조회이고, commit 이 된 다음 값을 정상적으로 가져왔다.

하지만 문제는 select 를 하는 첫번째 query 가 아직 transaction 단위가 끝나지 않았고 이는 data 의 일관성이 보장되어야 함을 뜻하는데
다른 query 의 수행 결과로 인해 전체 process 내에서 data 의 일관성이 깨져 반복해서 읽었을 때(repeat) 문제가 발생했다. 이러한 부분이 바로 Non-Repeatable Read 문제이다. 

---

- Phantom Read : 하나의 작업 내 반복된 read 시 다른 transaction 작업으로 인해 동일한 row 가 나오지 않는 경우. 마찬가지로 일관성이 깨진다.
``` bigquery
BEGIN;
SELECT * from SOME_TABLE; --(A)--
-- id 1, 2 와 value1, value2 가 출력, application 이 작업 수행 --
UPDATE SOME_TABLE SET SOME_COLUMN = 'SOME_VALUE' WHERE SOME_ID = '1'; --(A)--
-- value1 -> SOME_VALUE 로 변경 --
SELECT * from SOME_TABLE; --(A)--
-- id 1, 2, 3 과 SOME_VALUE2, value2, value3 이 출력, application 이 작업 수행, 일관성 깨짐 --
COMMIT;
```
위와 마찬가지로 동시 수행 후 두번째 select 가 수행되기 전 아래의 query 가 수행되었다고 가정한다면,
``` bigquery
BEGIN;
UPDATE SOME_TABLE SET SOME_COLUMN = 'SOME_VALUE2' WHERE SOME_ID = '1'; --(B)--
-- value1 -> SOME_VALUE 로 변경 --
INSERT INTO SOME_TABLE VALUES (3, 'value3') --(B)--
-- SOME_ID 가 3, SOME_COLUMN 이 value3 인 row 추가 --
COMMIT;
```
이와 같이 data 일관성이 깨지게 된다. 분명 A 입장에선 update 후 작업을 진행하려 했으나 실제 값은 B 가 commit 한
SOME_VALUE2 와 있지도 않던 row SOME_ID 3인 값이 튀어나왔다. 분명 commit 된 값을 읽었으니 Dirty Read 는 아니지만, 동시 update 및 insert 에 대해
제대로 lock 이 지켜지지 않았기에 발생한 일관성이 깨지는 문제이다.

이를 본인은 인지 못한 유령(phantom) 값이 생겼기에 phantom read 라 칭한다.

- DEFAULT : 사용하는 DB 에 따라 기본 isolation 을 따라간다. postgreSQL 의 경우 기본 값이 READ_COMMITTED 로 되어 있다.

- READ_UNCOMMITTED (level 0) : commit 이 되지 않아도 접근이 가능하다. 이럴 경우 Dirty Read 가 발생 할 수 있다.

- READ_COMMITTED (level 1) : commit 이 된 작업에 대해서만 접근이 가능하다. 만약 commit 되지 않은 부분에 대해 다른 transaction 이 접근 할 경우 lock 을 통해 제어한다.

  하지만 commit 된 data 를 읽기 때문에 read 중 commit 이 된다면 commit 된 데이터를 읽어, 반복 작업 중 data 일관성이 깨지는 Non-Repeatable Read 가 발생한다.

- REPEATABLE_READ (level 2) : Non-Repeatable Read 를 막는 isolation level. 동일한 table 내 작업 시 다른 transaction 에서 update 가 진행되어도
해당 transaction 내에서는 transaction 시작 시 data 에 대해 접근해서 가져오기 때문에 동일한 data 를 보여준다.

  하지만 data `read 일관성` 은 유지하더라도 다른 곳에서 update 및 insert, delete 가 commit 된다면 해당 transaction 중 update, insert, delete 한 data 는
  반영되지 않는 phantom read 가 발생 할 수 있다.

- SERIALIZABLE (level 3) : 가장 고수준의 isolation level. 위의 모든 작업에 대해 방어가 가능하다. ~~우주 방어~~

  다른 isolation level 에 비해 이 level 을 적용한다면 실제 동일 table 에 대해 transaction 발생 후 다른 transaction 이 update 를 시도할 경우 
  exception 이 발생하여 아예 다음 단계로 넘어가지 못하게 된다.
  
  다만, 당연하게도 이와 같은 방법을 쓰면 동시성에 대해 굉장히 떨어지게 되므로 performance 측면에서 불리할 수 있다.
  
isolation level 의 경우 점점 고레벨이 적용될수록 저레벨까지 커버하는 특징이 있다. 즉 SERIALIZABLE 을 적용하면 READ_COMMITTED, REPEATABLE_READ 가
모두 적용되기 때문에 하나만 사용 시 다른 것들에 대해 커버가 안되는 걱정은 하지 않아도 된다.

## 그래서?

~~그래서는 무슨 그래서야~~ 잘 쓰자. 있는 option 을 combine 해서 각 상황에 맞게 사용하면 performance 증대도 물론 있겠지만 ~~없으면 굳이 안만들어 놨겠지?~~
가장 중요한 점은 co-op 할 경우 '해당 method 는 이렇게 동작하는거야' 하고 interface 에 명시해주면 다른 개발자가 해당 interface 사용 시
'아 이건 이렇게 되어 있으니 내가 지금 사용 할때는 이것의 spec 에 맞게 사용하면 되겠다' 라고 별도 의사 소통 없이 사용가능하며
혹시 필요한 needs 가 생기면 추가 개선 요청을 통해 유동적으로 변경하여 개발을 진행하면 되겠다.

늘 말하지만 개발에 절대는 없고 항상 business logic 이 변하는 만큼 유동적으로 작업할 줄 알아야 하며 그러기 위해 해당 option 에 대해
잘 알고 사용해야 하기에 끊임없이 공부해야 할 것이다.