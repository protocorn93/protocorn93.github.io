---
title: Weekly Review(2)
subtitle: 20190107-20190113
date: 2019-01-13 21:02:03
category: ["Weekly Review"]
---

이번 주는 사실 만족스럽지 못했다. 개인적으로 여러 일들이 있었으며 공부에 만족스럽게 집중하지 못했다. 이번 주 계획이었던 Rx 공부와 클린 코드 책 읽기는 제대로 이루지 못하였다.  

하지만 이전부터 만들어보고 싶었던 기능 하나를 만들어보아서 그 부분에 대해선 만족스러웠다. 이번 주를 정리해보자!

---

### 한 주 동안 나는 무엇을 공부했을까?

하나씩 살펴보자

- [Functional Reactive Programming 패러다임](https://www.youtube.com/watch?v=cXi_CmZuBgg&feature=youtu.be)
  - Rx 공부를 하기 위해 당장 코드부터 살펴볼 것인지, 원리부터 제대로 파악해야 할지 고민을 많이 했다. 어찌됐든 간에 둘 모두를 살펴보아야 하는 것은 당연하다. 하지만 처음 공부를 시작하면서 어느 것에 더욱 비중을 두어야 할지를 고민하였다. 
  - 그래서 클린트님의 [awesome-swift-korean-lecture](https://github.com/ClintJang/awesome-swift-korean-lecture) 저장소를 참고하였다. (개인적으로 굉장히 추천드리는 저장소이다!!👍) 렛어스고에서 곰튀김님의 [Functional Reactive Programming 패러다임](https://www.youtube.com/watch?v=cXi_CmZuBgg&feature=youtu.be) 세션을 추천해주셨고 함수형 프로그래밍에서 주로 언급되는 용어들에 대한 이해와 함께 전반적인 함수형 프로그래밍에 대한 이해를 할 수 있었다. 
  - 
- `convert(_:, to:)` 그리고 `convert(_:, from:)` 
  - 지난 주에 공부했던 `UIView` 메소드 중 하나인 `convert` 메소드를 기록하였다. 
  - [정리한 글](https://ehdrjsdlzzzz.github.io/2019/01/13/convert-to-convert-from/)

- [Building Better Apps with Value Types in Swift](https://developer.apple.com/videos/play/wwdc2015/414/)

  - 영상을 몇 번 보면서 내용들을 이해하고 있다. 하지만 완벽하게 모든 내용을 파악한 것은 아니다. 하지만 코드를 보는 것만으로도 느끼고 배워가는 것이 많은 세션이었다. 
  - 기존에 자바를 학교 강의로 공부하면서 구조체는 거의 사용하지 않았다. 항상 클래스를 사용해서 코드를 작성하도록 교육받았었다. 그래서 스위프트에서 나의 선택지는 항상 클래스였다. 세션에서 언급하는 문제점들을 느끼고 있음에도 불구하고 나의 선택은 클래스였다. 구조체를 사용할 명확한 명분을 찾지 못했기 때문이다.
  - 그래서 스위프트의 원시 타입과 콜렉션 타입들, 그리고 다른 많은 타입들이 왜 구조체로 되어있는지 이해를 하지 못하였다. 하지만 세션을 들으며 공부하면서 조금은 그들의 의도를 느낄 수 있었다(물론 아직도 100프로는 아니다.😅 정말 많은 공부가 필요하다!)
  - 이 세션을 계기로 클래스와 구조체의 선택에 있어서 보다 신중하게 선택해야 한다는 생각을 갖게 되었고 앞으로 코드를 작성함에 있어서 이 부분을 항상 인지하고 작성해야겠다.
  - 글은 현재 정리 중이다! 월요일이나 화요일 중에 포스팅할 수 있도록 노력해야겠다!

- [How Strings and Substrings work in Swift](https://medium.com/@studymongolian/how-strings-and-substrings-work-in-swift-fd4dc43ee91d) 

  - 스위프트에서 문자열과 부분 문자열을 다루는 방법은 다른 언어와는 사뭇 다르다. 그리고 부분 문자열을 다루는 데 있어서 인덱싱을 하는 과정이 귀찮고 번거롭다...!!
  - 이전에도 한번 다룬 주제이지만 매번 인덱싱을 할 때마다 구글링을 하는 나를 보고 이 글을 다시 정독하고 정리하게 되었다. 굉장히 쉽게 잘 설명된 글이다.
  - [정리한 글](https://ehdrjsdlzzzz.github.io/2019/01/12/Strings-and-Substrings-in-Swift/)

- 사용자 정의 뷰 컨트롤러 전환

  - 나는 개인적으로 부드럽고 멋진 애니메이션에 관심이 많다. 그리고 최근 몇 달간 나의 관심은 iOS 11부터 바뀐 앱스토어였다. 
  - 특히 투데이에서 볼 수 있는 화면 전환이 나는 너무 신기했고 구현해보고 싶었다. 여러 글들을 찾아본 결과 단순 뷰 애니메이션이 아닌 뷰 컨트롤러 전환을 커스텀 한 것이었고 이와 관련된 프로토콜들에 대해서도 알게 되었다. 
    - [UIViewControllerTransitioningDelegate](https://developer.apple.com/documentation/uikit/uiviewcontrollertransitioningdelegate)
    - [UIViewControllerAnimatedTransitioning](https://developer.apple.com/documentation/uikit/uiviewcontrolleranimatedtransitioning)
  - 하지만 그들을 알았다고 원하는 화면을 구현하는 데는 많은 어려움이 있었고 Raywederlich를 참고하여 이들의 사용법을 익힐 수 있었다. 그리고 원하는 모습을 얼추 비슷하게 구현할 수 있었다.
    - [Custom UIViewController Transitions: Getting Started](https://www.raywenderlich.com/322-custom-uiviewcontroller-transitions-getting-started)
    - [Reproducing Popular iOS Controls](https://www.raywenderlich.com/5298-reproducing-popular-ios-controls/lessons/25)

  - 다음은 그 결과!!!!! 이 과정은 천천히 포스팅해 볼 예정이다. 아직 기능이 완성된 것도 아니니..!!!

  ![AppStoreTransition](https://ehdrjsdlzzzz.github.io/2019/01/13/Weekly-Review-2/AppstoreTransition.gif)

---

### 한 주 동안 나는 무엇을 읽었을까?

저번 주에 목표로 했던 로버트 C. 마틴의 클린 코드 책을 구매하고 받았다! 안타깝게도... 현재까지 읽은 양은 매우 적다. 항상 모니터로 보던 글을 종이 책으로 보려니 정말 어색했다. (물론 이건 핑계 중 하나...)  하루의 마무리를 이 책을 읽으며 하기로 했던 다짐이 와르르 무너진 한 주였다. 정말 제대로 정독하며 읽어야 하는데 밤이 되면 체력부터 무너지게 되더라...!! 다음 주부턴 이 계획을 지킬 수 있도록 전체적으로 체력 분배를 제대로 해야겠다. 생각보다 앉아서 코드를 작성하고 영상을 보며 이를 정리하는 과정에 적지 않은 체력이 소모되더라..!! 

공부하며 읽은 글

- [How Strings and Substrings work in Swift](https://medium.com/@studymongolian/how-strings-and-substrings-work-in-swift-fd4dc43ee91d) 
- [Custom UIViewController Transitions: Getting Started](https://www.raywenderlich.com/322-custom-uiviewcontroller-transitions-getting-started)

이번 주에는 읽어보고 싶었던 [SwiftBySundell](https://www.swiftbysundell.com)의 글을 읽어보지 못했다. 여유가 있는 시간이 있음에도 불구하고 그 시간을 생산적으로 보내지 못했다. 이동하는 중간에 게임을 하기보단 블로그 글을 정독하는 습관을 길러야겠다.

---

### 한 주 동안 나는 무엇을 보았을까?

생각보다 WWDC 영상을 보고 이해하며 이를 정리하고 기록하는 일련의 과정이 상당히 많은 시간을 필요로 했다. 그렇게 길지 않은 영상인데도 말이다...!! 다음 주에도 이 영상을 반복해서 보며 이해한 것만이라도 내용을 정리하여 기록해보아야겠다.

이번 주에 본 영상

- [Building Better Apps with Value Types in Swift](https://developer.apple.com/videos/play/wwdc2015/414/)

다음 주에 볼 영상

- [Swift 성능 이해하기: Value 타입, Protocol과 스위프트의 성능 최적화](https://academy.realm.io/kr/posts/letswift-swift-performance/)
  - WWDC 영상과 관련 있는 내용으로 한 번 살펴볼 예정이다. 

그리고 위에서 언급했듯이 나는 UI와 애니메이션에 관심이 많다. 그리고 그와 더불어 UX에도 관심이 맣ㄴ다. 그래서 두 어플들을 사용해보면서 앱 안에서의 UI와 애니메이션들을 보며 어떻게 만들어졌는지 고민해보았다. 

- [트리플](https://itunes.apple.com/kr/app/트리플-해외여행-가이드/id1225499481?mt=8)
- [News10](https://itunes.apple.com/kr/app/news10/id1245535228?mt=8)

트리플은 비교적 News10에 비해 화려한 UI와 애니메이션을 구현하고 있었다. News10은 비교적 단순하지만 사용성이 좋은 UX를 갖고 있는듯한 느낌을 받았다. 이런 멋진 앱들을 보면 어떻게 만들었는지가 너무 궁금하다. 기회가 된다면 이 둘을 직접 구현해보고 이를 기록해보고 싶다.

---

### 정리하는 글

이번 주는 원하는 만큼 시간을 생산적으로 보내지 못했다..!! 이 부분은 정말 반성하고 있다. 시간이 없는 것은 절대적으로 아니었다. 물론 여유로웠던 것은 아니지만 내 노력하에 충분히 계획했던 것들을 실행에 옮길 수 있는 시간이었다고 생각한다. 

이번 주는 약간 샛길로 빠진 느낌이 없지 않아 있지만 이제 곰튀김님의 오프라인 강의까지 얼마 남지 않았다..!! 그전까지 리액티브 프로그래밍에 대한 지식을 어느 정도 쌓아야 한다....!! WWDC 영상의 기록을 마무리하면 리액티브 프로그래밍에 전념해야겠다!

> 제발 시간을 생산적으로 사용하자...!!!