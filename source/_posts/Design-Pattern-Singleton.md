---
title: Design Pattern - Singleton
date: 2018-10-28 13:14:27
tags: ["Swift","iOS","Singleton"]
category: ["iOS", "Design Pattern"]
---

프로젝트를 진행하면서 남발했던 **싱글톤(Singleton)**에 대해 자세히 알아보고자 한다. 많은 개발자들이 안티 패턴이라 말하는 이유와 그럼에도 사용되는 이유 그리고 싱글톤 그 자체의 존재 의미를 다뤄보고자 한다. 

바로 위에서 말했듯이 나는 싱글톤을 아무 이유 없이 사용해왔다. 사실 아무 이유가 없진 않다. **어디서든지** **편하게** 접근하기 위해 싱글톤을 사용했다. 근데 그것이 싱글톤이 존재하는 이유고 많은 개발자들이 사용하는 이유일까? 하나씩 알아보자.

---

### 싱글톤(Singleton)이란?

간단하게 싱글톤을 정의하자면 앱의 생명 주기 동안 하나의 인스턴스만 생성을 보장하는 클래스를 의미한다. 이는 단순히 하나의 인스턴스 생성을 보장하는 것이 아니라 앱 전체에서 공유될 수 있는 자원들을 캡슐화한 객체라고 할 수 있다. 다음과 같이 클래스를 정의한다면 외부에서는 해당 클래스의 인스턴스는 생성할 수 없고 오직 `shared` 변수를 통해서만 접근할 수 있다. 

```swift
class SingleTonClass {
    static let shared = SingleTonClass()
    
    private init() {}
    
    func doSomething() {
        
    }
}

SingleTonClass.shared.doSomething()
```

이렇게 생성된 싱글톤 객체는 프로그램 전역에서 접근이 가능하다. 전역에서 가능하다..!! 그렇다 전역 변수와 비슷하다. 처음 프로그래밍을 배울 때 전역 변수를 지양하라는 이야기를 많이 들었다. 어디서든지 접근이 가능하다면 누구나 값을 바꿀 수 있고 전역 변수의 상태를 예측하기 힘들며 전역 변수가 많아지면 불필요한 메모리 역시 차지하게 된다는 단점들이 있다. 

싱글톤도 마찬가지다. 이에 대한 내용은 밑에서 좀 더 자세히 살펴보도록 하자. 

하지만 이런 여러 안티 패턴이라는 이야기와는 다르게 iOS 프레임워크 내에선 적지 않은 싱글톤 객체들을 찾아볼 수 있다. 

```swift
// Shared URL Session
let sharedURLSession = URLSession.shared

// Default File Manager
let defaultFileManager = FileManager.default

// Standard User Defaults
let standardUserDefaults = UserDefaults.standard

// Default Payment Queue
let defaultPaymentQueue = SKPaymentQueue.default()

```

하지만 이들이 제공하는 싱글톤 객체는 내가 프로젝트 내에서 사용했던 싱글톤과는 달랐다. 그들은 이 객체들을 활용한 **기본적인 행위**를 싱글톤 객체로 제공하였으면서도 필요하다면 충분히 직접 만들어서 사용할 수 있도록 하였다. 그럼 이제 싱글톤의 장/단점을 좀 더 자세히 살펴보도록 하자.

---

### 싱글톤의 장/단점

사실 싱글톤을 제대로 공부하기 위해 구글링을 통해 많은 글들을 찾아보았으나 정확히 *싱글톤은 이때 사용해라!* 라는 글은 찾아볼 수 없었다. 모두 하나같이 싱글톤의 단점을 설명하였다. 그들이 말하는 개발자들이 사용하는 이유는 바로 **편리함**이었다. 싱글톤 객체는 확실히 만드는 입장이나 그것을 사용하는 입장이나 확실히 편리하게 사용할 수 있다.

그래서 이를 두고 ***편리함을 위해 투명성을 희생한다*** 라고 표현하곤 한다.

그럼 먼저 단점을 살펴보자. 다음은 내가 여러 글들을 참고하면서 알게 된 단점들을 두 가지로 추려본 것이다. 

- **They are global mutable shared state**
  - 어디서든 접근이 가능하고 누구든지 상태를 변경할 수 있기 때문에 사용하는 시점에 우리는 싱글톤 객체의 상태를 예측하기 힘들 수 있다. 이렇게 예측이 불가능하기 때문에 싱글톤 객체의 생명 주기를 관리하는 것도 어려워진다. 만일 싱글톤 객체가 옵셔널한 프로퍼티를 갖게 된다면 사용하는 시점에 더욱이 그 상태를 예측하기 힘들 것이다. 
- **Singletons hinder unit testing**
  - 싱글톤 객체를 사용하는 객체들은 싱글톤 객체에 강한 의존성을 갖고 있기 때문에 이 둘은 강하게 묶여있고 그렇기 때문에 싱글톤 객체를 포함하는 객체를 테스트하기 위해선 싱글톤 객체를 온전히 구현해주어야 한다. 

프로젝트 내에 싱글톤 객체가 사용되기 시작하면 프로그램 전체가 싱글톤 객체에 의존하게 될 수 있으며 이렇게 의존성이 생기게 된다면 싱글톤 객체를 제거하는데 많은 시간이 소요될 것이다.

