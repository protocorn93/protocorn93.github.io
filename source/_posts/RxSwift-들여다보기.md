---
title: RxSwift 들여다보기
subtitle: Observer는 어디에?
date: 2019-07-28 13:55:49
tags: ["RxSwift"]
category: ["RxSwift"]
---

### RxSwift 들여다보기

------

RxSwift를 들여다보려 한다. 이유는 간단하다. 내부 동작 원리가 궁금했다. 

이 궁금증의 시작은 RxSwift의 대표적인 개념에서부터 시작했다. 정확히 말하자면 RxSwift만의 대표적인 개념이 아니라 리액티브 프로그래밍 언어들에서 볼 수 있는 대표적인 개념들이다. 

- `Observable`
- `Observer`
- `Subscription`

리액티브 프로그래밍 언어에서 `Observable`은 이벤트 스트림을 흘려보내고 `Observer`는 이를 `Subscription`하여 이벤트에 반응하게 된다. 

처음 RxSwift를 접했을 때 이 문장을 읽고 이해하는데 그렇게 오랜 시간이 걸리지 않았다. 하지만 코드를 작성하고 읽기 시작하면서 하나의 의문점이 생겼는데, 계속 공부를 하면 해결이 될까 했던 궁금증이 여전히 해결되지 않는 모습을 보였다. 

그 궁금증의 주체는 바로 `Observer`다. 간단한 코드를 살펴보자. 

```swift
Observable<String>.just("Hello World").subscribe(onNext: { 
  print($0)
}).disposed(by: disposeBag)
```

이 곳에서 위에서 언급된 대표적인 개념들을 찾아보자. 

`Observable`과 `Subscription`은 육안으로 확인할 수 있다. 그렇다면 `Observer`는 어디 있을까. 이게 내 궁금증의 시작이었다. 코드를 그대로 해석하면 `Observable`을 바로 `Subscription`하는 모습을 확인할 수 있다.

 *"여기서 `Observer`는 누구지 "* 

그래서 RxSwift 소스 코드를 열어보았다. 



### ObservableType 

------

먼저 `ObservableType`부터 살펴보자. `ObservableType`은 프로토콜로 `Observable`은 `ObservableType`을 따른다. `ObservableType`을 바로 보기 전에 앞서 `ObservableType+Extensions.swift` 파일 안의 `extension`부터 시작해보자. 

```swift
public func subscribe(onNext: ((Element) -> Void)? = nil, onError: ((Swift.Error) -> Void)? = nil, onCompleted: (() -> Void)? = nil, onDisposed: (() -> Void)? = nil)
    -> Disposable {
        let disposable: Disposable
        
        if let disposed = onDisposed {
            disposable = Disposables.create(with: disposed)
        }
        else {
            disposable = Disposables.create()
        }
        
        #if DEBUG
            let synchronizationTracker = SynchronizationTracker()
        #endif
        
        let callStack = Hooks.recordCallStackOnError ? Hooks.customCaptureSubscriptionCallstack() : []
        
        let observer = AnonymousObserver<Element> { event in
            
            #if DEBUG
                synchronizationTracker.register(synchronizationErrorMessage: .default)
                defer { synchronizationTracker.unregister() }
            #endif
            
            switch event {
            case .next(let value):
                onNext?(value)
            case .error(let error):
                if let onError = onError {
                    onError(error)
                }
                else {
                    Hooks.defaultErrorHandler(callStack, error)
                }
                disposable.dispose()
            case .completed:
                onCompleted?()
                disposable.dispose()
            }
        }
        return Disposables.create(
            self.asObservable().subscribe(observer),
            disposable
        )
}
```

⁉️

벌써 첫 번째 궁금증이 해결되었다! `subscribe(onNext: ,onError: ,onCompleted: ,onDisposed:) -> Disposable` 내부적에서 생성되고 있었다. 

우리가 넘겨준 `onNext`, `onError`, `onCompleted` 그리고 `onDisposed`들이 내부적으로 어떻게 사용되고 있는지 살펴보자. 꽤나 직관적인 걸 알 수 있다. 하지만 첫 번째 궁금증이 해결된 것과 동시에 여러 궁금증들이 생겼다.

직관적이어도 정확히 어떻게 동작하는지는 이 코드만 봐서 나는 모두 알 수 없었다. 그래서 하나씩 따라 들어가 보았다.

```swift
if let disposed = onDisposed {
  disposable = Disposables.create(with: disposed)
}
else {
  disposable = Disposables.create()
}
```

