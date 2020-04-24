---
title: Design Pattern - Strategy
date: 2019-01-18 09:21:57
tags: ["iOS", "Swift", "Strategy", "Design Pattern"]
category: ["iOS", "Design Pattern"]
---

오늘은 **Strategy** 패턴에 대해 공부를 해보았다. 처음 정의를 보았을 땐 명확하게 다가오지 않았지만 공부를 하고 나선 SOLID의 내용과 겹치는 듯한 느낌을 받아 이해하는데 도움이 되었다. 


먼저 [위키피디아의 정의](https://en.wikipedia.org/wiki/Strategy_pattern)를 살펴보면 다음과 같다. 

*In computer programming, the* **strategy** pattern *is a behavioral software design* pattern *that enables selecting an algorithm at runtime. Instead of implementing a single algorithm directly, code receives run-time instructions as to which in a family of algorithms to use.*

*컴퓨터 프로그래밍에서 **strategy** 패턴은 행동 소프트웨어 디자인 패턴으로 런타임 중 알고리즘을 선택을 가능케 한다. 직접적으로 단일 알고리즘을 구현하는 대신 코드는 런-타임 중 알고리즘 그룹 중 어떤 것을 사용해야 할지에 대한 지시를 받는다.*

많은 경우에서 그랬듯이 위키피디아 정의는 한 번에 와닿는 경우가 많이 없었다. 여기서 다시 의문점이 생긴 부분은 바로 *behavioral software design pattern* 이었다. 대체 이것이 무엇을 의미하나 다시 [위키피디아의 정의](https://en.wikipedia.org/wiki/Behavioral_pattern)를 살펴보았다. 

*In software engineering, behaviroral design patterns are design patterns that identify common communication patterns among objects and realize theses patterns*

*소프트웨어 공학에서 행동 디자인 패턴은 객체 간의 공통된 통신 패턴을 식별하고 이러한 패턴을 실현하는 설계 패턴이다.*


그래도 Strategy 패턴의 정의보다는 조금 더 와닿는 느낌이다. 역시 코드를 살펴보아야 더 잘 이해할 수 있을 것 같은 느낌이 든다. 그렇다면 언제 이 패턴을 사용할까?


- 같은 작업을 다른 방법으로 수행할 때
- 상속을 대체하기 위해 
  - 클래스의 기능을 확장하기 위해 상속을 하는 것 대신 Strategy 패턴을 사용한다.
- 조건문을 대체하기 위해
  - 클래스 내부에 `if-else` 혹은 `switch`와 같은 조건문이 많이 존재한다면 이는 SOLID 중 **단일 책임 원칙**과 **개방-폐쇄 원칙**을 위배하고 있을 수 있다는 신호다. Strategy 패턴을 이용해 이를 분산시킬 수 있다. 



간단한 예를 들어보자.

```swift
struct Logger {
    enum Style {
        case lowercase
        case uppercase
        case capitalized
    }
    
    let style: Style
    
    func log(_ message: String) {
        switch style {
		case .lowercaes:
            print(message.lowercase())
		case .uppercase:
            print(message.uppercase())
		case .capitalized:
            print(message.capitalized
        }
    }
}
```

흔히들 코드에서 냄새가 난다고 한다. 냄새가 나는가? (사실 본인은 냄새를 맡지 못했다...! 코드의 냄새를 맡을 수 있는 후각을 길러야겠다!)  

`Logger` 클래스가 `style`을 **구분**하여 각기 다르게 출력해주는 것에 책임을 가질 필요가 있을까? (*`style`에 대한 정보를 갖고 있을 필요가 있을까?*) 또한 스타일이 20개 50개로 늘어난다면 어떻게 될까? 우리는 `log(_:)` 메소드에 케이스를 추가하는 수정을 해야 한다. 이 두 냄새는 각각 **단일 책임 원칙 위배**와 **개방-폐쇄 원칙의 위배**이다. 

이런 상황에서 우리는 Strategy 패턴을 사용해 위의 두 냄새를 제거할 수 있다.

```swift
protocol LoggerStrategy {
    func log(_ message: String)
}

struct Logger {
    let strategy: LoggerStrategy
    
    func log(_ message: String) {
        strategy.log(message)
    }
}

struct LowercaseStrategy: LoggerStrategy {
    func log(_ message: String) {
        print(message.lowercased())
    }
}

struct UppercaseStratgy: LoggerStrategy {
    func log(_ message: String) {
        print(message.uppercased())
    }
}

struct CapitalizedStrategy: LoggerStrategy {
    func log(_ message: String) {
        print(message.capitalized)
    }
}

let logger = Logger(strategy: LowercaseStrategy())
logger.log("Hello") // "hello"
```

여기서 우리는 Strategy 패턴 사용에 필요한 세 가지 요소를 확인할 수 있다.

- Strategy를 사용하는 객체
  - `Logger`
- Strategy 프로토콜
  - `LoggerStrategy`
- Strategy의 구현체
  - `LowercaseStrategy`
  - `UppercaseStrategy`
  - `CaptializedStrategy`

이제 `Logger`는 `style`을 구분하지 않고 단순히 받아온 메시지를 `strategy.log(_:)`에 넘겨주어 출력만 해준다. 더 이상 `style`에 대한 정보를 갖고 있지 않아도 되는 것이다. - **단일 책임 원칙**

그리고 스타일이 늘어난다고 해서 `Logger` 내부의 코드를 수정하지 않아도 된다. `LoggerStrategy` 프로토콜을 구현한 스타일 객체를 정의하고 이를 `Logger`의 생성 시점에 주입해주면 되기 때문이다. 즉 변경이 아닌 확장인 것이다. - **개방-폐쇄 원칙**

이렇게 객체의 책임을 분산시키고 기능의 로직을 외부에서 주입해준다면 **테스트 코드 작성**에도 용이할 것이다. 

---

**출처**

- [Strategy pattern in Swift](https://medium.com/flawless-app-stories/strategy-pattern-in-swift-1462dbddd9fe?fbclid=IwAR0Lo6URRF3p-8fSnZ5fq-lspQSPMFlSB8cqcoOQS0GXcfHazxtzX_x4oPY)
- [if-else vs SOLID principles](https://blog.mjouan.fr/if-else-trees-vs-solid-principles/)