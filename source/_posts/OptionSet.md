---
title: OptionSet
date: 2019-03-23 20:12:09
tags: ["Swift", "OptionSet", "enum"]
category: ["Swift"]
---

오늘은 회사 과제를 진행하던 중 처음 접한 스위프트의 `OptionSet`이라는 친구를 살펴보려 한다. 처음 접한 개념이지만 알아보니 그렇게 어렵지 않은 개념이면서도 유용하게 사용할 수 있을 것 같은 개념이어서 이렇게 기록해보려 한다. 

### OptionSet

---

먼저 그 개념을 공식 문서를 통해 살펴보도록 하자. 

`OptionSet` 프로토콜은 비트들 각각이 집합의 요소를 표현하는 비트 집합 타입을 표현하는데 사용된다. 이 프로토콜을 채택한 사용자 정의 타입에선 요소 검사(해당 요소가 집합에 속하는지), 합집합, 교집합 연산과 같은 집한 연산들을 수행할 수 있다. 

옵션 집합(`OptionSet` 프로토콜을 채택한 사용자 정의 타입)을 만들기 위해선 타입 선언 부분에 `rawValue`를 포함시켜야 한다. 사용자 정의 타입으로 만든 옵션 집합이 기본 집합 연산을 수행하기 위해선 `rawValue` 프로퍼티는 반드시 `FixedWidthInteger` 프로토콜을 따르고 있는 타입(`Int`, `UInt8` 등등)이어야 한다. 다음으로는 정적(*static*) 변수로 고유한 2의 거듭제곱 값(1, 2, 4, 8, …)을 `rawValue`로 갖는 옵션들을 생성한다. 이렇게 2의 거듭제곱 값을 `rawValue`로 가져야 각각의 옵션들은 단일 비트로 표현이 가능하기 때문이다.

`OptionSet`은 다음과 같이 정의할 수 있다. 

```swift
struct ShippingOptions: OptionSet {
    let rawValue: Int

    static let nextDay    = ShippingOptions(rawValue: 1 << 0) // 0001
    static let secondDay  = ShippingOptions(rawValue: 1 << 1) // 0010
    static let priority   = ShippingOptions(rawValue: 1 << 2) // 0100
    static let standard   = ShippingOptions(rawValue: 1 << 3) // 1000

    static let express: ShippingOptions = [.nextDay, .secondDay] // ?? 
    static let all: ShippingOptions = [.express, .priority, .standard] // ??
}
```

먼저 왜 2의 거듭제곱 값으로 표현되어야 할까?

그 이유는 위에서 언급되었듯이 단일 비트로 각각의 값을 표현할 수 있기 때문이다. (Bitmask)

```
0001 // 1
0010 // 2
0100 // 4
1000 // 8
```

만일 2의 거듭제곱이 아니라면 어떻게 될까? 이에 대해서는 밑에서 얘기해보자. 

처음 `OptionSet`을 봤을 때 가장 먼저 든 생각은 *"열거형(enum)이랑 뭐가 다른거지!?"* 였다. 그 생각을 몇몇의 글들을 읽어보면서 정리해보았다.

**열거형과 `OptionSet`을 채택한 타입과의 가장 큰 차이점은 단일 타입 변수가 가질 수 있는 경우의 수의 차이다.** 열거형 타입은 해당 타입의 케이스를 하나만 나타낼 수 있다. 하지만 `OptionSet` 여러 케이스를 하나의 변수로 표현할 수 있다. 열거형에서 이를 표현하려면 여러 케이스에 해당하는 열거형 타입을 배열과 같은 콜렉션 타입으로 표현해야 한다. 

**`OptionSet`은 여러 케이스를 하나의 변수로 담을 수 있기 때문에 각각의 케이스는 유일해야 한다.** 그리고 이를 위해 우리는 비트 값으로 이를 표현한 것이고 2의 거듭제곱으로 표현한 것도 그와 같은 이유다. 실제로 **여러 케이스를 표현한다고 여러 값을 갖고 있을 필요는 없다.** 위의 `ShippingOptions`을 살펴보자. 

`express`는 `[.nextDay, .secondDay]`로 표현된다. 하지만 `[ShippingOptions]`이 아닌 `ShippingOptions` 타입 변수에 할당된다. 2의 거듭제곱으로 표현되고 있다는 걸 상기시켜보면 실제로 `express`의 값은 `.nextDay(0001)`과 `.secondDay(0010)`을 더한 `0011`인 것이다. 그리고 모든 원시 값이 2의 거듭제곱이기 때문에 `0011`만 보아도 `0001`과 `0010`의 조합인 걸 알 수 있다.   

여기서 2의 거듭제곱으로 해야 하는 이유가 나온다. 만일 2의 거듭제곱 값이 아닌 `0011` , 즉 3을 원시 값으로 갖는 케이스가 있다면 `.express`와 구분이 되지 않기 때문이다. 

또한 서버에 값을 전송할 때 리스트 형태의 값을 전달하는 것보다 이렇게 비트 마스크로 표현된 값을 보내는 것이 더 수월할 수 있다. (물론 서버 개발자와의 확실한 상호 협의가 요구되지만)

그리고 위에서 언급했듯이 `OptionSet`을 따르는 프로토콜은 **집합 연산**을 수행할 수 있다.

```swift
let normal: Pet = [.nextDay, .secondDay]
let options2: Pet = [.nextDay, .priority, .standard]
let intersection = options1.intersection(options2)
print(intersection) // ShippingOptions(rawValue: 1) 👉 nextDay

let union = options1.union(options2)
print(union) // ShippingOptions(rawValue: 15) 👉 nextDay, secondDay, priority & standard

let subtracting = options1.subtracting(options2)
print(subtracting) // ShippingOptions(rawValue: 2) 👉 secondDay

let contains = options1.contains(.standard)
print(contains) // false
```

</b>

오늘은 이렇게 간단히 `OptionSet`에 대해 알아보았다. 이를 사용한다면 보다 여러 케이스를 포함하는 상황에서 보다 적은 코드로  각각의 상황에 대응할 수 있을 것 같다.