먼저 `subscribe(onNext: ,onError: ,onCompleted: ,onDisposed:) -> Disposable` 메소드에 `onDisposed` 클로저가 넘어왔다면 이를 이용해 `Disposable`을 만들고 넘어오지 않았다면 빈 `Disposable`을 생성한다. 

```Swift
let observer = AnonymousObserver<Element> { event in            
  #if DEBUG
    synchronizationTracker.register(synchronizationErrorMessage: .default)
    defer { synchronizationTracker.unregister() }
  #endif

  switch event {
    case .next(let value):
    	onNext?(value)
    case .error(let error):
      if let onError = onError {
        onError(error)
      }
      else {
        Hooks.defaultErrorHandler(callStack, error)
      }
    disposable.dispose()
    case .completed:
    	onCompleted?()
    	disposable.dispose()
  }
}
```

여기서 생성되는 `Observer`는 `AnonymousObsever`로 생성자 함수 인자로 클로저를 받는다. 여기서 받는 클로저는 우리가  `subscribe(onNext: ,onError: ,onCompleted: ,onDisposed:) -> Disposable` 메소드의 인자로 받은 클로저들을 이벤트에 알맞게 호출해주는 행위를 정의하고 있다. 

그리고 위에서 생성된 `disposable`을 사용하고 있는 것에 주목하자. 

그럼 `AnonymousObserver`는 무엇이고 생성자로 전달된 클로저는 어떻게 사용되는 걸까 `AnonymousObserver`를 살펴보자.

 

**AnonymousObservable**

```swift
final class AnonymousObserver<Element>: ObserverBase<Element> {
    typealias EventHandler = (Event<Element>) -> Void
    
    private let _eventHandler : EventHandler
    
    init(_ eventHandler: @escaping EventHandler) {
#if TRACE_RESOURCES
        _ = Resources.incrementTotal()
#endif
        self._eventHandler = eventHandler
    }

    override func onCore(_ event: Event<Element>) {
        return self._eventHandler(event)
    }
    
#if TRACE_RESOURCES
    deinit {
        _ = Resources.decrementTotal()
    }
#endif
}
```

먼저 `AnonymousObserver`는 `ObserverBase` 클래스를 상속받고 있다. 그리고 `ObserverBase` 클래스의 `onCore(_:)` 메소드를 오버라이딩하고 있고 그 내부에서 우리가 생성자로 넘겨준 클로저를 호출하고 있는 것을 확인할 수 있다. 

그러면 이벤트가 흘러들어왔을 때 결국이 `onCore(_:)` 메소드가 호출되어야 우리가 넘겨준 클로저들도 호출될 텐데 `onCore(_:)`는 어디서 호출되는지 `ObserverBase` 클래스로 가보자. 



**ObserverBase**

```swift
class ObserverBase<Element> : Disposable, ObserverType {
    private let _isStopped = AtomicInt(0)

    func on(_ event: Event<Element>) {
        switch event {
        case .next:
            if load(self._isStopped) == 0 {
                self.onCore(event)
            }
        case .error, .completed:
            if fetchOr(self._isStopped, 1) == 0 {
                self.onCore(event)
            }
        }
    }

    func onCore(_ event: Event<Element>) {
        rxAbstractMethod()
    }

    func dispose() {
        fetchOr(self._isStopped, 1)
    }
}
```

`onCore(_:)` 메소드는`on(_:)` 메소드에서 호출되는 것을 확인할 수 있다. 그럼 다시 `on(_:)` 메소드가 호출되는 곳을 찾아야 하는데 이는 잠시 후에 밑에서 좀 더 알아보도록 하자. 일단 **`on(_:)` 메소드가 호출되어야 한다는** 사실만 기억하고 있자. 

`subscribe(onNext: ,onError: ,onCompleted: ,onDisposed:) -> Disposable` 메소드 내부에서 마지막으로 살펴볼 코드는 다음과 같다.

```swift
return Disposables.create(
  self.asObservable().subscribe(observer),
  disposable
)
```

`subscribe(onNext: ,onError: ,onCompleted: ,onDisposed:) -> Disposable` 메소드 정의에서 볼 수 있듯이 `Disposable`을 반환하는데 위와 같이 생성해서 반환한다. 

그럼 여기서 이제 나는 

```swift
self.asObservable().subscribe(observer)
```

