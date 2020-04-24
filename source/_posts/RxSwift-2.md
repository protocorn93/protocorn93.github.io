---
title: RxSwift (2)
subtitle: Observable
date: 2019-01-18 18:51:57
tags: ["Observables", "Subscribe", "Operators", "Dispose", "DisposeBag"]
category: ["RxSwift"]
---

### Observables 

`Observable`은 Rx의 핵심이라 할 수 있다. `Observable`, `Observable Sequence`, `Sequence` 모두 같은 것을 의미한다. `Observable`은 몇몇 특별한 기능을 갖고 있는 시퀀스이다. 그 중 중요한 하나가 바로 `asynchronous`이다. 

`Observable`은 한 주기동안 이벤트들을 생성하는데 이를 *emitting*이라 부른다. 이벤트는 숫자나 사용자 정의 타입의 인스턴스와 같은 값을 포함할 수도 있고 탭과 같은 제스쳐로 인식될 수도 있다. 

이것을 개념화하기 가장 좋은 방법은 마블 다이어그램을 사용하는 것이다. 

<img src="http://i1.wp.com/adamborek.com/wp-content/uploads/2016/11/rxMarbles_events.png?zoom=2&w=1080">

왼쪽에서 오른쪽으로 향하는 화살표는 타임라인을 의미하며 시간 순서대로 발생한 이벤트들을 원형의 마블로 왼쪽부터 나열한다.

이렇게 생성되는 이벤트를 `next` 이벤트라 한다. 그리고 타임라인에 수직으로 그려져 있는 선은 이벤트 발생이 끝났음을 의미한다. 이를 `completed` 이벤트라한다. 이벤트 발생이 끝난 것과 이벤트가 발생하지 않는 것의 차이점을 알아야 하는데 `completed` 이벤트는 탭을 할 수 있는 뷰가 사라져 더 이상 탭을 할 수 없는 상황을 말하고 이벤트가 발생하지 않는 것은 단순히 사용자가 탭을 하지 않는 상황을 말하며 언제든 이벤트가 발생할 수 있는 환경이다. 이렇게 `completed` 이벤트를 끝으로 `Observable`이 종료되는 것이 정상적인 종료이다. 

하지만 두 번째 그림처럼 X 표시가 있다는 것은 오류가 발생한 것으로 이를 `error` 이벤트라하며 `error` 이벤트가 발생하면 역시 `Observable`은 종료된다. 

다음의 코드를 시작으로 `Observable`을 직접 만져보며 익혀보자. 위에서 언급했듯이 `Observable`은 값을 포함하는 이벤트를 만들어낸다. 그리고 우리는 그렇게 만들어진 이벤트들이 적절히 처리되기 전에 가공할 수 있다. 이렇게 이벤트를 가공하는 행위를 하는 것이 `Operator`이다. 먼저 3개의 `Operator`를 살펴보자. 

**just**

`just`는 단일 요소를 인자로 받는다.

```swift
let one = 1
let two = 2
let three = 3

let observable: Observable<Int> = Observable<Int>.just(one)
```

`Int` 타입을 명시해주었다. 

**of**

```swift
let observable2 = Observable.of(one, two, three)
let observable3 = Observable.of([one, two, three])
```

`of`는 여러 개의 값을 인자로 받을 수 있으며 그를 통해 타입을 추론한다. 위에서 배열은 하나의 인자로 넘어간 것이다. `just`도 배열을 인자로 받을 수 있다. 배열 역시 *단일* 요소로 취급 받기 때문이다. 

**from**

```swift
let observable4 = Observable.from([one, two, three])
```

`from`은 배열 속의 각 요소들로부터 개별 타입 인스턴스의 `Observable`를 생성한다. `from`은 배열만을 인자로 받는데 `from(one)`을 해도 내부적으로는 `[one]`을 받는걸로 되어있어 타입을 검사해보면 `[Int]`인 것을 확인할 수 있다.

그럼 이제 이렇게 만들어진 `Observable`들을 *subscribe* 해보자.

### Subscribe

`Observable`을 **subscribe(구독)** 한다는 것을 `Observable`이 만들어낸 이벤트들을 받을 수 있다는 것이다. 이는 iOS에서 `NotificationCenter`와 비슷한 모습을 보이나 RxSwift에서 `Observable`은 subscriber가 존재하기 전에는 이벤트를 발생시키지 않는다. 

