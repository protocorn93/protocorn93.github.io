---
title: RxSwift (3)
subtitle: Subjects
date: 2019-01-23 14:11:26
tags: ["RxSwift", "Subject", "PublishSubject", "BehaviorSubject", "ReplaySubject", "Variable"]
category: ["RxSwift"]
---

### Subjects

------

우리는 위에서 RxSwift의 핵심인 `Observable`에 대해 공부를 하였다. 하지만 앱을 개발하면서 공통적으로 필요한 부분은 런타임 중  `Observable`에 값을 수동으로 넣어주고 이는 구독자에게 내보내야 한다는 것이다. 그렇기 때문에 아마 `Observable`인 동시에 그 이벤트 시퀀스에 관찰자인 것을 원하게 될 것이다. 그리고 그것을 우리는 **Subject**라 부른다. 

이들은 이벤트를 받고 바로 그것을 구독자(subscriber)에게 전달한다. 

RxSwift엔 4 가지 유형의 `Subject`가 존재한다. 

- `PublishSubject` 
  - 빈 것으로 시작하고 새로 들어온 이벤트만을 구독자에게 내보낸다. 
  - 구독하기 이전에 발생한 이벤트는 새 구독자에게 전달되지 않는다. 
- `BehaviorSubject`
  - 초기 값이 있는 상태로 시작하고 초기 값을 전달하거나 최신 값을 전달한다. 
  - 최신 값이라하면 구독 하기 전의 최신 값도 해당된다. 
- `ReplaySubject`
  - 버퍼 크기와 함께 초기화되고 구독 시점에서 최신 이벤트로부터 해당 버퍼 크기만큼의 이전 이벤트들을 구독자에게 함께 전달한다. 
- `Variable`
  - `BehaviorSubject`를 감싼것으로 현재 값을 상태로써 보존하고 최신 값이나 초기 값만을 구독자에게 전달한다. 



**PublishSubject**

------

`PublishSubject`는 구독자가 **구독 시점부터** 구독을 끊거나, `.error` 혹은 `.completed` 이벤트를 받아 `Subject`가 종료될 때까지 새로운 이벤트를 받아볼 수 있게끔 하고 싶을 때 유용하다. 

다음의 마블 다이어그램에서 최상단 라인은 `PublishSubject`, 두 번째, 세 번째 라인은 각각 구독자들을 의미한다. 위로 향하는 화살표는 구독 시점을 의미하고 아래로 향하는 내보내어진 이벤트를 의미한다.

