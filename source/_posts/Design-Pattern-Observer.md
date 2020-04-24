---
title: Design Pattern - Observer
date: 2019-04-16 21:30:06
tags: ["Observer Pattern", "KVO", "NotificationCenter"]
category: ["Design Pattern"]
---

오늘 살펴볼 디자인 패턴은 **옵저버(Observer) 패턴**이다. 말 그대로 관찰자다. 즉 관찰하는 대상에 변화가 생겼을 때 반응해주도록 하는 패턴이다. 옵저버 패턴은 많은 곳에서 유용하게 사용될 수 있다. 



### Observer Pattern

---

옵저버 패턴에는 두 가지 주체가 존재한다. **관찰을 당하는 객체(Observable)**, **관찰을 하는 객체(Observer)**, 우리는 이 두 객체를 이용해 옵저버 패턴을 사용할 수 있다. 

바로 코드를 통해 살펴보자. 

```swift
protocol ObservableProtocol: class { 
  var observers: [ObserverProtocol] { get set }
  
  func add(_ observer: ObserverProtocol)
  func remove(_ observer: ObserverProtocol)
  func notify(_ observers: [ObserverProtocol])
}

protocol ObserverProtocol { 
  var id: String { get set }
}
```

먼저 두 프로토콜을 정의해주었다. 

`ObservableProtocol`은 자신의 변화를 알려주는, 즉 관찰을 당하는 객체가 구현해야 하는 프로토콜이다. 기본적으로 이렇게 관찰을 당하는 객체는 자신에게 변화가 발생하였을 때 그 *변화를 관찰자(Observer)들에게 알려줄 의무가 있다.* 그렇기 때문에 자신을 관찰하고 있는 *관찰자들을 알고 있어야* 하고 관찰자가 원한다면 *관찰자 목록에서 해제*하여 더 이상 변화를 알리지 않을 수 있다.  이것을 코드로 옮겨놓은 것이 바로 위의 `ObservableProtocol`이다. 

`ObserverProtocol`은 관찰자가 구현해주어야 할 프로토콜인데, 기본적으로 관찰의 상대 객체에는 여러 관찰자들이 붙을 수 있고, 변화에 따른 관찰자들의 행동은 각기 다를 수 있기 때문에 각각의 관찰자들은 이 `id`로 구분된다. 

그럼 이제 구현체를 작성해보자. 

```swift
class Observable<T>: ObservableProtocol { 
  typealias CompletionHandler = ((T) -> Void)

  // 1. 
  var value : T {
    didSet {
      self.notifyObservers(self.observers)
    }
  }

  var observers : [String : CompletionHandler] = [:]

  init(value: T) {
    self.value = value
  }

  // 2.
  func addObserver(_ observer: ObserverProtocol, completion: @escaping CompletionHandler) {
    self.observers[observer.id] = completion
  }

  // 3.
  func removeObserver(_ observer: ObserverProtocol) {
    self.observers.removeValue(forKey: observer.id)
  }

  // 4.
  func notifyObservers(_ observers: [String : CompletionHandler]) {
    observers.forEach({ $0.value(value) })
  }

  // 5. 
  deinit {
    observers.removeAll()
  }
}
```

코드를 하나씩 살펴보자. 

1. 제네릭 타입 `T`로 관찰할 값의 타입을 선언한다. 값이 할당되면(`didSet`), `notifyObservers(_:)` 메소드를 통해 값을 관찰하고 있는 관찰자 객체들에 값의 변화를 알린다. (`notifyObservers(_:)`) 메소드는 밑에서 살펴보자.
2. 값에 관찰자를 추가한다. 이때 관찰자와 값의 변화에 따른 관찰자의 행동은 딕셔너리 형태로 저장되고 이때 관찰자 고유의 값인 `id`가 키값이 된다. 

- 관찰자(`ObserverProtocol`)을 직접 저장하는 것에 비해 메모리 누수에 대한 걱정을 덜 수 있다. 

3. 키-값 형태의 딕셔너리로 저장되어 있기 때문에 관찰자의 `id` 값을 통해 해당 관찰자의 핸들러를 지울 수 있다. 
4. 값이 할당되었을 때 호출되는 메소드로 저장되어 있는 딕셔너리를 순회하며 딕셔너리의 값, 즉 저장된 관찰자 핸들러들을 호출해준다. 이 메소드는 값이 바뀐 후 호출되기 때문에 파라미터로 넘어가는 값은 새로 바뀐 값이 된다. 

간단한 예제를 통해 동작을 살펴보자. 

```swift
struct Children {
  var age : Observable<Int>
  var isGraduationDay: Observable<Bool> = Observable(value: false)
  init(age: Int) {
    self.age = Observable(value: age)
  }
}


struct Parent: ObserverProtocol {
  var id = UUID().uuidString
}

let bobby = Children(age: 10)
let mommy = Parent()
bobby.age.addObserver(mommy) { newAge in
  print("Let's throw a birthday party")
}

bobby.isGraduationDay.addObserver(mommy) { isGraduationDay in
	print("Let's throw a graduation party")
}

bobby.age.value = 11
// Let's throw a birthday party
bobby.isGraduationDay.value = true
// Let's throw a graduation party
```

이렇게 옵저버 패턴은 일 대 다수의 단방향 통신을 지원한다. 우리는 이렇게 객체 간의 일 대 다수의 통신을 할 수 있는 것에 익숙한데 바로 KVO와 `NotificationCenter`다. 

**NotificationCenter, KVO**

그럼 이 둘과 우리가 작성한 옵저버 패턴의 차이점은 무엇일까? 간단하게 살펴보자. 

먼저 `NotificationCenter`를 사용하는 데 있어서 이벤트 송신자와 수신자는 서로의 존재를 몰라도 사용이 가능하다. 하지만 직접 이벤트를 코드로 발생시켜줘야 한다는 단점(`NotificationCenter.default.post`)이 존재한다. 그리고 단순히 이벤트의 발생을 알리는 것이지 변화된 값의 전달은 역시 따로 지정해주어야 한다. 

반면 KVO는 값에 변화가 생기면 자동으로 그 변화를 알리며 변화된 값도 함께 알리지만 이를 사용하기 위해선 `NSObject`를 상속받아야 하고 `observe` 메소드를 오버라이딩해야 하며 관찰을 하기 위한 값에 `dymamic`을 붙여주어야 한다. 

반면에 우리가 작성한 옵저버 패턴은 위의 코드만 구현하면 상속, 오버라이딩을 하지 않아도, 직접 이벤트 발생을 알리지 않아도 값의 관찰과 이에 따른 행동을 정의하고 취할 수 있다.

오늘은 이렇게 옵저버 패턴에 대해 알아보았다.

