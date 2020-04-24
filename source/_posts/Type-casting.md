---
title: Type casting
date: 2019-04-14 15:34:27
tags: ["Swift", "is", "as?", "as!"]
category: ["번역"]
---

오늘은 스위프트 공식 문서에 나와있는 타입 캐스팅의 내용을 번역하면서 약간의 나의 생각과 의견을 덧붙여보았다.

## Type Casting

타입 캐스팅(Type casting)이란 인스턴스의 타입을 검사하는 방법이고, 자신의 클래스 계층 안에서 인스턴스를 부모 클래스나 자식 클래스로 다루는 방법이다. 

스위프트에서 타입 캐스팅은 `is`와 `as` 연산자로 구현되어 있다. 이 두 연산자는 값의 타입을 검사하거나 다른 타입으로 변경하는데 간단하고 이해하기 쉬운 방법으로 제공한다. 

> 여기서 이해하기 쉬운 방법이란 `expressive way`로 타입 검사에는 `is`, 타입 캐스팅에는 `as`라는 영단어를 사용함으로써 좀 더 문장을 읽듯이 코드를 받아들일 수 있다는 의미이다. 
>
> `is` the Student is Person?
>
> Person `as` Student have to go school

또한 타입 캐스팅을 활용해 타입이 프로토콜을 채택하고 있는지 역시 검사할 수 있다. 



### 타입 캐스팅을 위한 클래스 계층 정의

------

클래스 및 하위 클래스의 계층 구조와 함께 타입 캐스팅을 사용하여 특정 클래스 인스턴스의 타입을 검사하고 동일한 계층 내의 다른 클래스에 해당 인스턴스를 캐스팅할 수 있다. 아래에 정의된 세 개의 코드는 클래스의 계층 구조를 정의하고 배열은 타입 캐스팅의 예제에 활용하기 위해 해당 클래스의 인스턴스들을 담고 있다. 

첫 번째 코드는 최상위 클래스인 `MediaItem`을 정의하고 있다. 이 클래스는 디지털 미디어 라이브러리에서 보여질 수 있는 기본적인 기능들을 제공한다. 구체적으로 말하자면 `String` 타입의 `name` 프로퍼티와 `name`을 초기화하는 생성자를 선언하고 있다. (모든 영화와 노래를 포함한 미디어 아이템은 이름을 갖고 있을 것이라는 가정을 하고 있다.)

```swift
class MediaItem { 
  var name: String 
  init(name: String) {
    self.name = name
  }
}
```

다음 코드는 `MediaItem`의 하위 클래스들을 정의하고 있다. 첫 번째 하위 클래스인 `Movie` 클래스는 영화나 필름에 관한 추가 정보를 감싸고 있다. 이 클래스는 최상위 클래스 `MediaItem`에 `director` 프로퍼티를 더했고 그에 해당하는 생성자를 추가하였다. 두 번째 하위 클래스인 `Song` 클래스는 `artist` 프로퍼티와 그에 해당하는 생성자를 추가하였다. 

```swift
class Movie: MediaItem {
  var director: String
  init(name: String, director: String) {
    self.director = director
    super.init(name: name)
  }
}

class Song: MediaItem {
  var artist: String
  init(name: String, artist: String) {
    self.artist = artist
    super.init(name: name)
  }
}
```

마지막 코드는 `library`라는 배열을 선언했고 이는 두 개의 `Movie`와 세 개의 `Song` 클래스 인스턴스를 담고 있다. 이 배열의 타입은 배열이 담고 있는 요소들에 의해 정해진다. 스위프트 타입 검사기는 `Movie`와 `Song`으로부터 공통 부모 클래스인 `MediaItem` 클래스를 추론할 수 있고 배열 `library`을 `[MediaItem]` 타입으로 추론한다. 

```swift
let library = [
  Movie(name: "Casablanca", director: "Michael Curtiz"),
  Song(name: "Blue Suede Shoes", artist: "Elvis Presley"),
  Movie(name: "Citizen Kane", director: "Orson Welles"),
  Song(name: "The One And Only", artist: "Chesney Hawkes"),
  Song(name: "Never Gonna Give You Up", artist: "Rick Astley")
]
// the type of "library" is inferred to be [MediaItem]
```



### 타입 검사

------

타입 검사 연산자인 `is`를 사용해 인스턴스가 특정 하위 클래스 타입인지 아닌지 검사한다. 인스턴스가 특정 타입이 맞다면 `true`를 반환하고 아니라면 `false`를 반환한다. 

