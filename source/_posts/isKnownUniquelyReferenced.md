---
title: isKnownUniquelyReferenced
date: 2019-07-23 19:37:36
tags: ["isKnownUniquelyReferenced"]
category: ["Swift"]
---

오늘은 [Modern Swift API Design - WWDC 2019](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=2ahUKEwicgb7g7crjAhVEG6YKHaOiCEAQwqsBMAB6BAgJEAQ&url=https%3A%2F%2Fdeveloper.apple.com%2Fvideos%2Fplay%2Fwwdc2019%2F415%2F&usg=AOvVaw1WuQfg4izctmtydHfBW-3J) 영상을 보다가 처음 본 함수를 접해서 간단하게 정리해보려 한다. 먼저 영상 속에서 이 함수가 나오게 된 계기를 살펴보자. 영상 속에선 Copy-on-write를 효율적으로 구현하기 위해 사용한다.

그렇다면 어떻게 사용했는지 영상 속 예제 코드를 살펴보자. 

```swift
struct Material {
  public var roughness: CGFloat
  public var shade: CGFloat
  private var _texture: Texture

  var isSparkly: Bool {
    get { return _texture.isSparkly}
    set {
      if !isKnownUniquelyReferenced(&_texture) { _texture = Texture(copying: _texture) }
      _texture.isSparkly = newValue
    }
  }

  init(roughness: CGFloat, shade: CGFloat, texture: Texture) {
    self.roughness = roughness
    self.shade = shade
    self._texture = texture
  }
}
```

Copy-on-write를 간단히 설명하자면 값을 할당해주는 시점, 즉 값이 변경되는 시점에 복사가 일어나는 것을 말한다. 객체를 참조하는 참조 횟수가 1회를 넘지 않도록 유지할 수 있는 방법이기도 하다. 이렇게 되면 값의 변화를 예측하기 쉽고 Reference type을 Value semantics 하게 사용할 수 있게 된다. 

영상을 보고 나면 Value, Reference Type과 Value, Reference Semantics는 구분을 할 수 있다. Type보다는 Semantics에 집중해야 하는데, Semantics는 어떻게 행동하냐를 의미한다. Value Type이라고 Value Semantics로 행동할 것이라고 보장할 순 없다. 

아래의 사진이 그 예다.

![](https://ehdrjsdlzzzz.github.io/2019/07/23/isKnownUniquelyReferenced/diagram.png)

분명 `Material`은 Value 타입이다. 하지만 내부적으로 Reference Type 프로퍼티를 갖고 있다. 그래서 `Material` 객체를 복사하면 그 내부 Reference Type 프로퍼티도 복사되어 두 Value Type 객체가 내부적으로 동일한 Reference Type 객체를 참조하게 된다. 그렇기 때문에 한 곳에서 값을 바꾸게 되면 다른 곳에도 영향을 주게 된다. 

분명 Value Type 객체이지만 Value Semantics 하게 행동하지 않는다.

`isKnownUniquelyReferenced`를 설명하기 위해 너무 돌아왔다. 자세한 내용은 영상을 시청하길 바란다.

다시 예제 코드로 돌아와 Copy-on-write를 효율적으로 사용하기 위해 `isKnownUniquelyReferenced`를 사용한다 했는데, 매번 값이 할당될 때마다 매번 복사해서 할당해주는 것이 아니라 mutable의 위험이 있는 상황에서만 복사만 하는 것이 효율적일 것이다. 

이때 우리는 `isKnownUniquelyReferenced`를 사용해서 강한 참조가 1개만 존재하는지 검사한다. 이 함수는 강한 참조가 1개만 존재할 때 `true`를 반환하고 그렇지 않으면 `false`를 반환한다. 그렇기 때문에 `Texture` 객체는 `Material` 내부 참조 프로퍼티인 `_texture`에 의해서만 참조가 되고 있기 때문에 immutable 하게 관리될 수 있다. 

하지만 이 역시 보장할 수 없다. 그 이유를 공식 문서의 내용을 통해 알아보자.

객체에 대한 약한 참조가 존재해도 `true`를 반환할 수 있다. 약한 참조가 가능하다는 것은 어디에선가 강한 참조를 하고 있다는 것을 의미하는데 약한 참조를 이 함수에 인자로 넘기면 강한 참조가 1개라도 무조건 `false`를 반환한다. 약한 참조로만은 강한 참조가 1개라는 것을 보장할 수 없기 때문이다. 

```swift
class A {
    private weak var b: B?
    
    init(b: B) {
        self.b = b
        print(isKnownUniquelyReferenced(&self.b))) // false
    }
}

class B {
    
}

var bInstance = B()
let aInstance = A(b: bInstance)

print(isKnownUniquelyReferenced(&bInstance)) // true
```

코드를 살펴보면 `B`의 객체는 `bInstance`에 의해서만 강한 참조가 존재한다. 그렇기 때문에 `isKnownUniquelyReferenced(&bInstance)`는 `true`를 반환한다. 하지만 `A` 생성자 안에서 약한 참조 프로퍼티를 이용해 검사를 해보면 항상 `false`가 나오는 것을 확인할 수 있다.

또한 여러 스레드에서 동시에 접근하고 있는 객체를 검사해도 `true`가 나올 수 있다. 그렇기 때문에 다중 스레드에서 접근할 수 있는 객체를 검사할 때는 항상 스레드가 동기화된 환경에서만 검사를 진행해야 한다. 

이제 위에서 `isKnownUniquelyReferenced`를 써도 immutable을 보장할 수 없다는 말을 이해할 수 있을 것이다. 



**참고 자료**

- [isKnownUniquelyReferenced(_:)](https://developer.apple.com/documentation/swift/2429905-isknownuniquelyreferenced)

> 여전히 저에겐 쉽지만은 않은 주제인 것 같습니다. 혹시 틀리거나 부족한 부분이 있다면 댓글로 피드백 부탁드립니다. 😁