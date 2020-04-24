---
title: Weekly Review(14)
subtitle: 20190401-20190407
date: 2019-04-07 20:59:39
tags: ["Hit Testing", "Result", "Swift5", "클린 코드"]
category: ["Weekly Review"]
---

이번 주는 회사 일로 개인 공부에 많은 시간을 투자하지 못했다. 그래도 일단 무엇을 했는지 정리해보는 시간을 갖자!



### 한 주 동안 나는 무엇을 공부했을까? 

---

- Hit Testing

  - 예전에 Hit Testing부터 Responder Chain까지 사용자 이벤트를 받고 처리하는 과정에 대해 훑어본 적이 있다. 그중에서 제일 흥미로웠던 부분이 Hit Testing 이었다. 간단히 Hit Testing의 역할을 설명하자면 터치가 일어난 최상단 뷰를 찾는 과정이라 할 수 있다.
  - 이에 대해 다시 살펴본 계기는 코드스쿼드 슬랙방에서 @Mason이 올린 질문에서 시작되었다. 단순히 정리하면 하나의 뷰에서 `hitTest` 함수가 두 번 호출되는데 그 이유가 궁금하다는 질문이었는데…! 나도 그 이유가 궁금했고 여러 자료들을 찾아보았다. 하지만 그 어느 곳에서도 명확한 이유를 말해주지 않았고 *"그냥 내부에서 그렇게 작동해~"* 와 같은 답변뿐이었다. 
  - 하지만 그 중에서도 와닿는 답변이 존재했는데 그 내용은 이러하다. 
    - *hitTest: is a utility method designed to find a view at a specific point. It DOES NOT represent a user tapping on the touch screen. It is totally sensible for hitTest to be called several times in response to the same event; all the method is supposed to do is return the view under the point and it SHOULD NOT trigger any side effects.*
      - 간단히 요약하자면 이벤트가 발생한 뷰를 찾는 과정에 사용되는 메소드이지, 하나의 터치 이벤트에 한 번 실행되는 그런 메소드가 아니라는 이유다. 
    - 또한 다음과 같은 답변도 존재했다. 
      - *Yes, it’s normal. The system may tweak the point being hit tested between the calls. Since hitTest should be a pure function with no side-effects, this should be fine*
  - 두 답변 모두 `hitTest`는 부수 효과가 없어야 하는 메소드이기 때문이라는 부연 설명을 덧붙인다. 하긴 터치 이벤트가 발생할 때마다 그 이외에 다른 곳에 영향을 주면 그게 이상한 것이라 생각한다! 그래도 아직 명확한 이유는 알아내진 못했지만 어느 정도는 납득은 된 것 같다. 

- Swift 5 - `Result`

  - 이번 Swift 5에서 변경된 점 중 가장 직관적인 부분이 바로 `Result` 타입이다. 말 그대로 어떤 결과를 표현하는 타입인데 적지 않은 곳에서 사용될 수 있다는 생각이 든다. 
  - 이전 주간 회고록에서도 언급했는데 최근에 나는 네트워크 계층을 효과적으로 작성하는 방법에 대해 공부하고 있다. `Result` 타입 역시 이 부분에서 유용하게 사용될 수 있을 것 같다. 
  - 네트워크 호출 코드를 작성하면 다음 코드는 굉장히 낯이 익을 것이다.

  ```swift
  ... {data, response, error in
      if error != nil { 
      	// Handle Error
          return 
      }
       
      // Handle Data
  }
  ```

  - 에러가 있다면 에러를 처리해주고 아니라면 받아온 데이터를 처리하는 코드이다. 우리는 `Result` 타입을 이용해 이를 보다 직관적이고 효율적으로 처리해줄 수 있다. 

  ```swift
  switch result { 
  case .success(let data):
      // Handle Data
  case .failure(let error):
      // Handle Error
  }
  ```

- 알고리즘
  - 역시 알고리즘 문제를 매일매일은 아니지만 적지 않게 풀어보았다…! 이번 주는 그나마 생긴 재미도 떨어져 나간 한 주 였다. 
  - 정말 알고리즘의 중요성을 깨닫게 하거나 자료구조의 활용방안에 대해 고민해볼 수 있는 알고리즘 문제라면 풀고도 혹은 풀이를 보고도 깨닫는 부분이 많다. 하지만…! 그게 아니라 이게 알고리즘 문제인지..아이큐 테스트인지..그런 문제는 풀고도 기분이 찝찝하고 해설은 더더욱 보기가 싫어진다.
  - 이것도 하나의 과정이니… 익숙해져야겠지만! 가끔 이런 문제를 만나면 재미가 팍 꺾이는 기분이다.



### 한 주 동안 나는 무엇을 보았을까?

---

위에서 언급한 Swift 5의 `Result` 타입을 가장 잘 사용한 예제라 할 수 있는 네트워크 호출 예제를 간단하게 보여주는 영상이 있다. (역시나 브웅형님)

- [Swift 5 Brand New Result Type : Write Cleaner API Code](https://youtu.be/9iazQQdNoNU)

기타 여러 영상들을 보았던 것 같은데.. 기억에 딱히 남지 않는거 보니 집중해서 본 것 같지 않다! (반성하자)



### 한 주 동안 나는 무엇을 읽었을까?

---

클린 코드를 계속 읽고 있는데 (속도가 무지 느리다..) 이번엔 4장 주석에 관한 내용이었다. 

나도 개인적으로 주석을 써야 이해가 가는 코드를 작성하면 괜히 찜찜하다. 역시 이에 대한 내용을 담고 있는데 주석은 최대한 지양해야 하고 주석 없이 코드만 봐도 의도가 드러나야 좋은 코드라고 설명을 하고 있다. 물론 주석의 적절한 예도 보여주긴 하지만 그래도! 그래도 주석은 지양해야 하는 게 좋다고 말하고 있다. 

주석을 쓰지 않아도 내가 의도한 바를 상대방에게 온전히 전달할 수 있는 코드…!! 프로그래머라면 누구나 작성하고 싶지 않을까 싶다! 



### 정리하는 글

---

이번 주부터 본격적으로 디프만 프로젝트를 시작했다. 역시 사이드 프로젝트는 재밌다! 프로젝트를 통해 공부하는 내용이 가장 와닿는 것 같다. 그리고 클린 코드를 읽으며 알게 된 내용을 최대한 접목시켜보려 하고 있다. 

요즘 Raywenderlich 1년 구독을 할까 정말 고민 중이다.. 가뜩이나 돈 나갈 곳도 많은데..!!