아래의 두 변수인 `movieCount`와 `songCount`는 `library` 배열 안의 인스턴스들 중 `Movie` 클래스와 `Song` 클래스 타입 인스턴스의 개수를 의미한다.  

```swift
var movieCount = 0
var songCount = 0

for item in library {
  if item is Movie {
    movieCount += 1
  } else if item is Song {
    songCount += 1
  }
}

print("Media library contains \(movieCount) movies and \(songCount) songs")
```

이 예제는 `library` 배열을 탐색하면서 `MediaItem` 클래스 인스턴스를 검사한다. `item`이 `Movie` 클래스인지 검사하고 맞는다면 `movieCount` 값을 증가시킨다. `songCount` 역시 위와 같은 방식으로 증가된다. 



### 다운캐스팅

------

특정 클래스 타입의 상수나 변수가 실제론 하위 클래스를 참조하고 있을 수 있다. 이러한 사실을 믿고 있다면 타입 캐스트 연산자인 `as?` 혹은 `as!`를 통해 하위 클래스 타입으로 다운 캐스팅을 시도해볼 수 있다. 

다운 캐스팅은 실패할 수 있기 때문에 타입 캐스트 연산자는 서로 다른 두 가지 형태를 갖는다. 조건문 형태의 `as?`는 다운 캐스팅하려는 값의 옵셔널 타입을 반환한다. 강제 형태인 `as!`는 한 번에 다운 캐스팅을 진행하고 강제로 값을 벗겨 하여 반환한다. 

다운 캐스팅이 성공할지 확실하지 않을 경우에는 조건문 형태의 다운 캐스트 연산자인 `as?`를 사용하자. 이는 항상 옵셔널 타입의 값을 반환할 것이고 만일 다운 캐스팅이 불가능하다면 `nil`을 반환할 것이다. 이는 성공적인 다운 캐스팅 검사를 가능케 한다.

강제 다운 캐스팅 연산자인 `as!`는 항상 성공할 것이라는 보장이 있는 곳에서만 사용하자. 

> 개발자가 모든 상황을 기억하고 있을 순 없다. 특히 다른 사람이 작성한 코드는 더더욱 그렇다. 그러니 웬만하면 `as?`를 사용하자.

만일 부정확한 타입으로의 다운 캐스팅을 진행한다면 런타임 에러를 발생시킬 것이다. 

다음의 예제는 `library` 배열의 `MediaItem`을 탐색하면서 각 아이템에 맞는 설명을 출력해주고 있다. 이를 위해 `MediaItem`이 아니라 실제 타입인 `Movie`나 `Song`에 접근해야 한다. 설명에서 사용될  `Movie` 혹은 `Song`의 `director` 혹은 `artist` 프로퍼티를 사용하기 위해선 이에 대한 접근이 필요하다. 

예제에서 배열 안의 각 아이템은 `Movie` 혹은 `Song` 일 것으로 보인다. 하지만 각 아이템이 실제 어떤 클래스인지는 확실히 알 수 없으므로 `as?` 연산자를 통해 동일한 계층 내의 다른 클래스에 해당 인스턴스를 캐스팅할 수 있다. 

> 즉 `MediaItem`의 하위 클래스들인 것 같은데 해당 인스턴스들이 정확히 어떤 클래스인지 알 수 없기 때문에 조건형 다운 캐스팅 연산자인 `as?`를 통해 캐스팅을 진행한다는 것이다. 

```swift
for item in library {
  if let movie = item as? Movie {
    print("Movie: \(movie.name), dir. \(movie.director)")
  } else if let song = item as? Song {
    print("Song: \(song.name), by \(song.artist)")
  }
}

// Movie: Casablanca, dir. Michael Curtiz
// Song: Blue Suede Shoes, by Elvis Presley
// Movie: Citizen Kane, dir. Orson Welles
// Song: The One And Only, by Chesney Hawkes
// Song: Never Gonna Give You Up, by Rick Astley
```

위의 에제는 현재 검사하는 아이템을 먼저 `Movie` 타입으로의 다운 캐스팅을 시도한다. `item`은 `MediaItem` 타입이기 때문에 이는 `Movie`가 될 수도 있고 `Song`이 될 수도 있다. 혹은 단순히 `MediaItem`일 수도 있다. 이러한 불확실성 때문에 `as?` 연산자는 하위 클래스로의 다운 캐스팅을 진행할 때 옵셔널 값을 반환한다. `item as? Movie`의 결과는 `Movie?`가 된다. 

