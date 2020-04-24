---
title: RxSwift (4)
subtitle: Operators - Filtering 
date: 2019-02-03 22:09:26
tags: ["RxSwift", "Operator", "skip", "take", "filter", "distinctUntilChanged"]
category: ["RxSwift"]
---

### Operators 

------

`Operator`는 `Observable`이 내보내는 이벤트들을 변형하고 처리하는데 사용할 수 있다. 이러한 `Operator`에는 다양한 종류가 존재한다. 

- Filtering 
- Transforming 
- Combining 
- Time Based 

이들을 하나씩 살펴보자

> `Operator`들을 보다 쉽게 이해하는 방법은 마블 다이어그램을 보는 것인데 이런 마블 다이어그램을 모아둔 사이트인 [RxMarbles](https://rxmarbles.com)를 참고하면서 살펴보면 이해에 도움이 될 것이다.

### Filtering Operators

------

필터링 연산자들은 조건에 따라 이벤트를 무시하거나 원하는 이벤트들을 받고 싶을 때 사용한다. 그럼 바로 필터링 연산자에는 무엇들이 있는지 하나씩 살펴보자. 필터링 연산자에도 종류가 존재하는데 그 종류는 다음과 같다.

- Ignoring Operator
- Skipping Operator
- Taking Operator
- Distinct Operator

그럼 이들을 하나씩 살펴보자. 

<br>

**Ignoring Operators**

**`ignoreElements`**

`.next` 이벤트는 모두 무시하고 `.completed` 혹은 `.error`와 같이 시퀀스를 종료시키는 이벤트만 받고 싶을 때 사용한다. 

```swift
let strikes = PublishSubject<String>()
let disposeBag = DisposeBag()

strikes
	.ignoreElements()
	.subscribe { _ in 
    	print("You're out!")
	}.disposedd(by: disposeBag)
```

구독 이전에 우리는 `.ignoreElements` 연산자를 사용하여 모든 `.next`는 거르기로 했다. 

```swift
strikes.onNext("X")
strikes.onNext("X")
strikes.onNext("X")
strikes.onCompleted()

// You're out! 
```

`.next` 이벤트는 모두 무시하고 `.onCompleted`만 받는다. 하지만 위의 예제가 야구라고 가정한다면 3번째 스트라이크와 같이 n번째 이벤트를 다뤄야 한다. 이를 위해 우리는 `.elementAt` 연산자를 사용할 수 있다. 

**`elementAt`**

받고 싶은 이벤트의 인덱스를 넣어주면 해당 인덱스의 이벤트만을 받게 된다.  

```
----E----E----E----E---->
    |    |    |    |
------elementAt(1)------
         |
---------E-------------->
```

```swift
let strikes = PublishSubject<String>()
let disposeBag = DisposeBag()

strikes
	.elementAt(1)
	.subscribe { _ in 
    	print("You're out!")
	}.disposedd(by: disposeBag)

strikes.onNext("X")
strikes.onNext("X")
strikes.onNext("X")

// You're out!
```

이렇게 전체 혹은 단일 이벤트를 거르는 연산자말고도 우리에게 익숙한 `filter` 같이 조건에 맞는 이벤트들만을 받을 수 있게하는 연사자인 `filter`가 RxSwift에도 존재한다.

**`filter`**

조건을 담고 있는 클로저를 인자로 받고 조건에 맞는 이벤트만을 밑으로 흘려보낸다.

```swift
let disposeBag = DisposeBag() 

Observable.of(1,2,3,4,5)
	.filter {
    	$0 % 2 == 0
	}.subscribe(onNext: { 
    	print($0)
    }).disposed(by: disposeBag)

// 2
// 4
// 6
```

<br>

**Skipping Operators**

특정 이벤트들은 생략하고 싶을 때 우리는 다음과 같은 연산자들을 사용할 수 있다. 

**`skip`**

인자로 넘어가는 숫자만큼의 이벤트를 생략하고 그 이후 이벤트부터 수신한다.

```
----E----E----E----E---->
    |    |    |    |
--------skip(3)--------
                   |
-------------------E--->
```

```swift
let disposeBag = DisposeBag() 

Observables.of(10,11,12,13,14,15)
	.skip(3)
	.subscribe(onNext: { 
    	print($0)
    }).disposed(by: disposeBag)

// 13
// 14
// 15
```

`filter`에 우리가 조건을 정의할 수 있듯이 우리는 `skip`에서도 이와 유사한 기능을 제공하는 연산자가 존재한다. 

**`skipWhile`**

조건을 정의하고 해당 조건에 맞는 이벤트를 생략하는 연산자로 얼핏보면 `filter`와 크게 다를 것이 없어보이지만 `skipWhile`은 해당 조건에 맞는 최초의 이벤트만을 생략하고 이후에는 조건에 맞아도 이벤트를 흘려보낸다. 

```
----1----2----3----4----5---->
    |    |    |    |    |
---skipWhile { $0 % 2 == 1}---
         |    |    |    |
---------2----3----4----5---->
```

```swift
let disposeBag = DisposeBag()
Observables.of(1,2,3,4,5)
	.skipWhile { $0 % 2 == 1}
	.subscribe(onNext: { 
    	print($0)
    }).disposed(by: disposeBag)

// 2
// 3
// 4
// 5
```

지금까지의 필터링 연산자는 정적(*static*)이었다. 다른 `Observable`에 기반하여 이벤트를 내보내는 동적인 연산자는 무엇이 있을까? 

**`skipUntil`** 

하나의 `Observable`의 이벤트를 무시하다가 다른 `Observable`이 이벤트를 내보내는 것을 기점으로 이벤트를 받는 연산자로 다른 `Observable`에 의해 *trigger* 되기 이전의 이벤트들은 생략된다. 

```
----E----E----E----E----> source
    |    |    |    |
------T------------------> trigger
-------skipUntil---------
         |    |    |
---------E----E----E---->
```

```swift
let disposeBag = DisposeBag() 

let sourceSubject = PublishSubject<String>()
let triggerSubject = PublishSubject<String>() 

sourceSubject
	.skipUntil(triggerSubject)
	.subscribe(onNext: { 
    	print($0)
    }).disposed(by: disposeBag)
```

`sourceSubject`를 구독해도 `triggerSubject`가 이벤트를 흘려보내지 않는다면 이벤트를 수신할 수 없다.  

```Swift
sourceSubject.onNext("A")
sourceSubject.onNext("B")

triggerSubject.onNext("Trigger")

sourceSubject.onNext("C")

// C
```

<br>

**Taking Operators**

이는 생략 연산자의 반대로 수신받고 싶은 이벤트를 명시해주는 연산자이다. 

**`take`**

시작부터 원하는 갯수의 이벤트를 명시한다면 해당 갯수에 해당하는 이벤트만을 수신하고 이후는 무시하게 된다.

```swift
let disposeBag = DisposeBag()

Observable.of(1,2,3,4,5)
	.take(3)
	.subscribe(onNext: { 
        print($0)
    }).disposed(by: disposeBag)

// 1
// 2
// 3
```

**`takeWhile`**

`skipWhile`처럼 `takeWhile`도 존재한다. 또한 발생하는 이벤트와 함께 해당 이벤트의 순서 인덱스도 받고싶다면 `enumerated` 연산자를 사용할 수 있다. 

```swift
let disposeBag = DisposeBag()

Observable.of(2,2,4,4,6,6)
	.enumerated()
	.takeWhile { index, integer in 
    	integer % 2 == 0 && index < 3
	}.map { $0.element }
	.subscribe(onNext: {
    	print($0)	
	}).disposed(by: disposeBag)
```

이벤트를 해당 이벤트의 순서 인덱스와 함께 받고 조건에 해당하는 이벤트만을 취한다. 그리고 `map` 연산자를 통해 튜플에서 `element`에 해당하는 값만을 밑으로 흘려보낸다. 

**`takeUntil`**

역시 `skipUntil`과 유사하게 `takeUntil`도 존재한다. 이는 *trigger* `Observable`이 이벤트를 내보내기 전까지의 이벤트만을 수신하고 그 이후의 이벤트는 무시한다. 

```swift
let disposeBag = DisposeBag()

let sourceSubject = PublishSubject<String>() 
let triggerSubject = PublishSubject<String>() 

sourceSubject
	.takeUntil(triggerSubject)
	.subscribe(onNext: { 
    	print($0)
    }).disposed(by: disposeBag)

sourceSubject.onNext("1")
sourceSubject.onNext("1")

triggerSubject.onNext("X")

sourceSubject.onNext("3")

// 1
// 2
```

`takeUntil`은 `DisposeBag`을 사용하는 대신하여 구독을 처리하는데 사용되기도 한다. RxCocoa에서 보다 적은 양의 코드로 구독의 메모리 누수를 보다 간편하게 관리할 수 있다. 

```swift
someObservable
	.takeUntil(self.rx.deallocated)
	.subscribe(onNext: { 
    	print($0)
    })
```

위와 같은 코드에서 `self`는 뷰 컨트롤러나 뷰 모델이 될 수 있고 이들이 메모리에서 사라질 때 까지 이벤트를 받겠다는 것인데 이들이 사라지면 이들에 속한 `Observable`과 이에 대한 구독도 같이 사라지기 때문에 메모리 누수에 대한 걱정을 줄일 수 있다. 

<br>

**Distinct Operators**

지금부터 살펴볼 연산자는 연속된 중복 이벤트를 예방할 수 있는 연산자이다.

**`distinctUntilChanged`**

이 연산자는 이전 이벤트와 비교했을 때 다른 이벤트만을 흘려보낸다. 

```
----1----2----2----1---->
    |    |    |    |
--distinctUntilChanged--
    |    |         |
----1----2---------1---->

```

```swift
let disposeBag = DisposeBag() 

Observable.of("A", "A", "B", "B", "A")
	.distinctUntilChanged()
	.subscribe(onNext: { 
    	print($0)
    }).disposed(by: disposeBag)

// A
// B
// A
```

문자열(String) 타입은 기본적으로 `Equatable` 프로토콜을 따르고 있기 때문에 같은 이벤트인지에 대한 여부를 판단할 수 있다. 만일 사용자 정의 타입이거나 하나의 객체라면 동일한 경우에 대한 조건을 명시해주어 이 연산자를 사용할 수 있다. 

```swift
Observable.of(객체1(1), 객체2(1), 객체3(2))
	.distinctUntilChanged { $0.value == $1.value }
	.subscribe(onNext: { 
    	print($0)
    }).disposed(by: disposeBag)


// 객체1
// 객체3
```

