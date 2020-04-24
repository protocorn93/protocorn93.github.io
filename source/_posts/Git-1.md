---
title: Git (1)
subtitle: Intro
date: 2019-01-27 16:57:09
tags: ["Git", "Github"]
category: ["Git"]
---

### Intro

------

**Version Control(VCS)**

- 파일의 변화를 시간에 따라 기록하여 특정 시점의 버전을 다시 불러올 수 있는 시스템
- ex) Git, Mercurial ... 

**Git**

- 개발과정, 소스파일 등을 버전별로 관리할 수 있는 도구
- 히스토리 관리를 통해 개발 과정, 역사를 시간의 흐름을 기준으로 볼 수 있으며 특정 시점으로 복구가 가능
- **분산 버전 관리 시스템**

**Github**

- Git이라는 도구를 응용할 때 필요한 원격 저장소 사이트
- 각족 원격 저장소들의 집합



### Workflow

------

![workflow1](http://ehdrjsdlzzzz.github.io/2019/01/27/Git-1/workflow1.png)



![](http://ehdrjsdlzzzz.github.io/2019/01/27/Git-1/workflow2.png)



![](http://ehdrjsdlzzzz.github.io/2019/01/27/Git-1/workflow3.png)

```
git init 
git add <file name / directory name>
git commit -m "<commit message>"
git remote add origin <remote repository URL>
git push -u origin master
```

- `git init`
  - 현재 디렉토리를 Working Directory로 지정. 일어나는 작업들을 추적하겠다는 의미로 초기화
- `git add <file name / directory name>` 
  - 추적되고 있지 않는 파일들을 스테이지에 올려 추적, 관리 시작
  - `git status`를 통해 현재 상태 확인 가능 (*Untracked*, *modified*)
- `git commit -m "<commit message>"` 
  - 변경 사항을 메시지와 로컬 저장소에 적용
  - `git log`를 통해 커밋 로그들을 확인할 수 있다.
- `git remote add origin <remote repository URL>`  
  - 주어진 URL의 원격 저장소를  `origin` 이라는 이름으로 등록 
- `git push -u origin master` 
  - `origin` 원격 저장소의 `master` 브랜치에 작업 사항을 동기화



### Collaborate

------

**Flow**

일반적으로 오픈소스에 기여를 하거나 협업을 진행할 때 볼 수 있는 전체적인 흐름을 나타낸 다이어그램이이다. (방법은 여러가지가 있다.)

![](http://ehdrjsdlzzzz.github.io/2019/01/27/Git-1/collaboration.png)



1. 먼저 기여할 혹은 협업할 저장소(이하 `Collabo Remote`)를 자신의 깃헙 계정 저장소로 **folk**를 한다.
2. 자신의 깃헙 계정에 folk된 저장소(이하 `User Remote`)를 자신의 PC로 **clone**을 진행한다. 
3. PC로 클론한 프로젝트(이하 `User Local`)에서 추가하고 싶은 혹은 자신이 담당할 기능에 대한 브랜치를 생성 (`feature`)하고 그곳에서 코드 작성을 진행한다. 
4. 새로운 기능을 담고 있는 `feature` 브랜치를 `User Remote` 저장소로 **push**한다. 
5. `User Remote`에 올라간 `feature` 브랜치를 `Collabo Remote`의 `master`(혹은 다른 브랜치) 병합을 요청한다. (**Pull Request, PR**)
   1. 코드 리뷰가 진행된다. 
   2. 코드의 수정이 필요하다면 계속해서 `feature` 브랜치에 수정 사항을 반영한다.
6. 병합(**Merge**)가 되면 이제 `Collabo Remote`의 `master` 브랜치는 새로운 기능이 추가된 최신 상태가 되었다. 
7. 이제 이러한 `master` 브랜치의 최신 상태(*`feature` 브랜치의 내용이 반영된*)를 `User Local`, `User Remote`에도 반영해주어야 한다. 먼저 `User Local`로 최신 변경 사항을 가져와 이전 상태와 병합을 진행한다. 
8. 그리고 최신의 상태를 유지하고 있는 `User Local`의 `master` 브랜치를 `User Remote`로 **push**하여 `User Remote` 역시 최신 상태로 갱신한다. 

> 이와 같은 내용을 담고 있는 코드스쿼드의 [Git 강좌](https://youtu.be/CbLNbCUsh5c)