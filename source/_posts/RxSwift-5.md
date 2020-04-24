---
title: RxSwift (5)
subtitle: observeOn vs subscribeOn
date: 2019-05-05 17:04:07
tags: ["subscribeOn", "observeOn", "RxSwift", "Scheduler"]
category: ["RxSwift"]
---

## subscribeOn vs observeOn

오늘은 RxSwift를 공부하면서 이해가 잘 가지 않았던 `subscribeOn`과 `observeOn`의 차이점에 대해 공부하고 이를 기록하고자 한다. 

오늘은 **스케쥴러(Scheduler)**보단 `subscribeOn`과 `observeOn` 두 차이점에 대해 자세히 알아보려 한다. 



### Intro

------

먼저 이 둘의 차이점에 대해 알아보기 전에 먼저 `Observable`에 대한 구독이 일어나는 과정을 분리해서 살펴보자. 

> `subscribeOn`과 `observeOn`에 대해 공부할 때 이를 설명하고 있는 용어들이 설명하고 있는 곳마다 다소 차이가 보여 혼란스러웠다. 하지만 [observeOn vs subscribeOn](http://rx-marin.com/post/observeon-vs-subscribeon/) 이 글을 읽고 용어의 정리와 구독의 과정을 단계 별로 나누어 확실히 이해할 수 있었다. 구독이 일아나는 과정에 대한 내용은 위의 글을 대부분 참고해서 작성하였다. 

먼저 구독을 다음과 같이 세 단계로 나누어 볼 수 있다.

![](http://rx-marin.com/images/subscribe-schema.png)

1. 먼저 `Observable`을 정의한다. 위의 예제처럼 `create`를 통해 `Observer`들에게 보낼 이벤트를 만들 수도 있다. 이렇게 `Observable`을 정의하는 행위는 그 즉시 실행을 의미하지 않는다. `Observable`은 구독이 시작되기 전에는 어떤 이벤트도 내보내지 않는다. 즉 `{ }` 안의 코드는 누군가 구독을 시작하지 않으면 실행되지 않는다. 
2. `map`, `filter`와 같은 연산자를 통해 생성된 이벤트에 대한 가공 행위를 해줄 수 있다. 하지만 이 역시 구독이 시작되기 전에는 실행되지 않는다. 
3. `subscribe` 메소드를 호출했을 때만 `Observable`은 이벤트를 만들어 내보내기 시작한다. 즉 `create { }` 구문이 실행된다. 



여기서 우리는 **subscription code**와 **observation code**로 나누어 살펴볼 수 있다. 

![](http://rx-marin.com/images/subscribe-terms.png)

- **Subscription code**는 `subscribe` 메소드에 의해 호출되는 코드로 위의 예제에서 `create`의 블록 안의 코드이다. 즉 이벤트 요소를 생성하는 코드가 subscription code이다. 
- **Observation code**는 방출되는 이벤트 요소들을 관찰하는 코드로 `onNext : { … }` 혹은 `onCompleted: { … }`와 같은 코드들을 말한다. 실제로 값의 관찰이 일어나는 곳이다. 

> 용어 때문에 subscription code는 `subscribe` 블록 안의 코드로 이해하였고 그로 인해 혼란스러웠다. 하지만 `subscribe`에 의해 호출되는 코드라 이해한 뒤로 구분을 확실히 할 수 있었다.



### subscribeOn

------

먼저 이 둘의 차이점을 살펴보기 전에 이 둘을 나타내는 마블 다이어그램부터 살펴보자!

![](http://reactivex.io/documentation/operators/images/schedulers.png)



`subscribeOn`을 살펴보자 파란색으로 스레드로의 변경을 했다. 그리고 이런 스레드 변화의 영향을 받는 곳은 `Observable`이 이벤트를 생성해서 내보내는 부분이다. **즉 `subscribeOn`은 Subscription code의 스레드를 결정해주는 역할을 한다. ** 다시 말하자면 `Observable`이 이벤트를 어느 스레드에서 생성하는지를 결정해준다. 

`subscribeOn`을 사용하지 않으면 기본적으로 subscription code는 `subscribe`가 호출된 스레드에서 작동한다. 

결과적으로 `subscribeOn`은 `subscribe`를 호출할 때 스레드를 변경하기 때문에 오퍼레이터 체인에서 어느 곳에 위치하여도 상관없다. 



### observeOn

------

위의 다이어그램에서 `observeOn`을 살펴보자. `observeOn`을 통해 작업 스레드를 변경하면 바로 아래 스트림에 영향을 준다. `subscribeOn` 보다는 비교적 직관적인 것을 알 수 있다. `observeOn`은 연산자가 진행하는 작업이 진행될 스레드를 지정해준다. 그렇기 때문에 여러번 사용될 수 있고, 그 위치에 따라 다르게 작동할 수 있다. 

예제 코드를 통해 둘의 차이를 살펴보자.

```swift
let backgroundScheduler = SerialDispatchQueueScheduler(globalConcurrentQueueQOS: .Default)

[1,2,3,4,5].toObservable()
	.subscribeOn(MainScheduler.instance) 	// 1
	.doOnNext {
  	UIApplication.sharedApplication().networkActivityIndicatorVisible = true
  	return $0
	} 		// 2
	.observeOn(backgroundScheduler) // 3
	.flatMapLatest {
  	HTTPBinDefaultAPI.sharedAPI.get($0)
	}		// 4
	.observeOn(MainScheduler.instance) 		// 5
	.subscribe {
  	UIApplication.sharedApplication().networkActivityIndicatorVisible = false
  	print($0)
}		// 6
```

> 다른 글들의 예제에선 데이터를 받아오는 스레드를 한 번 UI를 처리해주는 스레드를 한 번 이렇게 두 번 지정하는 예제를 사용하는 반면 민소네님 글의 예제에선 받아오는 동안의 UI처리를 위한 스레드 지정이 한 번 더 들어가 있어 더 좋은 예제인 것 같아 이 예제를 사용하였습니다. 
>
> 출처 : [민소네님 블로그](http://minsone.github.io/programming/reactive-swift-observeon-subscribeon)

1. `Observable`의 이벤트 방출 스레드를 메인으로 지정한다. 
2. 네트워크를 호출하기 전 사용자에게 이를 알리기 위한 UI 작업을 진행한다. 시작을 메인 스레드에서 시작하였기 때문에 UI 작업을 위한 스레드 변경 작업이 필요 없다. 
3. UI가 블록되지 않고 네트워크 호출을 하고 결과물을 받기 위해 api 호출 전에 백그라운드 스레드로 변경한다. 
4. api 호출
5. 네트워크 호출 결과물로 UI를 갱신하기 위해 다시 메인 스레드로 변경한다.
6. UI 갱신



### Raywenderlich

------

다음은 내가 Raywenderlich의 RxSwift 책을 보면서 이해하지 못했던 부분이다. 이렇게 정리가 된 후에 읽어보니 그 의미를 이해할 수 있었다.

> 출처 : [Raywenderlich](https://store.raywenderlich.com/products/rxswift)

------

#### Using subscribeOn

In some cases you might want to change on which scheduler the *observable computation code* runs — not the code in any of the *subscription operators*, but the *code that is actually emitting the observable events*.

> 여기서 언급하는 *observation computation code*와  *code that is actually emitting the observable events*가 바로 `create { … }` 구문을 말한다. 그리고 *subscription operators*은 말그대로 연산자로 `map`, `filter`와 같은 연산자를 의미하는 것으로 보인다.

The way to set the scheduler for that computation code is to use subscribeOn. It might sound like a counterintuitive name at first glance, but after thinking about it for a while, it starts to make sense. *When you want to actually observe an observable, you subscribe to it. This determines where the original processing will happen.*

> 역시 책에서도 용어의 혼란을 언급하고 있다. `Observable`을 관찰하기 위해선 먼저 구독(subscribe)이 선행되어야 하기 때문에 `subscribeOn`이라는 용어를 사용한다고 설명하고 있다.
>
> 그리고 본연의 이벤트가 어디서 처리될지를 결정한다고 되어 있는데 본연의 이벤트 역시 `create { … }`와 이벤트가 방출되는 시점을 말하고 있는 것으로 보인다. 

If subscribeOn is not called, then RxSwift automatically uses the current thread

#### Using observeOn

Observing is one of the three fundamental concepts of Rx. It involves an entity producing events, and an observer for those events. In this case, and in opposition to subscribeOn, the operator *observeOn changes the scheduler where the observation happens.*

> `observeOn`은 관찰이 발생하는 곳의 스레드를 변경해줄 때 사용한다고 설명하고 있다. 이벤트를 관찰하여 이벤트를 가공하는 연산자들 역시 여기에 해당한다.

So once an event is pushed by an Observable to all the subscribed observers, this operator will ensure that the event is correctly handled in the correct scheduler.



**참고 자료**

- [ReactiveX.io](http://reactivex.io/documentation/scheduler.html)
- [observeOn vs subscribeOn](http://reactivex.io/documentation/scheduler.html)
- [[ReactiveX][RxSwift]Scheduler, observeOn, subscribeOn](http://minsone.github.io/programming/reactive-swift-observeon-subscribeon)
- [마블 다이어그램 이해하기: observeOn 과 subscribeOn](https://youtu.be/GPvJGFa4UOk)