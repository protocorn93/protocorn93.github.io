---
title: Strings and Substrings in Swift
subtitle: How Strings and Substrings work in Swift
date: 2019-01-12 20:53:42
tags: [String", "Substring", "String.Index"]
category: ["번역", "Swift"]
---

## [번역] How Strings and Substrings work in Swift

스위프트에선 문자열을 다루는 방법이 다른 프로그래밍 언어들과는 사뭇 다르다. 티스토리 블로그를 운영할 때도 이와 연관된 내용을 다룬 적이 있으나 이번에 한 번 더 다뤄보고자 쉽게 설명된 블로그 글을 번역하여 기록하고자 한다.

[How Strings and Substrings work in Swift](https://medium.com/@studymongolian/how-strings-and-substrings-work-in-swift-fd4dc43ee91d?fbclid=IwAR3pUPic5LYG6dSlhBLhNYpKmauA2H1F0rcY8CEQvlED3sK8eJ3k_QxclS4)

------

이 포스팅에서 사용되는 모든 예제는 다음과 같다. 

```swift
var str = "Hello, Playground"
```



### Strings and substrings

스위프트의 문자열은 여러 버전들을 거쳐 변해왔다. 스위프트4에선 문자열의 부분 문자열을 가져오게 되면 그 부분 문자열의 타입은 `String`이 아닌  `Substring` 타입이다. 왜 그럴까? 문자열은 스위프트에서 값 타입이다. 그 말인즉슨 하나의 문자열을 사용해 다른 문자열을 만들면 이는 복사된 것이다. 이 방법은 안전성은 좋지만(다른 변수에 할당하면 값의 복사가 일어나기 때문에 값의 변경에 있어서 보다 안전하다.) 효율은 떨어지는 방법이다. 

반면에 부분 문자열은 자신의 원본 문자열에 대한 참조이다. 복사가 필요하지 않기 때문에 사용하는데 보다 더 효율적이다. 하지만 100만 개 문자로 이루어진 문자열의 10개의 문자로 이루어진 부분 문자열을 갖고 있다고 가정하자. 부분 문자열은 문자열에 대한 참조이기 때문에 부분 문자열을 다루는 데 있어 매우 긴 원본 문자열에 대한 참조를 유지하고 있어야 한다. 그러므로 언제든지 부분 문자열을 다룰 때는 이를 문자열로 변환해야 한다. 

> 이 부분은 스위프트 공식 문서에서도 언급되고 있다. 
>
> *Substrings aren’t suitable for long-term storage—because they reuse the storage of the original string, the entire original string must be kept in memory as long as any of its substrings are being used.* - [Swift Docs](https://docs.swift.org/swift-book/LanguageGuide/StringsAndCharacters.html)

<img src="https://docs.swift.org/swift-book/_images/stringSubstring_2x.png">

```swift
let myString = String(mySubstring)
```

이는 부분 문자열을 복사할 것이고 이전의 문자열의 메모리는 회수될 수 있다. 부분 문자열(타입으로써)은 수명이 짧아야 한다.

스위프트4에서의 또 다른 개선점은 문자열들은 콜렉션(Collectiotion)이라는 것이다. 이것은 콜렉션에 할 수 있는 모든 작업들은 문자열에서도 가능하다는 것이다. (서브스크립트, 문자들을 거치는 순환, 필터 등등)



### String,Index

부분 문자열을 더 살펴보기 전에 문자열을 이루는 문자에 대한 문자열의 인덱싱이 어떻게 이루어지는지에 대해 이해하는 것이 도움이 될 것이다. 

> 원하는 부분 문자열을 가져오기 위해 범위를 정할 수 있는 인덱스를 가져오는 과정이다. 인덱스를 가져오는 과정은 문자열로부터 부분 문자열을 가져오는 과정 전에 선행되어야 한다.

**startIndex and endIndex**

- `startIndex`는 첫 번째 인덱스의 문자이다.
- `endIndex`는 마지막 문자의 **다음** 인덱스이다.

```swift
var str = "Hello, playground"

// character
str[str.startIndex] // H
str[str.endIndex]   // error: after last character

// range
let range = str.startIndex..<str.endIndex
str[range]  // "Hello, playground"
```

스위프트4의 [one-sided ranges](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/BasicOperators.html)로 범위는 아래와 같이 간단한 형태로 표현할 수 있다.

```swift
let range1 = str.startIndex...
let range2 = ..<str.endIndex
// 모두 문자열 전체의 범위를 의미
```

본인(원작자)은 예제에서 명확성의 목적을 위해 완벽한 형태(`str.startIndex..<str.endIndex`)를 사용할 것이지만 가독성의 목적으로 당신은 아마 one-sided ranges를 사용하길 원할 것이다. 



**after**

`index(after: String,Index)`

- `after`은 주어진 인덱스의 바로 다음 인덱스에 해당하는 문자를 의미한다. 

```swift
// character
let index = str.index(after: str.startIndex)
str[index]  // "e"
// range
let range = str.index(after: str.startIndex)..<str.endIndex
str[range]  // "ello, playground"
```



**before**

`index(before: String,Index)`

- `before`은 주어진 인덱스의 바로 이전 인덱스에 해당하는 문자를 의미한다. 

```swift
// character
let index = str.index(before: str.endIndex)
str[index]  // d
// range
let range = str.startIndex..<str.index(before: str.endIndex)
str[range]  // Hello, playgroun
```



**offsetBy**

`index(String.Index, offsetBy: String,IndexDistance)`

- `offsetBy` 값은 양수 혹은 음수가 될 수 있고 주어진 인덱스부터 시작한다. 비록 타입은 `String,IndexDistance`이지만 단순히 `Int` 타입의 값을 넣어주어도 무방하다. 

```swift
// character
let index = str.index(str.startIndex, offsetBy: 7)
str[index]  // p
// range
let start = str.index(str.startIndex, offsetBy: 7)
let end = str.index(str.endIndex, offsetBy: -6)
let range = start..<end
str[range]  // play
```

> 시작 인덱스를 0으로 하여 올라가고 내려간다. 첫 번째 예제 `str.index(str.startIndex, offsetBy: 7)`에서 `H`을 0으로 하여 7까지 올라가면 `p`인 것을 확인할 수 있다. 또한 `str.index(str.endIndex, offetBy: -6)`에서 역시 마지막 인덱스(마지막 문자열의 다음 칸)를 0으로 하여 내려가면 그 값은 `g`가 된다. `start..<end`이기 때문에 `str[range]`의 결과는 `play`가 된다.



**limitedBy**

`index(String.Index, offsetBy: String,IndexDistance, limitedBy: String.Index)`

- `limitedBy`는 오프셋이 인덱스의 범위를 벗어나는 것을 막는데 유용하다. 오프셋은 버위를 벗어날 수 있음에도 불구하고 이 메소드를 사용하면 옵셔널을 반환해준다. 인덱스가 범위를 벗어나면 `nil`을 반환한다.



```swift
// character
if let index = str.index(str.startIndex, offsetBy: 7, limitedBy: str.endIndex) {
    str[index]  // p
}
```

만일 오프셋이 7이 아니라 77이라면 조건문의 안으로 들어가지 않을 것이다.



### Getting substrings

우리는 서브스크립트나 몇 개의 메소드(`prefix`, `suffix`, `split`)들을 통해 문자열로부터 부분 문자열을 가져올 수 있다. 여전히 `String.Index`를 사용할 필요가 있지만 범위 설정을 위해 `Int`는 사용하지 않아도 된다.

> 본격적으로 들어가기 전에 `prefix`와 `suffix`의 사전적 의미를 살펴보자. 각각은 접두사, 접미사란 뜻으로 **단어 앞뒤에 붙어 하나의 다른 언어를 이루는 말**을 의미한다. `prefix`는 인덱스를 기준으로 앞의 범위를 `suffix`는 인덱스를 기준으로 뒤의 범위의 부분 문자열을 가져오는데 사용된다.

**Beginning of a string**

서브스크립트를 사용할 수 있다.

```swift
let index = str.index(str.startIndex, offsetBy: 5)
ley mySubstring = str[..<index] // Hello
```

혹은 `prefix`를 사용할 수도 있고

```swift
let index = str.index(str.startIndex, offsetBy: 5)
let mySubstring = str.prefix(upto: index) // Hello
```

이를 더 쉽게 사용하면 다음과 같다. 

```swift
let mySubstring = str.prefix(5) // Hello
```



**End of string**

역시 서브스크립트를 사용할 수 있고

```swift
let index = str.index(str.endIndex, offsetBy: -10)
let mySubstring = str[index...] // playground
```

`suffix` 사용도 사용되며

```swift
let index = str.index(str.endIndex, offsetBy: -10)
let mySubstring = str.suffix(from: index) // playground
```

더욱 간단하게 표현하자면 다음과 같다.

```swift
let mySubstring = str.suffix(10) // playground
```

`suffix(from: index)`를 사용할 때는 마지막 끝에서부터 거꾸로 값을 계산해야 했다. 하지만 `suffix(x)`를 사용할 때는 인덱스를 고려하여 계산할 필요가 없고 단순히 뒤에서부터 `x`개의 문자를 가져오기 때문에 보다 쉽게 사용할 수 있다. 



**Range in string**

역시 여기서도 우리는 서브스크립트를 사용할 수 있다. 

```swift
let start = str.index(str.startIndex, offsetBy: 7)
let end = str.index(str.endIndex, offsetBy: -6)
let range = start..<end
let mySubstring = str[range]  // play
```



**Converting Substring to String**

부분 문자열을 저장할 준비가 되면 이전 문자열의 메모리가 회수될 수 있도록 `String`으로 변환하는 것을 잊지 말자

```swift
let myString = String(mySubstring)
```



### Using an Int index extension 

`Int` 기반 인덱스를 사용하여 문자열을 다루는 것이 훨씬 쉬울 것이다. 그리고 실제로 [`Int` based extension](https://stackoverflow.com/a/39677704/3681880)을 사용하여 문자열 인덱싱의 복잡성을 숨길 수 있다 (쉽게 사용할 수 있고). 하지만 본인(원작자)는 *Airspeed Velocity and Ole Begemann*의 [Strings in Swift3](https://oleb.net/blog/2016/08/swift-3-strings/) 글을 읽은 후 이를 주저하게 되었다. (*`Int` 를 사용하여 문자열 인덱싱을 쉽게 해주는 extension의 사용*) 또한 스위프트 팀 역시 `Int` 인덱스를 일부러 사용하지 않았다. 여전히 `String.Index`를 사용한다. 그 이유는 스위프트에서 문자들은 모두 같은 길이를 같지 않기 때문이다. 단일 스위프트 문자는 하나, 둘 심지어 그 이상의 유니코드로 이루어 져있을 수 있다. 그러므로 각 고유의 문자열은 자신의 문자들의 인덱스들을 계산해야 한다. 

본인(원작자)는 언젠가 스위프트 팀이 `String.Index`를 추상화할 수 있는 방법을 찾길 희망한다. 하지만 그전까지는 본인은 그들의 API를 사용할 것이다. 이는 문자열 조작은 단순한 `Int` 인덱스 조회가 아니라는 것을 기억하게 해주기 때문이다.

------

### 마무리

실제로 나도 스위프트를 사용하면서 부분 문자열을 다루는데 여간 귀찮고 복잡하다고 여긴 적이 한두 번이 아니다. 그 생각은 이 글을 읽은 후에도 변함이 없다. 실제로 불편한 건 사실이기 때문이다. 하지만 매번 부분 문자열을 가져오기 위해 인덱싱을 구글링하는 수고는 이제 필요 없을 것 같다. 그리고 부끄럽게도 부분 문자열은 원본 문자열에 대한 참조를 유지하고 있다는 사실은 이 글을 통해 처음 알게 되었다. 앞으로는 이 부분을 유념하여 보다 효율적으로 코드를 작성할 수 있는 계기가 될 수 있을 것 같다.