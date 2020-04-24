---
title: Weekly Review(7)
subtitle: 20190211-20190217
date: 2019-02-19 23:40:35
tags: ["iOS", "SFSafariViewController", "MVVM", "Closures", "Delegates"]
category: ["Weekly Review"]
---

인턴을 시작하고 1주일이 지났다. 적응을 위한 기간이라 매우 정신이 없었고 정리가 되지 않은 상태였다. 그래서 퇴근 후의 시간 관리도 소홀해졌다. 그래서 개인적인 공부에 많은 시간을 투자하지 못했다. 그래도 최대한 자기 전의 시간을 활용하기로 했고 이를 통해 공부하고 알게 된 것들을 정리해보고자 한다. 

------

### 한 주 동안 나는 무엇을 공부했을까?

- `SFSafariViewController` 
  - 인턴을 진행하면서 2개의 과제를 부여받았다. 그 중 앱 내 사소한 버그를 수정하는 과제를 진행하면서 `SFSafariViewController`와 `SFAuthenticationSession`의 차이에 대해 알아보았다.
  - 물론 현재는 `SFAuthenticationSession`은 `deprecated` 되었고 이는 [`ASWebAuthenticationSession`](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession)로 대체되었다. 
  - `SFAuthenticationSession`은 기본적으로 `SFSafariViewController`와 다르게 인증 과정과 같이 민감한 정보가 오가는 상황에 사용된다. 해당 인증 과정이 다른 프로세스에서 진행이 되므로 앱은 해당 결과에 대한 일종의 토큰만을 부여받을 수 있다. 이는 **iOS 11**부터 사용이 가능했고 iOS 10버전에선 이 대신 `SFSafariViewController`가 사용되었고, 작업이 진행되는 중에 스레드의 문제로 버그가 발생하였었다. 
  - [정리한 글](https://ehdrjsdlzzzz.github.io/2019/02/13/SFAuthenticationSession-SFSafariViewController/)
- MVVM in iOS
  - 최근 몇 개월간의 나의 관심사와 공부 키워드는 클린 코드이다. 이를 위해 여러 아키텍쳐를 공부하는데 그중 많이 언급되는 MVVM에 대해 여러 글들을 읽어보며 전체적인 윤곽을 잡아보려 하고 있다. 
  - 그러한 글들 중 기초적이지만 부실하지 않았고 생각할 거리도 남겨준 [이 글](https://medium.com/@azamsharp/mvvm-in-ios-from-net-perspective-580eb7f4f129)을 번역하여 기록하였다. 
  - 앞으로 진행 예정 중인 개인 프로젝트에서 MVVM을 사용할 계획이기 때문에 여러 상황에서의 코드 구현 방법을 찾아보며 이에 대한 지식을 쌓아가고 있다. 



> 개인적으로 아키텍쳐에 대해 관심이 많고 그중 요즘 관심을 집중하고 있는 것은 MVVM인데, 사실 많은 고수분들이 말씀하시길 아키텍쳐는 정답이 없고 프로젝트에 가장 적합한 것을 골라야 한다고 말씀하신다. *왜 MVVM을 공부해야할까?* *왜 테스트 코드를 작성해야할까?* 에 대한 의문이 존재한다. 사실 단순히 지금은 iOS 오픈 채팅방에서 핫하고 다른 사람들이 모두 하니까 공부를 하고 있다. 물론 이렇게 사용되는 이유가 있을테지만 아직은 내가 반드시 이것을 사용해야 해서 공부한다는 느낌은 들지 않는다. 
>
> 그래서 아마 다음의 몇몇 포스팅에선 테스트 코드를 작성해야 하는 이유와 아키텍쳐를 사용하는 이유에 대해 조금 더 알아보고 그 본연의 주제들에 대해 공부를 해보려 한다. 사실 그 "이유"들에 대해서는 대충은 알고 있으나 이를 좀 더 머릿속에 명확하게 할 필요가 있다고 생각한다. 내가 **MVVM을 사용하는 명백한 이유**를 알고 싶다.

------

### 한 주 동안 나는 무엇을 읽었을까?

- [Mastering MVVM on iOS](https://medium.com/@mecid/mastering-mvvm-on-ios-f875d2b99816)
  - MVVM에 대해 소개하는 글로 단순히 개념만을 소개하는 것이 아닌 뷰 모델 컴포지션(Composition), 네트워크 호출을 다루는 방법을 제안하고 있다. 
- [MVVM in iOS](https://medium.com/@azamsharp/mvvm-in-ios-from-net-perspective-580eb7f4f129)
  - 역시 위와 비슷한 구성과 내용의 글이며 글 내부의 관련된 글이라고 첨부된 글도 읽어보는 것을 추천한다. 

이외에도 다수 읽었지만 훑어보는 수준이라 언급은 하지 않았다. 역시 MVVM에 대한 글을 많이 읽고 있다. 그리고 이외에 버그 수정 외에 진행 중인 과제에서 `UIAlertController`를 좀 더 효과적(?)으로 띄우기 위한 방법들에 대해 살펴보았고 `Error`를 처리하는 방법들에 대해 다른 개발자들은 어떻게 처리하고 다루고 있는지에 대해 알아보고 글들을 찾아 읽어보았다. 

------

### 한 주 동안 나는 무엇을 보았을까?

- [Use Closures not Delegates](https://youtu.be/vST4UN4s1kg)

  - 습관적으로 코드를 작성할 때 델리게이트 패턴을 사용하고 있었다. 인터페이스가 동일할 경우 프로토콜로 추상화를 진행했었고 이용해 해당 역할을 하는 객체들을 확장해주었다. 하지만 굳이 이렇게 할 필요가 없는 상황에서 델리게이트 패턴을 사용을 했던 적도 있었고 이 영상을 통해 클로져를 통해 보다 적은 코드로 원하는 효과를 얻어낼 수 있었다. 
  - 클로저를 사용한다면 따로 프로토콜을 정의해주고 델리게이트 변수를 선언해주고 프로토콜 메소드들을 **반드시** 구현해주어야 하는 번거로움을 덜 수 있다. 가끔은 필요 없음에도 불구하고 프로토콜에 정의되어 있기 때문에 빈 메소드를 정의해줘야 하는 경우가 종종 있다. 

  ```swift
  class MyTableViewCell: UITableViewCell {
      @IBOutlet weak var button: UIButton! 
      
      var buttonDidTapped: ((UIButton) -> Void)?
      ...
      
      @IBAcion func handleButtonDidTapped(_ sender: UIButton) {
          buttonDidTapped?(sender)
      }
  }
  
  class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
      
      ...
      
      func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
          let cell = ... 
          
          cell.buttonDidTapped = { _ in
              // Handler Codes 
          }
          
          return cell 
      }
  }
  ```

이에 대해서는 좀 더 자세히 공부해보고 포스팅해보려 한다. 

------

### 정리하는 글

인턴을 진행하면서 시간을 관리하는데 굉장히 큰 어려움을 느꼈으며 자괴감도 느꼈다. 출근을 하기 전 다짐했던 회사와는 별개의 나의 공부를 하는데도 많은 시간을 할애하지 못했다. 물론 직장에서 하는 일에 가장 많은 시간과 노력을 할애하는 것은 당연하지만 그와 별개의 공부를 하지 않는 것에 대해 도태 될 것만 같은 기분이 들었기 때문이다. 

새로 시작하는 주에서는 보다 균형을 잡고 스스로에게 보다 냉정한 판단을 내릴 수 있는 한 주가 되었으면 좋겠다. 

(피곤하다고 그냥 누워버리지 말자 바로 잔다…! 😭😭😭)