이 코드를 먼저 살펴보려 한다. 먼저 `asObservable()`은 `Observable.swift` 파일에서 확인할 수 있다. `ObservableType` 프로토콜 메소드로 기본 구현되어 있는 메소드를 `Observable` 메소드는 아래와 같이 정의했다.

```swift
public func asObservable() -> Observable<Element> {
  return self
}
```

프로토콜인 `ObservableType`이 아닌 `Observable` 타입을 반환하기 위한 메소드이다.

그다음으로 살펴볼 것은 바로 `ObservableType.swift` 파일에 정의되어있는 `func subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element` 메소드다. 우리는 위에서 생성된 `AnonymousObserver` 객체를 이 메소드 안에 넣어주었다.  

파일에 설명된 정의를 살펴보면 다음과 같다. 

```
Subscribes `observer` to receive events for this sequence.
이 시퀀스에 대한 이벤트를 수신하기 위해선 `observer`를 구독해라.

* sequences can produce zero or more elements so zero or more `Next` events can be sent to `observer`
*시퀀스는 0개 혹은 그 이상의 요소들을 생산할 수 있고 그렇기 때문에 0개 혹은 그 이상의 `Next` 이벤트들은 `observer`에 전해질 수 있다.

* once an `Error` or `Completed` event is sent, the sequence terminates and can't produce any other elements
* `Error` 혹은 `Completed` 이벤트가 전달되면, 시퀀스는 종료되고 다른 요소를 생산할 수 없다. 

It is possible that events are sent from different threads, but no two events can be sent concurrently to `observer`.
다른 스레드로부터 이벤트가 전달될 수 있다. 하지만 두 이벤트가 동시에 `observer`에 전달될 순 없다. 

When sequence sends `Complete` or `Error` event all internal resources that compute sequence elements will be freed.
시퀀스가 `Complete` 혹은 `Error` 이벤트를 보내면 내부에서 시퀀스 요소들을 연산하던 자원들은 모두 자유로운 상태가 된다. 

To cancel production of sequence elements and free resources immediately, call `dispose` on returned subscription.
시퀀스 요소 생산을 취소하고 자원을 즉시 자유롭게하고 싶다면, 반환되는 구독 객체의 `dispose` 메소드를 호출해라.

returns: Subscription for `observer` that can be used to cancel production of sequence elements and free resources.
반환값: `observer`에 대한 구독 객체로 시퀀스 요소 생산을 취소하거나 자원을 자유롭게 하는데 사용된다. 
```

여기서 제일 위의 설명을 살펴보자.

```
이 시퀀스에 대한 이벤트를 수신하기 위해선 'observer'를 구독해라
```

