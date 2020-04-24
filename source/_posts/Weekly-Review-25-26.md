---
title: Weekly Review(25, 26)
subtitle: 20190617-20190623, 20190624-20190630
date: 2019-07-02 20:20:21
tags:
---

다시 2주치 회고록이다. 

> **할말하않** (할말이 많지만 하지는 않겠다.😅)

일단 진행하던 새 사이드 프로젝트는 멈춘 상태다. RxSwift + MVVM과 더불어 원하는 기능까지 구현하려니 조금은 버거운 감이 없지 않아있었다.

그래서 기존 프로젝트인 [GitBingo](https://github.com/ehdrjsdlzzzz/GitBingo)를 RxSwift와 MVVM을 사용해 리팩토링을 진행해보는 방향으로 바꿨다. 



### 두 주 동안 나는 무엇을 공부했을까? 📝

---

- [RxDataSources](https://github.com/RxSwiftCommunity/RxDataSources)

  - 기존의 GitBingo는 콜렉션 뷰로 데이터를 보여주는데, 섹션을 나누어 보여준다. 기존의 RxCocoa나 RxSwift로 원하는 화면을 구성하는 데는 무리가 있어 보였고, 여러 글들에서도 이런 상황에선(섹션을 사용) RxDataSources를 사용해야 한다고 안내되어 있었다.
  - RxDataSource의 문서는 꽤나 직관적이었다. 예제는 테이블 뷰로 되어있었으나, 콜렉션 뷰와 함께 사용하는데도 큰 무리가 없었다. RxDataSources를 통해 어떻게 기존의 GitBingo의 홈 화면을 구성했는지는 아래에서 살펴보도록 하자. 

- Postman

  - 포스트맨을 사용은 해봤지만 이를 활용했다는 느낌은 없었다. 하지만 사내 개발자 테크톡에서 알게 된 내용을 회사 프로젝트를 진행하는데 접목시켜 사용해봤다. 포스트맨을 잘 활용한다면 보다 효율적으로 쉽게 API를 테스트해볼 수 있다는 것을 알게 되었다. 
  - [정리한 글](https://ehdrjsdlzzzz.github.io/2019/06/21/Postman-활용기/)

- [Advanced iOS App Architecture](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=2ahUKEwiKpcG4k5bjAhWSv5QKHc2HB1QQFjAAegQIARAB&url=https%3A%2F%2Fstore.raywenderlich.com%2Fproducts%2Fadvanced-ios-app-architecture&usg=AOvVaw0AzuTlvK9j4d4iOImrw0gu)

  - @메이슨, @준상님, @지용님이랑 함께 스터디를 시작했다. 스터디에서 다루는 주제는 아키텍처로 Advanced iOS App Architecture 책을 사용한다. 1주차는 DI(Dependency Injection) 부분을 공부하고 궁금했던 것이나 이야기할 거리들을 갖고 서로의 생각과 의견을 나누어보았다. 

  - 역시 내가 가장 부족했다.😅 RIBs에 대해서도 들어보긴 했지만 이를 살펴보지는 못했는데, 나를 제외하곤 모든 분들이 가볍게라도 영상과 자료를 읽어보신 경험이 있으셨다. 

    > 그래서 나도 귀가하자마자 Let's Swift의 RIBs를 주제로 한 영상을 시청했다. 

    그리고 이렇게 실력이 있으신 분들과 이야기를 나누고 코드를 작성하면서 고민했던 것들, 궁금했던 점들을 이야기하고 의견을 나누는 과정이 너무나 즐거웠다. 즐거웠지만 내가 많이 부족하다는 사실을 깨달았고, 겉핥기식으로만 공부한다는 사실을 깨달았다. 공부도 공부였지만 자기반성을 할 수 있었던 시간이었고 앞으로 스터디가 너무나 기대된다.

### 내 프로젝트 어디까지 진행되었나? 💻📱

---

위에서 언급했듯이 나는 RxSwift로 먼저 기존의 로직을 변경하는 것을 목표로 했다. 여기서 위에서 언급한 이유로 RxDataSources를 사용했는데 이를 사용해야 하는 것은 알겠는데 어떻게 사용해야 하는지가 막막했다. 기본적으로 구글링을 통해 나온 예제들은 모두 정적인 데이터를 사용하고 있었다. 

역시 여기서 나는 @메이슨의 도움을 받았다. (감사합니다!🙏)

기존에는 네트워크 호출의 결과로 반환된 Raw한 데이터를 `Parser`를 통해 화면에 보여주기 위한 모델로 가공해주었다. 여기까지는 동일했다. 이제 이렇게 가공된 모델을 섹션 별로 나누어 `PublishSubject<[섹션헤더모델타입, 셀모델타입]>`의 형태로 흘려보내주고 뷰 컨트롤러는 이를 바인드해 콜렉션 뷰에 보여주도록 구현하였다. 

지금 설명은 굉장히 간단하지만 Rx를 제대로 이해하지 못한 상태에선 이를 구현하기 위해 정말 많은 고민을 했었다. 그래도 원하는 결과가 제대로 나와 이후 진도도 잘 나가고 있다. 현재는 에러 처리를 위해 고민하고 있다.

### 두 주 동안 나는 무엇을 읽었을까? 📖

---

위에서 언급했듯이 [Advanced iOS App Architecture](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=2ahUKEwiKpcG4k5bjAhWSv5QKHc2HB1QQFjAAegQIARAB&url=https%3A%2F%2Fstore.raywenderlich.com%2Fproducts%2Fadvanced-ios-app-architecture&usg=AOvVaw0AzuTlvK9j4d4iOImrw0gu) 이 책의 Dependency Injection장을 중심적으로 읽었다. 이 책에서 의존성을 주입하는 방법으로 네 가지를 소개해주었다. 

- On-Demand
- Factory 클래스 
- Container 클래스
- Container Hierarchy

위에서부터 아래로 내려가며 단점을 보완한 방법이지만 그에 반비례하여 복잡성은 비교적으로 증가된다. DI 툴로 [Swinject](https://github.com/Swinject/Swinject)를 들어본 적이 있는데 레포지토리에 소개된 기능에 다음과 같은 항목이 소개되어있다.

- [Object Scopes as None (Transient), Graph, Container (Singleton) and Hierarchy](https://github.com/Swinject/Swinject/blob/master/Documentation/ObjectScopes.md)

기능으로 소개된 것 중 Graph, Container 그리고 Hierachy가 이 책에서 언급되는 Container와 Hierachy 개념을 말하는 것으로 생각된다. 다시 한번 책을 정독하고 Swinject를 사용한 예제도 간단하게 만들어볼 생각이다. 



### 정리하는 글 🧐

---

회사일에 굉장히 스트레스를 받는 한 주였다. 공부를 하고 책에서 읽은 내용들을 접목시켜보고자 했지만 뜻대로 되지 않아 자괴감도 많이 들고 코드를 엉망으로 작성하고 있는 나를 발견하고 스스로에게 굉장히 실망을 했다. 

클린 코드에서 "깨어진 창문" 이야기가 나온다. 온전한 창문과 반대로 깨어진 창문은 누구나 쉽게 더 창문을 깨뜨린다는 의미다. 누군가 코드에 금을 가게 하면 그 뒤부터 코드를 망치는 것은 굉장히 쉽다. 내가 창문을 깨는 행위에 동참하는 것 같아 스스로에게 많이 많이 힘든 한 주였다. 

하지만 다시 마음을 다잡고 새로운 한 주를 시작해야겠다. 어떻게 하면 이 깨진 창문을 더 깨지지 않게 유지할 수 있는 방법에 대해 고민해보고 또 고민해봐야겠다.