위에서 언급했듯이 `Observable`은 서로 다른 타입(`next`, `error`, `completed`)를 발생시키고 subscriber에서 우리는 이벤트 타입 별로 서로 다른 핸들러를 작성해줄 수 있다. 

```swift
let observable = Observable.of(one, two, three)
observable.subscribe { event in 
	print(event)
}
```

`subsribe` 메소드의 형태는 다음과 같다.

```swift
func subscribe(_ on: @escaping (Event<Int>) -> Void) -> Disposable
```

이벤트 타입을 인자로 받고 아무것도 반환하지 않는 형태의 `@escaping` 클로저를 인자로 받고 `Disposable`을 반환 타입으로 갖는다. 위 코드의 실행 결과는 다음과 같다. 

```
next(1)
next(2)
next(3)
completed
```

`Observable`은 각 요소에 해당하는 `next` 이벤트를 생성하고 마지막으로 `completed` 이벤트를 생성하고 종료된다. 그리고 이벤트가 아닌 요소, 값에 접근하기 위해선 다음과 같이 코드를 작성할 수 있다. 

```swift
observable.subscribe { event in 
	if let element = event.element {
    	print(element)                  
	}
}

observable.subscribe(onNext: { element in
	print(element)
})
```

이번엔 조금은 다른 유형의 `Operator`를 알아보자. 먼저 `empty`를 살펴보자. 

`empty`는 말 그대로 비어있는 `Observable`로 비어있는 `Observable`로 `completed` 이벤트만 생성하고 종료된다. 즉시 종료되거나 의도적으로 아무 값도 생성하지 않는 `Observable`을 만들어야 하는 상황에 사용된다. 타입을 명시해주어야 하고 `Void`로 명시해주었다. 

```swift
let emptyObservable = Observable<Void>.empty()

emptyObservable.subscribe(
	onNext: { element in 
    	print(element)
    },
    onCompleted: {
        print("Completed")
    }
)

// Output : Completed
```

다음으로 살펴볼 것은 `never`이다. 아무 이벤트도 생성하지 않는 `Operator`로 *절대* 종료되지 않는다. 무한의 기간을 표현할 때 사용되곤 한다. 

```swift
let neverObservable = Observable<Any>.never()

neverObservable.subscribe(
    onNext: { element in 
        print(element)
    },
    onCompleted: {
        print("Completed")
    }
)
// Output : 
```

### Disposing and Terminating

`Observable`은 구독되기 전에는 어떠한 이벤트도 발생시키지 않는다. 구독은 `Observable`의  `.error`나 `.completed` 이벤트가 발생되어 종료될 때 까지 이벤트를 발생시킨다. 하지만 구독을 취소함으로써 우리는 `Observable`을 종료시킬 수 있다. 

**Dispose**

```swift
let observable = Observable.of(1,2,3)
let subscription = observable.subscribe { event in
    // Handling Event
}
subscription.dispose()
```

`.subscribe()` 메소드의 반환 타입을 살펴보면 `Disposable` 이라는 것을 확인할 수 있다. `Disposable`의 사전적 의미를 찾아보면 *사용 후 버릴 수 있는* 이라는 뜻이다. 

> 사실 이 뜻이 지금 사용하는 입장에서 크게 와닿지는 않는다. 구독을 하고 쉽게 이를 취소할 수 있다는 것을 의미하는 것인가?  데이터의 흐름을 쉽게 제어할 수 있다는 뜻인가? 이는 조금 더 경험봐야 알 것 같다.

우리는 `Disposable`의 메소드인 `dispose()`를 호출함으로써 구독을 취소할 수 있다. 사용이 끝난 혹은 이벤트 발생이 종료된 `Observable`을 *dispose* 해주지 않는다면 메모리 누수의 위험이 있다. 그렇다고 모든 구독에서  `dispose()` 메소드를 호출해주는 것은 개발자의 실수(호출하는 것을 잊어버리는 등)를 유발할 수 있다. 그렇기 때문에 RxSwift에선 `DisposeBag` 타입을 제공한다. 

