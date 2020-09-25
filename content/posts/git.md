---
title: "Git"
date: 2020-09-25T11:22:38+09:00
draft: true
tags: ["VCS", "git"]
---

## git?

Version Control System 의 약어인 VCS 중 가장 강력한 tool 이다. 

최초 개발자는 그 유명한 [리누스 토발즈](https://ko.wikipedia.org/wiki/리누스_토르발스) 가 linux 커널 초기 개발자들과 함께 개발을 진행했다.

VCS 중 DVCS(Distribute Version Control System) 가 오히려 맞는 분류인데, 그 이유는 DVCS 에서 현재 git 에서 사용하는 여러 branch 와 merge 의 개념이 도입되었기 때문이다.

즉 master base 아래 여러 branch 로 분산 작업을 하고 그 중 merge 를 통해 버전을 관리하는게 git 의 주 목적이기 떄문에
일반적인 VCS 가 아닌 DVCS 로 분류된다.

git과 함께 대표적인 DVCS 는 머큐리얼, Bazaar 가 있다.

## 왜 알아야 하나?

아주 예전엔 SVN 이라는 걸 통해서 (그 전엔 CVS 가 있었음) 작업을 진행했었다. 당시 SVN 은 지금까지도 몇몇 기업에서 사용할 정도로 뛰어난 VCS 임은 부정 할 수 없지만
VCS 내에서도 단점이 존재하는데, root 에 commit 을 하는 순간 해당 작업은 되돌려질 수 없으며 commit 이 된 부분이 다른 개발자들에게 즉각적으로 반영이 된다는 점이 바로 그 단점이다.

반면 git 의 경우 모든 개발자가 master 에서 파생된 각자의 branch 를 가지고 있고 이 부분을 통해 각자 버전을 관리 할 수 있다.
또한 필요에 따라 rebase 등을 통해 branch 관리도 가능하기 때문에 분산된 환경에서의 작업일수록 효율적으로 작업이 가능하다.

## 구성

- branch
    
    영어로, 뜻은 '가지' 라고 한다. 가지라는 이름에서 알 수 있듯이 (그리고 git-flow 를 보면 알 수 있듯이) git 내 하나의 작업 단위라고 볼 수 있다.
    
    크게는 master 도 하나의 branch 로 볼 수 있고 작게는 본인의 branch 내에서 또 다른 하나의 기능 개발에 대해 새 version 관리를 했다면 이 또한 branch 라고 볼 수 있다.
    
    git 의 거의 모든 명령어는 이 branch 에 대한 관리로 이루어져 있다.

git 은 commit 과 push 가 별도로 분리되어 있다.

- commit
    
    로컬 branch 내에서 변경 사항 적용, 이렇게 적용된 부분은 원격 branch 에는 적용되지 않는다. 이를 통해 commit 후 오류 혹은 변경점 발생 시 다시 수정하여 commit 을 하면
    추후 push 시 최종 반영분만 1회의 push 로 인지된다.
    
    원격 혹은 마스터 branch 입장에서는 깔끔하게 정리가 되기 떄문에 관리에 용이하다.
    
- push
    
    로컬에서 원격 branch 로의 변경점 적용이다. 별도의 허가 없이 push 시 SVN commit 처럼 적용이 된다.
    
이처럼 변경점에 대한 적용 부분도 2가지로 나뉘어 있고 이러한 개념을 바탕으로 분산 버전 관리가 가능하다.

또한 mater - 원격 branch 간의 동기화 부분에서도 많은걸 지원해준다.

가장 간단한 merge 부터, rebase 까지 사용하기에 따라 다양하고 편리하게 사용이 가능하다.
다만 이러한 부분들 때문에 역설적이게도 learning curve 가 높은 편이다.

rebase 와 기타 branch 전략은 추후에 다루도록 하겠다.

## 그래서?

사실 대부분의 회사가 SVN -> git 으로 욺기는 추세이고 git 은 정말 아는 만큼 편하게, 다양하게 사용이 가능한 만큼 생각보다 많이 배워야 한다.
단순히 commit, push 한다고 다 되는게 아니고 branch 관리라던가 기타 이러한 부분들에 대해서도 잘 알아야 git 을 쓴다고 말할 수 있다.