![](http://ehdrjsdlzzzz.github.io/2019/01/23/RxSwift-3/publishsubjectmarble.png)

첫 번째 구독자는 이벤트 1이 발생한 후 구독하였기 때문에 이벤트 1은 받지 못하고 2와 3을 받는다. 마찬가지로 두 번째 구독자 역시 이벤트 2가 발생한 후 구독하였기 때문에 이벤트 3만 받을 수 있다. 다음의 코드를 살펴보자. 

```swift
// 빈 PublishSubject를 생성
let subject = PublishSubject<String>()
// 구독자가 없기 때문에 아무도 이 이벤트를 받지 못한다. 
subject.onNext("Is anyone listening?")

// 첫 번째 구독자
let subscriptionOne = subject.subscribe(onNext: { string in
	print("1)", string)
})

// 구독자가 존재하므로 "1"은 출력이 된다.
subject.on(.next("1"))
// onNext() 메소드로도 이벤트를 수신할 수 있다.
subject.onNext("2")

// 두 번째 구독자.
let subscriptionTwo = subject.subscribe({ event in
	print("2)", event.element ?? event)
})

// 두 번째 구독자는 3부터 받을 수 있다.
subject.onNext("3")

// 첫 번째 구독자가 구독을 취소하였다. 
subscriptionOne.dispose()

// 두 번째 구독자만이 4를 받을 수 있다. 
subject.onNext("4")

// Subject가 .onCompleted 이벤트를 수신하고 종료되었다. 
subject.onCompleted()

// 불가능
subject.onNext("5")

// 두 번째 구독자가 구독을 취소하였다.
subscriptionTwo.dispose()

let disposeBag = DisposeBag()

// 세 번째 구독자가 구독을 하여도 이미 .onCompleted 이벤트를 받고 종료되었다. 
// 종료된 상태에서는 종료를 야기시킨 이벤트만을 수신한다. 여기서 세 번째 이벤트는 .onCompleted만을 수신받는다. 
subject.subscribe {
    print("3)", $0.element ?? $0)
}.disposed(by: disposeBag)

// 다른 값을 전달하려 해도 세 번째 구독자는 .onCompleted만 받는다. 
subject.onNext("?")

// Output
// 1) 1
// 1) 2
// 1) 3
// 2) 3
// 2) 4
// 2) completed
// 3) completed
```

종료된 `Subject`를 구독하는 구독자는 종료를 야기한 이벤트만을 받게 된다.

실제로 모든 `Subject` 유형은 한 번 종료되면 그 이후의 구독자들에게 종료를 야기한 이벤트(`.onCompleted`, `.onError`)를 재전송한다. 그렇기 때문에 이러한 이벤트들에 대한 핸드럴를 포함시키는 것은 좋은 방법이다. `Subject`가 종료될 때 알람을 받는 것 뿐만 아니라 종료된 `Subject`에 대해 구독을 할 때도 알람을 받을 수 있기 때문이다. 

`PublishSubject`는 경매 어플과 같은 시간에 민감한 데이터의 모델링이 필요한 어플에 사용할 수 있다. 오전 10시 1분에 가입한 사용자에게 오전 9시 59분에 경매에 1분밖에 남지 않았다고 경고하는 것은 이치에 맞지 않을 것이다.

하지만 가끔은 새로운 구독자에게 최신 이벤트가 무엇인지 알려주고 싶을 수 있다. 그 최신 이벤트가 구독 전에 발생한 것일지라도 말이다. 이 역시 선택 사항이 존재한다. 



**BahaviorSubject**

------

`BehaviorSubject`는 최신 `.next` 이벤트를 새로운 구독자에게 다시 한번 전달한다는 것을 제외하곤  `PublishSubject`와 비슷하게 동작한다. 다음의 마블 다이어그램을 살펴보자. 

![](http://ehdrjsdlzzzz.github.io/2019/01/23/RxSwift-3/behaviorsubjectmarble.png)

최상단의 라인이 `Subject`이다. 두 번째 라인의 구독자는 이벤트 1이후 , 2 이전에 구독을 시작했다. `BehaviorSubject`이기 때문에 구독한 시점에서 최신 이벤트인 1을 받고 구독 이후에 들어온 2, 3 이벤트를 받을 수 있다. 

세 번째 라인의 구독자는 이벤트 2이후, 3 이전에 구독을 시작했다. 역시 구독 시점에서 최신 이벤트인 2와 구독 이후에 들어온 3 이벤트를 받을 수 있다. 

```swift
enum MyError: Error {
    case anError
}

func print<T: CustomStringConvertible>(label: String, event: Event<T>) {
    print(label, event.element ?? event.error ?? event)
}

let subject = BehaviorSubject(value: "Initial Value")
let disposeBag = DisposeBag()

subject.subscribe {
    print(label: "1)", event: $0)
}.disposed(by: disposeBag)
```



> Note: `BehaviorSubject`는 항상 최신 이벤트를 내보내야 하기 때문에 디폴트 값과 함께 반드시 초기화를 진행해주어야 한다. 디폴트 값을 제공할 수 없는 상황이라면 `PublishSubject`를 사용해야 한다는 것을 의미한다. 

위의 코드를 보면 알 수 있다시피 구독은 `BehaviorSubject` 생성 후에 발생했다. 이후 다른 어떤 이벤트도 추가되지 않았기 때문에 결과는 다음과 같다. 

```
1) Initial Value
```

이번엔 이벤트를 추가해보자. 

```swift
subject.onNext("X")
```

이젠 X가 출력될 것이다. X가 최신 값이기 때문이다. 만일 에러 이벤트를 추가하면 어떻게 될까

```swift
subject.onError(MyError.anError)

subject.subscribe {
    print(label: "2)", event: $0)
}.disposed(by: disposeBag)
```

위의 코드를 살펴보면 에러 이벤트를 추가하고 새로운 구독이 생겨났다. 출력 결과는 어떻게 될까?

```
1) anError
2) anError
```

`BehaviorSubject`는 최신 데이터로 뷰를 미리 만들고자 할 때 유용하다. 예를 들어 사용자 프로필 화면의 컨트롤과 `BehaviorSubject`를 바인드하고 앱이 새로운 데이터를 가져오는 동안 최신 값(가져온 값 중 먼저 가져온 값)을 통해 미리 화면에 해당하는 뷰를 만들 수 있다. 

하지만 만약 최신 값을 하나 이상 필요로 한다면 어떻게 해야 할까? 예를 들어 검색 화면에서 최근 검색어 5개를 보여주고 싶을 수 있다. 이러한 상황에서 우리는 `ReplaySubject`를 이용해 문제를 해결할 수 있다. 



**ReplaySubject**

------

`ReplaySubject`는 지정한 크기까지 최신 이벤트들을 임시로 저장한다. (*Cache, Buffer*) 그리고 이렇게 버퍼에 저장된 이벤트들을 새로 구독자에게 다시 보낸다. 

다음 마블 다이어그램은 버퍼 크기가 2인 `ReplaySubject`이다. 첫 번째 구독자(가운데 라인)는 이미 구독을 하고 있던 상황이기 때문에 `ReplaySubject`가 내보내는 이벤트들을 모두 받을 수 있다. 두 번째 구독자(마지막 라인)는 이벤트 2 이후에 구독을 했고 버퍼에 저장된 1,2 이벤트를 모두 받고 시작한다. 

![](http://ehdrjsdlzzzz.github.io/2019/01/23/RxSwift-3/replaysubjectmarble.png)

`ReplaySubject`를 사용할 때는 버퍼가 항상 메모리를 잡고 있다는 것을 명심하자. 만일 많은 양의 메모리를 요구하는 특정 타입의 인스턴스(이미지와 같이)를 담는 크기가 큰 버퍼를 사용하는 것은 자신의 발을 쏘는 것과 마찬가지다. 그리고 또 주의해야 할 점은 배열을 담는 버퍼를 갖는 `ReplaySubject`를 사용할 경우다. 매번 내보내어지는 요소는 배열이 될 것이고 버퍼 사이즈는 해당 배열을 모두 담을 수 있는 크기가 되어야 한다. 만일 이 점을 주의하지 않는다면 메모리 압박에 시달릴 것이다. 

```swift
let subject = ReplaySubject<String>.create(bufferSize: 2)
let disposeBag = DisposeBag()

subject.onNext("1")
subject.onNext("2")
subject.onNext("3")

subject.subscribe {
    print(label: "1)", event: $0)
}.disposed(by: disposeBag)

subject.subscribe {
    print(label: "2)", event: $0)
}.disposed(by: disposeBag)
```

버퍼 사이즈가 2인 `ReplaySubject`를 생성하였다. 그리고 1,2,3을 순서대로 담았다. 그렇다면 버퍼에는 최신 데이터 2개인 2와 3을 담고 있을 것이다. 결과는 다음과 같다. 

```
1) 2
1) 3
2) 2
2) 3
```

코드를 추가해보자

```swift
subject.onNext("4")

subject.subscribe {
    print(label: "3)", event: $0)
}.disposed(by: disposeBag)
```

새로운 값을 추가하고 새로운 구독자가 등장했다. 기존의 구독자들은 최신 값(4)를 받는다. 버퍼에 담긴 값은 구독을 처음 시작한 순간에만 넘겨준다. 그리고 새로운 이벤트 4가 들어왔으니 버퍼에 존재하는 값은 3과 4다. 그러므로 새로운 구독자는 3과 4를 받는다. 

```
1) 4
2) 4
3) 3
3) 4
```

만일 마지막에 에러를 3번째 구독자를 추가하기 전, 이벤트 4를 추가한 후에서 발생시킨다면 어떻게 될까? 

```swift
subject.onNext("4")

subject.onError(MyError.anError)

subject.subscribe {
    print(label: "3)", event: $0)
}.disposed(by: disposeBag)
```

결과는 예상 밖일 수 있다. 

```
1) 4
2) 4
1) anError
2) anError
3) 3
3) 4
3) anError
```

`ReplaySubject`는 에러 이벤트와 함께 종료되었고 여타 다른 `Subject`들과 동일하게 종료된 상태에서 새로운 구독자가 등장하면 종료를 야기한 이벤트를 내보낸다. 하지만 버퍼는 여전히 살아있기 때문에 종료가 된 이후의 구독자들에게도 이를 전달할 수 있다. 

이번엔 에러 이벤트를 발생시키는 코드 밑에 다음 코드를 위치시켜보자.

```swift
subject.dispose()
```

그럼 ***Object \`RxSwift.(unknown context at 0x12293acc8).ReplayMany<Swift.String>\` was already disposed.*** 에러 이벤트를 확인할 수 있을 것이다. 

직접 `dispose()`를 호출한다면 이후의 구독자는 이미 *disposed*된 `ReplaySubject`를 구독했다는 에러 이벤트만을 받을 것이다. (버퍼 이벤트는 받지 않는다.) 

직접 `dispose()` 메소드를 호출하는 일은 일반적으로 필요하지 않다. 왜냐하면 구독을 `DisposeBag`에 담았다면 (어떠한 강한 참조 순환도 피하기 위해)  모든 것은 알아서 *disposed* 될 것이고 주인(뷰 컨트롤러 혹은 뷰 모델)이 메모리에서 사라질 때 함께 사라질 것이다. 

`PublishSubject`, `BehavoirSubject` 혹은 `ReplaySubject`를 사용함으로써 필요한 대부분을 모델링할 수 있다. 하지만 단순히 현재 값이 무엇인지만 알고 싶을 경우엔 `Variable`을 사용할 수 있다.



**Variable**

------

`Variable`은 `BehaviorSubject`를 감싼 것으로 현재 값을 상태로 저장한다. 그리고 `value` 프로퍼티를 통해 이에 접근할 수 있다. 그리고 다른 `Subject`나 `Observable`과는 다르게 이 프로퍼티를 사용해 새 값을 할당해줄 수 있다. `onNext(_)`를 사용하지 않아도 된다는 의미이다. 

`BehaviorSubject`를 감싼 것이기 때문에 디폴트 값과 함께 초기화해야 한다. 그리고 마찬가지로 최신 값이나 초기 값을 새로운  구독자에게 전달한다. 내부의 `BehaviorSubject`에 접근하기 위해선 `asObservable()` 메소드를 호출해주면 된다. 

다른 `Subject`들과 또 다른 차이점이라면 `.error`를 절대 내보내지 않는 것을 보장한다. 그리고 메모리에서 사라질 때 자동으로 `.onCompleted` 이벤트를 내보내기 때문에 직접 이를 추가할 필요가 없다. 코드로 살펴보자.

```swift
let variable = Variable("Initial value")

let disposeBag = DisposeBag()

variable.value = "New initial value"

variable.asObservable().subscribe {
    print(label: "1)", event: $0)
}.disposed(by: disposeBag)

```

내부의 `BehaviorSubject`에 접근하기 위해 `asObservable()` 메소드를 호출하고 이를 구독하였다. 결과는 쉽게 예상할 수 있다시피 최신 값인  `"1) New initial value"`가 출력될 것이다. 여기에 다음의 코드를 추가해보자.

```swift
variable.value = "1"

variable.asObservable().subscribe {
    print(label: "2)", event: $0)
}.disposed(by: disposeBag)

variable.value = "2"
```

`variable.value`를 통해 최신 값을 넣어주었다. 그리고 두 번째 구독자가 구독을 시작했고 다시 최신 값을 새로 할당해주었다. 이번에도 쉽게 예상할 수 있듯이 다음과 같은 결과물이 나온다. 

```
1) 1
2) 1
1) 2
2) 2
```

다음과 같이  `.error` 혹은 `.compeleted` 이벤트를 넣어주려 시도한다면 컴파일 에러가 발생한다. 

```swift
variable.value.onError(MyError.anError)

variable.asObservable().onError(MyError.anError)

variable.value = MyError.anError

variable.value.onCompleted()

variable.asObservable().onCompleted() 
```

`Variable`은 유연하게 사용될 수 있다. 새 이벤트가 발생할 때마다 이에 대해 반응을 해주기 위해선 `asObservable`을 통해 구독을 하면 되고 단순히 최신 값만을 체크하고 싶다면 `value` 프로퍼티를 통해 구독 없이 접근할 수 있다. 

> `Vairable`은 *deprecated* 되었고 `BehaviorReplay`로 대체되었다.