```swift
var disposeBag = DisposeBag()
let observable = Observable.of(1,2,3)
observable.subscribe { event in
    // Handling Event
}.disposed(by: disposeBag)
```

이렇게 `disposeBag`에 담아놓고 한 곳에서 `disposeBag`의 값을 초기화해줌으로써 담고 있는 구독들을 손쉽게 모두 취소할 수 있다.  

이번엔 기존의 `Observable`을 만드는 방법(`of`, `from`, `just`)이 아닌 `create` operator로 만드는 방법을 알아보자.

```swift
var disposeBag = DisposeBag()
Observable<String>.create { observer in 
    observer.onNext("1")
    
    observer.onCompleted()
    
    observer.onNext("2")
    
    return Disposables.create()
}.subscribe(onNext: {
    print($0)
}, onError: {
    print($0)
}, onCompleted: { 
	print("Completed")
}, onDisposed: {
    print("Disposed")
})

// output 
// 1
// Completed
// Disposed
```

코드를 살펴보기 전 먼저 `create` 메소드의 외형부터 살펴보자. 

```swift
static func create(_ subscribe: @escaping (AnyObserver<String>) -> Disposable) -> Observable<String>
```

`subscribe`는 `AnyObserver<String>`을 인자로 받고 `Disposable`을 반환해주는 클로저 타입이다. 그렇기 때문에 마지막 블록의 마지막 라인에 `Disposables.create()`로 빈 `Disposable` 객체를 반환해준다. 

그리고 코드의 결과는 보면 알 수 있다시피 `onCompleted()`가 먼저 호출되었기 때문에 `onNext("2")`는 실행되지 않는다. 즉 블록 안에서 `Observable`이 만들어내는 이벤트들을 정의해주는 것이다.

### Using Traits 

`Trait`는 `Observable`의 기능을 좁힌 것으로 어디서든 `Observable`로 대체가 가능하며 선택적으로 사용할 수 있다. `Trait` 를 사용함으로써 코드를 읽는 다른 사람에게 보다 개발자의 의도를 직관적이고 명확하게 전달할 수 있다. 

`Trait`의 종류에는 세 가지가 있다. 

- **Single**
  - `.success(value)` 혹은 `.error` 둘 중 하나만을 반환하며 `.success(value)`는 `.next`와 `.completed`로 구성되어 있다. 
  - 이는 데이터를 다운로드 받거나 디스크로부터 불러오는 것과 같이 성공한다면 값을 포함하고 실패한다면 에러를 포함하는 결과물을 갖는 일회성 프로세스에 유용하다. 
- **Completable**
  - `.completed` 와 `.error` 이벤트만을 반환하고 어떠한 값도 반환하지 않는다. 
  - 디스크에 데이터 쓰기 작업과 같이 작업의 완료 유무만을 판단하는 프로세스에 유용하다.
- **Maybe**
  - **Single**과 **Completable**을 섞어 놓은 것으로 작업의 결과가 성공 혹은 실패로 나뉘어지며 결과 값이 있을 수도 없을 수도 있는 작업에 유용하며 `.success(value)`, `.error`, `.completed`를 반환할 수 있다. 

Single을 예로 코드를 작성해보자.

```swift
func getRepo(_ repo: String) -> Single<[String: Any]> {
    let url = URL(string: "https://api.github.com/repos/\(repo)")!
    return Single<[String: Any]>.create { single in
        let task = URLSession.shared.dataTask(with: url) { data, _, error in
            if let error = error {
                single(.error(error))
                return
            }

            guard let data = data,
                  let json = try? JSONSerialization.jsonObject(with: data, options: .mutableLeaves),
                  let result = json as? [String: Any] else {
                single(.error(DataError.cantParseJSON))
                return
            }

            single(.success(result))
        }

        task.resume()

        return Disposables.create { task.cancel() }
    }
}
```

위와 같이 url로부터 데이터를 받아오는 상황이고 결과에 따라 적절히 `.error`와 `.success`를 사용해준다. 그리고 이를 다음과 같이 구독하여 이벤트들을 처리할 수 있다.

```swift
getRepo("ReactiveX/RxSwift")
    .subscribe { event in
        switch event {
            case .success(let json):
                print("JSON: ", json)
            case .error(let error):
                print("Error: ", error)
        }
    }
    .disposed(by: disposeBag)
```

