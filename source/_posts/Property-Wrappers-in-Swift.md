---
title: Property Wrappers in Swift
date: 2020-01-03 21:27:18
tags: ["@Binding", "@State", "Property Wrappers"]
category: ["Swift"]
---

## Property Wrapper in iOS

Property wrapper는 프로퍼티를 정의하는 코드(*code that defines a property*)와 프로퍼티가 어떻게 저장되는지를 관리하는 코드(*code that manages how a property is stored*) 사이의 계층(*layer*)이다.

{% asset_img property_wrapper.png %}

Property wrapper를 사용하면 정의할 때 관리 코드(management code)를 한 번만 작성하고 여러 프로퍼티에 **재사용**할 수 있다. 사실 이런 wrapper 개념은 이번에 갑자기 나타난 것이 아니다. `lazy`나 `@NSCopying` 같은 키워드도 wrapper의 한 종류다. 하지만 Swift 5.1에선 개발자가 이런 wrapper를 직접 만들 수 있게 되었다.

Property wrapper를 이해하기 위해선 Property wrapper가 어떤 문제를 해결할 수 있는지를 보면 된다. 객체 안의 프로퍼티에 값이 할당될 때마다 로그를 출력해주어야 한다고 가정해보자. 우린 다음과 같이 코드를 작성할 수 있다. 

```swift
struct Bar {
  private var _x = 0

  var x: Int {
    get { _x }
    set {
      _x = newValue
      print("New value is \(newValue)") 
    }
  }
}

var bar = Bar()
bar.x = 1 // Prints 'New value is 1'
```

가장 직관적인 방법이다. 하지만 객체 안에 `x`뿐만 아니라 수많은 프로퍼티가 존재하고 이런 프로퍼티 모두가 값이 할당될 때마다 로그를 출력해주어야 한다면 상황은 달라진다. 

이런 상황을 해결하기 위해 우린 새로운 타입을 정의해줄 수 있다. 

```swift
struct ConsoleLogged<Value> {
  private var value: Value

  init(wrappedValue: Value) {
    self.value = wrappedValue
  }

  var wrappedValue: Value {
    get { value }
    set { 
      value = newValue
      print("New value is \(newValue)") 
    }
  }
}
```

그리고 기존의 `Bar` 구조체를 아래와 같이 `ConsoleLogged` 타입을 사용해 바꿀 수 있다.

```swift
struct Bar {
  private var _x = ConsoleLogged<Int>(wrappedValue: 0)

  var x: Int {
    get { _x.wrappedValue }
    set { _x.wrappedValue = newValue }
  }
}

var bar = Bar()
bar.x = 1 // Prints 'New value is 1'
```

이렇게 `ConsoleLogged` 타입을 사용해 중복 코드를 줄일 수 있다. Swift 5.1에선 이런 패턴을 Property wrapper라는 *Syntatic Sugar*로 제공한다. 사용 방법은 매우 간단하다. 기존의 `ConsoleLogged`에 `@propertyWrapper`만 붙이면 된다. `@propertyWrapper`를 타입(`struct`, `class`, `enum`) 앞에 붙이고 `wrappedValue`만 정의해주면 된다.

```swift
@propertyWrapper
struct ConsoleLogged<Value> {
  private var value: Value

  init(wrappedValue: Value) {
    self.value = wrappedValue
  }

  var wrappedValue: Value {
    get { value }
    set { 
      value = newValue
      print("New value is \(newValue)") 
    }
  }
}
```

> 프로퍼티가 어떻게 저장되는지를 관리하는 코드(*code that manages how a property is stored*) 

이렇게 선언한 Property wrapper는 사용하기도 쉽다. 아래와 같이 두 가지 방법으로 Property wrapper를 사용할 수 있다.

```swift
struct Bar { 
  @ConsoleLogged var x = 0 // 1
  @ConsoleLogged(wrappedValue: 2) var y // 2
}
```

> 프로퍼티를 정의하는 코드(*code that defines a property*)