`library` 배열 안의 `Song` 인스턴스를 `Movie` 클래스로 다운 캐스팅을 시도한다면 이는 실패할 것이다. 이를 적절히 대응하기 위해 위의 에제에선 옵셔널 바인딩을 사용해 `Movie?` 타입이 실제 값을 포함하고 있는지를 검사한다. 옵셔널 바인딩은 `if let movie = item as? Movie`로 작성되어 있고 이는 다음과 같이 읽을 수 있다

"`item`을 `Movie`로써 접근을 시도하라. 만일 성공한다면 `movie`라 불리는 새 임시 상수에 옵셔널 `Movie`에 저장된 값을 할당해라."

다운 캐스팅에 성공한다면 `director  `를 포함한 `movie`의 프로퍼티들은 `Movie` 인스턴스를 설명을 출력하는데 사용된다. 

> 캐스팅은 실제로 인스턴스를 수정하거나 값들을 변경하지 않는다. 기본적인 인스턴스는 그대로 남아있는다. 이는 단순히 캐스팅된 타입의 인스턴스로 취급되고 접근된다. 



### Any 그리고 AnyObject에 대한 타입 캐스팅

------

스위프트는 nonspecific 타입을 사용하기 위해 두 가지 특별한 타입을 제공한다. 

> nonspecific이란 "특이하지 않는" 이라는 의미를 갖고 있고 사용하려는 객체가 정확히 어떤 타입인지 모르는 경우를 말한다. 
>
> 하지만 개인적으로 정말 필수불가결한 상황이 아니라면 이러한 경우는 지양해야 한다고 생각한다. 특히 다른 사람이 작성한 코드에서 이러한 타입을 본다면 추측하기 여간 까다로울 수 없다. 

제공되는 동작과 기능이 명시적으로 필요한 경우에만 `Any`와 `AnyObject`를 사용하자. 언제든 명확한 타입을 명시하는 것이 바람직하다. 

다음은 `Any`를 사용해서 서로 다른 여러 타입들을 섞어서 사용하는 예제이다. 타입들에는 함수 타입이나 클래스가 아닌 타입도 포함된다. 예제 코드는 `things`라는 배열을 생성하고 `Any` 타입의 값들을 담을 수 있다. 

```swift
var things = [Any]()

things.append(0)
things.append(0.0)
things.append(42)
things.append(3.14159)
things.append("hello")
things.append((3.0, 5.0))
things.append(Movie(name: "Ghostbusters", director: "Ivan Reitman"))
things.append({ (name: String) -> String in "Hello, \(name)" })
```

`things` 배열은 두 개의 `Int` 타입의 값, 두 개의 `Double` 타입, 한 개의 `String` 타입, 한 개의 `(Double, Double)` 튜플 타입, `Movie` 타입, `String`을 받아서 `String` 타입의 값을 반환하는 클로져 타입을 담고 있다. 

이렇게 `Any`나 `AnyObject`로 알려진 변수나 상수에 대해서는 `as` 혹은 `is` 패턴을 사용할 수 있다. 아래의 예제에선 `things` 안의 아이템들을 순회하면서 `switch` 문을 통해 타입을 검사하고 캐스팅한다. 

```swift
for thing in things {
    switch thing {
    case 0 as Int:
        print("zero as an Int")
    case 0 as Double:
        print("zero as a Double")
    case let someInt as Int:
        print("an integer value of \(someInt)")
    case let someDouble as Double where someDouble > 0:
        print("a positive double value of \(someDouble)")
    case is Double:
        print("some other double value that I don't want to print")
    case let someString as String:
        print("a string value of \"\(someString)\"")
    case let (x, y) as (Double, Double):
        print("an (x, y) point at \(x), \(y)")
    case let movie as Movie:
        print("a movie called \(movie.name), dir. \(movie.director)")
    case let stringConverter as (String) -> String:
        print(stringConverter("Michael"))
    default:
        print("something else")
    }
}

// zero as an Int
// zero as a Double
// an integer value of 42
// a positive double value of 3.14159
// a string value of "hello"
// an (x, y) point at 3.0, 5.0
// a movie called Ghostbusters, dir. Ivan Reitman
// Hello, Michael
```

> `Any` 타입은 옵셔널을 포함해 어떠한 타입의 값도 표현한다. `Any` 타입을 기대하는 값에 옵셔널 값을 사용한다면 스위프트는 경고를 줄 것이다. 정말 옵셔널 타입의 값을 `Any` 타입의 값에 사용하고 싶다면 아래와 같이 업 캐스팅 연산자  `as`를 사용해 `Any`로 캐스팅해야 한다.
>
> ```swift
> let optionalNumber: Int? = 3
> things.append(optionalNumber)        // Warning
> things.append(optionalNumber as Any) // No warning
> ```