개인적으로 고민을 해보았을 때 하나의 **객체에 대한 접근 방법 중 하나**로써의 싱글톤은 하나의 옵션이 될 수 있을 것 같다. 하지만 싱글톤으로만 접근 가능하게 만드는 것은 최대한 지양해야 하는 것 같다.

싱글톤에 대해 공부를 하다 보면 많은 글들에서 의존성 주입을 싱글톤을 피할 수 있는 방법 중 하나로 제시하곤 한다. 이제는 의존성 주입에 대해 코드와 함께 살펴볼 것이다.

> 의존성 주입에는 고려하고 알고 있어야 하는 것들이 굉장히 많다고 알고 있다. 하지만 이번 포스팅에서는 대략적인 접근법만을 다뤄볼 예정이다. 

---

### 의존성 주입

> 코드는 본인의 [GitBingo](https://github.com/ehdrjsdlzzzz/GitBingo) 프로젝트를 리팩토링하는 과정을 참고하였다. 

기존의 프로젝트에서 네트워크 통신을 하는 객체를 싱글톤으로 구현하였다. 하지만 테스트를 하는 과정에서 이 싱글톤 객체는 테스트 코드 작성을 굉장히 힘들게 하였으며 사실 이 객체를 싱글톤으로 갖고 있을 필요도 없었다. 

```swift
func APIService {
    static let shared = APIService()
    private let session = URLSession(configuration: .default)		     
    private let session = URLSession(configuration: .default)
    
    private init() {}
    
    func fetchContributionsDots(of id: String, completion: @escaping (Contribution?, GitBingoError?) -> ()) {
        ...
        parseHTML()
        ...
        // Fetch
    }
    
    private func parseHTML() {
        
    }
}
```

이 코드에는 사실 문제점이 싱글톤뿐만이 아니였다. 데이터를 받아오는 과정에서 HTML을 파싱 해야 하는데 객체 안에 받아온 HTML을 파싱 하는 역할까지 부여하였고 이는 **SOLID**의 **단일 책임 원칙에 위배**된다고 생각하였기 때문이다. 그래서 싱글톤 패턴과 이 문제점을 해결하기 위해 본인은 프로토콜을 이용해 추상화를 해주었다. 

```swift
protocol APIServiceProtocol: class {
    init(parser: HTMLParsingProtocol)
    func fetchContributionDots(of id: String, completion: @escaping (Contribution?, GitBingoError?) ->())
}

protocol HTMLParsingProtocol: class {
    func parse(from data: Data?) -> Contribution?
}
```

그리고 이들을 채택한 클래스를 정의해주었다.

```swift
class Parser: HTMLParsingProtocol {
    func parse(from data: Data?) -> Contribution? {
        // Parse
        return Contribution(dots: dots)
    }
}

class APIService: APIServiceProtocol {
    //MARK: Properties
    private let session = URLSession(configuration: .default)
    private let parser: HTMLParsingProtocol
    
    required init(parser: HTMLParsingProtocol) {
        self.parser = parser
    }
    
    //MARK: Methods
    func fetchContributionDots(of id: String, completion: @escaping (Contribution?, GitBingoError?) -> ()) {
        // Fetch
    }
}
```

그리고 이를 사용하는 Presenter의 생성 시점에 주입해준다. 

```swift
class MainViewPresenter {
    private var service: APIServiceProtocol
    
    init(service: APIServiceProtocol) {
        self.service = service
    }
}
```

이렇게 프로토콜을 따라는 객체를 생성자에 넣어줌으로써 우리는 해당 객체가 어디에 의존성을 갖고 있는지 명확하게 알게 되고 이를 통해 투명성이 증가하게 된다. 그리고 이렇게 분리한 프로토콜로 테스트를 위한 Mock 객체를 보다 간단하게 작성할 수 있게 된다.

---

### 마무리

의존성 주입이라는 것도 절대 쉽게 볼 수 있는 주제는 아닌 것 같다. 그리고 본인이 이를 정확하게 사용하지 않았을 수도 있다. 하지만 늘 생각 없이 편리함만을 위해 사용했던 싱글톤 패턴이 생각보다 많은 문제점을 내포하고 있다는 사실을 알게 되었고 이를 해결하기 위한 방법 중 하나로 의존성 주입이 사용될 수 있다는 사실을 알게 된 것만으로도 앞으로 프로그램을 어떻게 작성해야 할지에 대한 생각을 키우는 계기가 된 것 같다. 또한 이를 계기로 의존성 주입에 대해 공부를 더욱 자세히 해볼 예정이다. 

> 글이나 코드를 읽으면서 잘못되었거나 부족한 부분이 있다면 언제든 피드백 부탁드립니다. 

---

**참고 자료**

- [What is Singleton and How to create one in swift](https://cocoacasts.com/what-is-a-singleton-and-how-to-create-one-in-swift)

- [Are Singletons Bad](https://cocoacasts.com/are-singletons-bad)
- [Avoiding singletons in Swift](https://medium.com/@johnsundell/avoiding-singletons-in-swift-5b8412153f9b)
- [Let's examine the pros and cons of the Singleton design pattern](https://medium.freecodecamp.org/singleton-design-pattern-pros-and-cons-e10f98e23d63)