Property wrapper 안에는 `wrappedValue`와 같이 프로퍼티뿐만 아니라 메소드도 정의할 수 있다. 하지만 안에 정의된 메소드를 사용할 땐 스코프(*scope*)에 주의할 필요가 있다. 

```swift
@propertyWrapper
struct Wrapper<T> {
  var wrappedValue: T

  func foo() { print("Foo") }
}
```

위와 같이 Property wrapper안에 `foo`라는 메소드를 정의했다.

```swift
struct HasWrapper {
    @Wrapper var x = 0

    func foo() { _x.foo() }
}
```

그럼 당연히 `Wrapper`를 Property wrapper로 사용하는 객체 내부에선 `foo` 메소드를 사용할 수 있다. (`_x.foo()`)

```swift
let a = HasWrapper()
a._x.foo() // ❌ '_x' is inaccessible due to 'private' protection level
```

하지만 위와 같이 외부에서의 접근은 불가능하다. 이유는 `private` 수준의 접근 제어를 갖기 때문이다. 물론 외부에서 접근할 수 있는 방법은 존재한다. `projectedValue`를 사용하면 된다. 

우리는 `projectedValue`를 통해 부수적인 API를 외부에 노출시킬 수 있다. `wrappedValue`와 다르게 `projectedValue`는 타입의 제한이 없다. 

```swift
@propertyWrapper
struct Wrapper<T> {
  var wrappedValue: T

  var projectedValue: Wrapper<T> { return self }

  func foo() { print("Foo") }
}
```

그리고 우리는 `projectedValue`를 `x`에 `$`을 붙여 접근할 수 있다.  

```swift
let a = HasWrapper()
a.$x.foo() // Prints 'Foo'
```

SwiftUI엔 `@State`, `@Binding`과 같은 빌트인 Property wrapper가 존재한다. 이에 대해선 추후 다루게 될 SwiftUI 포스팅에서 하나씩 살펴보도록 하자. 

이번 포스팅에선 간단한 예로 `UserDefaults`를 Property wrapper를 통해 보다 간편하게 사용할 수 있는 예제를 살펴보자.

```swift
@propertyWrapper
struct UserDefault<T> {
  var key: String
  var initialValue: T
  var wrappedValue: T {
    set { UserDefaults.standard.set(newValue, forKey: key) }
    get { UserDefaults.standard.object(forKey: key) as? T ?? initialValue }
  }
}
```

먼저 `UserDefault`라는 Property wrapper를 위와 같이 정의했다. `initialValue`를 통해 값이 존재하지 않을 때 초기값을 제공할 수 있다. 그리고 아래와 같이 사용할 수 있다. 

```swift
enum UserPreferences {
  @UserDefault(key: "isCheatModeEnabled", initialValue: false) static var isCheatModeEnabled: Bool
  @UserDefault(key: "highestScore", initialValue: 10000) static var highestScore: Int
  @UserDefault(key: "nickname", initialValue: "cloudstrife97") static var nickname: String
}

UserPreferences.isCheatModeEnabled = true
UserPreferences.highestScore = 25000
UserPreferences.nickname = "squallleonhart"
```

물론 Property wrapper로 선언된 프로퍼티에도 한계가 존재한다. 

- 하위 클래스에서 오버라이딩될 수 없다. 
- `lazy`, `weak`, `@NSCopying`과 같이 사용할 수 없다. 
- 커스텀 `set`, `get`을 사용할 수 없다.
- 프로토콜이나 익스텐션에서 선언될 수 없다. 



오늘은 Swift 5.1에 새로 추가된 Property wrapper에 대해 매우 간단히 알아보았다. 이후에는 SwiftUI에서 사용되고 있는 Property wrapper들에 대해서 소개해보려 한다. 



------

**참고 자료**

- [The Complete Guide to Property Wrappers in Swift5](https://devsday.ru/blog/details/3752)
- [Understanding Property Wrappers in Swift By Examples](https://medium.com/swlh/understanding-property-wrappers-in-swift-by-examples-604206022b5c)
- [Swift Docs](https://docs.swift.org/swift-book/LanguageGuide/Properties.html)