그리고 [리액티브 공식 사이트](http://reactivex.io/documentation/observable.html)에서 설명하고 있는 `Observable`의 정의를 살펴보자. 

```
In ReactiveX an observer subscribes to an Observable. Then that observer reacts to whatever item or sequence of items the Observable emits
ReactiveX에서 observer는 Observable을 구독한다. 그럼 그 observer는 Observable이 방출하는 아이템에 반응한다. 
```

여기서 내가 이해한 것은 `Observer`는 `Observable`을 구독하고 그 구독 객체를 우리가 관찰하는 것으로 이해했다. 

> 음.. 구독에 대한 반응을 구독하는…? 그런 느낌으로 이해하고 있는데 이에 대해 피드백을 부탁드립니다. 😭

결국 우리가 `subscribe(onNext: ,onError: ,onCompleted: ,onDisposed:) -> Disposable`을 호출하는 대상은 `func subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element`가 반환하는 구독 객체다. 

`Observable`을 바로 구독하고 있던 것이 아니였다!

그럼 이번엔 위에서도 보았던 `Disposable`에 대해 조금 살펴보도록 하자. 먼저 `Disposable`이 `subscribe(onNext: ,onError: ,onCompleted: ,onDisposed:) -> Disposable` 안에서 여러 방식으로 생성되고 사용되고 있는 것을 확인할 수 있다.  `Disposable`을 생성하는 방법은 몇 가지가 있다. 오늘은 위에서 확인할 수 있었던 세 가지 방법에 대해서만 살펴보도록 하자. 



**AnonymousDisposable**

`subscribe(onNext: ,onError: ,onCompleted: ,onDisposed:) -> Disposable` 내부에서 살펴볼 `Disposable` 생성 방법 첫번째는 `AnonymousDisposable`이다. `AnonymousDisposable.swift`을 살펴보자.

```swift
fileprivate final class AnonymousDisposable : DisposeBase, Cancelable {
  public typealias DisposeAction = () -> Void

  private let _isDisposed = AtomicInt(0)
  private var _disposeAction: DisposeAction?

  /// - returns: Was resource disposed.
  public var isDisposed: Bool {
    return isFlagSet(self._isDisposed, 1)
  }

  /// Constructs a new disposable with the given action used for disposal.
  ///
  /// - parameter disposeAction: Disposal action which will be run upon calling `dispose`.
  fileprivate init(_ disposeAction: @escaping DisposeAction) {
    self._disposeAction = disposeAction
    super.init()
  }

  // Non-deprecated version of the constructor, used by `Disposables.create(with:)`
  fileprivate init(disposeAction: @escaping DisposeAction) {
    self._disposeAction = disposeAction
    super.init()
  }

  /// Calls the disposal action if and only if the current instance hasn't been disposed yet.
  ///
  /// After invoking disposal action, disposal action will be dereferenced.
  fileprivate func dispose() {
    if fetchOr(self._isDisposed, 1) == 0 {
      if let action = self._disposeAction {
        self._disposeAction = nil
        action()
      }
    }
  }
}

extension Disposables {

  /// Constructs a new disposable with the given action used for disposal.
  ///
  /// - parameter dispose: Disposal action which will be run upon calling `dispose`.
  public static func create(with dispose: @escaping () -> Void) -> Cancelable {
    return AnonymousDisposable(disposeAction: dispose)
  }

}
```

가장 먼저 보이는 것은 `AnonymousDisposable`은 `DisposeBase` 클래스를 상속받고 `Cancelable`을 체택하고 있다는 것이다. 그리고 `create(with:) -> Cancelable`을 보면 알 수 있듯이 `Cancelable` 프로토콜 타입으로 생성 결과를 반환한다. 그리고 `dispose` 메소드가 호출될 때 생성 인자로 받은 액션 클로저를 실행하는 것을 확인할 수 있다. 

`AnonymousDisposable`은 `subscribe(onNext: ,onError: ,onCompleted: ,onDisposed:) -> Disposable`에서 `onDisposed`에 인자가 넘어오면 생성된다. 즉 `diposed`될 때 추가적으로 해주어야 할 동작이 넘어오면 해당 동작을 실행시킬 수 있는 `AnonymousDisposable`을 생성하는 것이다. 



**NopDisposable**

그 다음으로 살펴볼 방법은 `NopDisposable`이다. 

```swift
fileprivate struct NopDisposable : Disposable {

  fileprivate static let noOp: Disposable = NopDisposable()

  fileprivate init() {

  }

  /// Does nothing.
  public func dispose() {
  }
}

extension Disposables {
  /**
  Creates a disposable that does nothing on disposal.
  */
  static public func create() -> Disposable {
    return NopDisposable.noOp
  }
}
```

코드를 살펴보면 알 수 있듯이 어떠한 추가적인 행동을 하지 않는다. `dispose` 메소드 안도 비어있다. `subscribe(onNext: ,onError: ,onCompleted: ,onDisposed:) -> Disposable`에서 `onDisposed`가 `nil`일때 `NopDisposable`을 생성한다. 즉 어떤 추가적인 행동을 취할 필요가 없을 때 `NopDisposable`을 생성하는 것으로 이해할 수 있다. 



**BinaryDisposable**

마지막으로 살펴볼 방법은 `BinaryDisposable`을 통한 생성이다. `BinaryDisposable`은 두 개의 `Disposable` 객체를 인자로 받아 생성된다. 그리고 `dispose` 호출시 두 개의 `Disposable`의 `dispose` 메소드를 호출해준다.

```swift
private final class BinaryDisposable : DisposeBase, Cancelable {

  private let _isDisposed = AtomicInt(0)

  // state
  private var _disposable1: Disposable?
  private var _disposable2: Disposable?

  /// - returns: Was resource disposed.
  var isDisposed: Bool {
    return isFlagSet(self._isDisposed, 1)
  }

  /// Constructs new binary disposable from two disposables.
  ///
  /// - parameter disposable1: First disposable
  /// - parameter disposable2: Second disposable
  init(_ disposable1: Disposable, _ disposable2: Disposable) {
    self._disposable1 = disposable1
    self._disposable2 = disposable2
    super.init()
  }

  /// Calls the disposal action if and only if the current instance hasn't been disposed yet.
  ///
  /// After invoking disposal action, disposal action will be dereferenced.
  func dispose() {
    if fetchOr(self._isDisposed, 1) == 0 {
      self._disposable1?.dispose()
      self._disposable2?.dispose()
      self._disposable1 = nil
      self._disposable2 = nil
    }
  }
}

extension Disposables {

  /// Creates a disposable with the given disposables.
  public static func create(_ disposable1: Disposable, _ disposable2: Disposable) -> Cancelable {
    return BinaryDisposable(disposable1, disposable2)
  }
}
```

`subscribe(onNext: ,onError: ,onCompleted: ,onDisposed:) -> Disposable`의 결과로 반환되는 `Disposable`은 `BinaryDisposable`이다. 그리고 이 `BinaryDisposable`은 `onDisposed` 인자값의 상태에 따라 만들어진 `Disposable` (`AnonymousDisposable` 혹은 `NopDisposable`)과 `Observable`에 대한 `Observer`의 구독 객체를 의미하는 `Disposable`로 만들어진다. `Observable`에 대한 `Observer`의 구독 객체는 우리가 `Observable`을 어떻게 생성하냐에 따라 다르다. 

그럼 `just`를 통해 생성된 `Observable`을 예로 살펴보자. 

**just**

`just`를 통해 생성된 `Observable`은 인자로 받은 단일 이벤트를 그 즉시 방출하고 바로 완료되어 `Observable`의 시퀀스가 종료된다. 

먼저 `just` 메소드부터 살펴보자. 

```swift
extension ObservableType {
  /**
  Returns an observable sequence that contains a single element.

  - seealso: [just operator on reactivex.io](http://reactivex.io/documentation/operators/just.html)

  - parameter element: Single element in the resulting observable sequence.
  - returns: An observable sequence containing the single specified element.
  */
  public static func just(_ element: Element) -> Observable<Element> {
    return Just(element: element)
  }
}
```

`Just` 객체를 `just(_:)` 인자로 받은 `Element`로 생성하여 반환한다. 그럼 `Just` 정의부를 살펴보자. 

```swift
final private class Just<Element>: Producer<Element> {
  private let _element: Element

  init(element: Element) {
    self._element = element
  }

  override func subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element {
    observer.on(.next(self._element))
    observer.on(.completed)
    return Disposables.create()
  }
}
```

먼저 `Just`는 클래스로 `Producer`를 상속받는다.

> `Producer`는 `Observable`을 상속받는다. 여기선 이 정도만 알고 넘어가자. 끝까지 들어가니 머리가 뒤죽박죽이라 현재는 제대로 이해를 하지 못했다.😭

그리고 `func subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element`  메소드를 오버라이딩하고 있다. 

`just`가 단일 이벤트를 방출하고 바로 종료된다는 뜻을 위의 코드를 통해 이해할 수 있을 것이다. 그리고 `just`는 `NopDisposable`을 생성한다. 그리고 `Observable`은 구독이 일어나기 전까지 이벤트를 방출하지 않는다는 의미를 여기서 코드로 이해할 수 있다. 

`func subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element`가 호출됨에 따라 `on(_:)` 메소드가 호출되고 `func subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element` 메소드는 ``subscribe(onNext: ,onError: ,onCompleted: ,onDisposed:) -> Disposable`` 메소드가 호출되어야 호출되는 메소드기 때문이다. 

> 그럼 위에서 " 일단 **`on(_:)` 메소드가 호출되어야 한다는** 사실만 기억하고 있자. " 라고 했던 말을 여기서 확인할 수 있다. 



뭔가 흐름이 잡힌 것 같으면서도 잡히지 않은 것 같은 느낌이 든다. 아직 왜 여기서 `BinaryDisposable`을 사용하는 이유는 잘 이해가 가질 않는다. 그래도 `Observable`의 생성되고 이벤트에 반응하는 것까지의 흐름을 파악할 수 있어서 유익한 시간이었다. 아직 100프로 이해하지 못한 부분들에 대해서는 추후에 계속해서 이해해보려 노력할 것이다. 



### 참고자료

------

- [[RxSwift] Observer](https://medium.com/@rkdthd0403/rxswift-observer-fdc8d2772d6c)
  - 나와 같이 `Observer` 존재에 대한 의문을 갖고 글을 작성하셨다. 덕분에 어디부터 어떻게 살펴보아야 할지 감을 잡을 수 있었다. 
- [RxSwift Disposable](https://medium.com/cashwalk/rxswift-disposable-fbfb4114fa6e)
  - 역시 RxSwift의 코드를 열어보면서 코드들의 존재 이유에 대해 서술해주신 글이다.

