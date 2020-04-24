---
title: RxSwift (1)
subtitle: Functional Reactive Programming 패러다임
date: 2019-01-17 21:03:43
tags: ["FRP", "RxSwift", "Reactive Programming"]
category: ["RxSwift"]
---

> Let us: Go! 2017 가을 행사에서 곰튀김님이 발표하신 내용을 바탕으로 정리하였습니다. 리액티브 프로그래밍을 처음 시작하는 저에게는 확실한 **용어 정리**가 필요하였는데 굉장히 직관적이고 명확하게 전달해주셔서 많은 도움이 되었습니다. 
>
> https://youtu.be/cXi_CmZuBgg

현재의 개발 패러다임은 `Concurrency` 시대라고 할 수 있다. 동시에 여러 프로그램이 돌아가야 하고 각 프로그램은 동시에 돌아가고 있는 다른 프로그램에 영향을 주어선 안된다. 그렇기 때문에 **퍼포먼스**와 **신뢰성**에 중점을 두게 된다. 

**Immutable**

동시성 프로그래밍에선 공유되는 자원에 대한 동기화 이슈가 발생하기 마련이다. 기존에는 이 문제를 **세마포어**, **뮤텍스** 등을 사용하여 해결하였다. 최근엔 데이터를 공유해아하므로 동시에 `write`를 못하도록 한다. 이를 반영하듯 최신 프로그래밍 언어들에선 `Immutable`한 변수 타입을 제공한다. (`let`)

**Side effect, Pure function**

두 예제를 통해 살펴보자.

```swift
1 var data: Int = 10
2
3 func addAndGet(add: Int) -> Int {
4     return data + add
5 }
6 
7 addAndGet(add: 1) // 11
8 data = 20
9 addAndGet(add: 1) // 21
```

7번, 9번 라인을 살펴보자. 동일한 함수에 동일한 파라미터를 값을 넣어주었는데 각기 다른 결과를 반환한다.

```swift
1 let data: Int = 10
2 var backUp: Int = data
3 
4 func addAndGet(add: Int) -> Int {
5     let result = data + add 
6     backUp = result
7     return result
8 }
9 
10 backUp // 10
11 addAndGet(add: 1) // 11
12 backUp // 11
13 addAndGet(add: 1) // 11
```

11번, 13번 라인을 살펴보자. 동일한 함수에 동일한 파라미터 값을 넣어주어 같은 결과를 반환하였지만 `backup` 변수의 값이 변경되었다. 이렇게 단순히 하나의 함수가 돌아가면서 외부의 하나의 변수만 영향을 주면 별 것 아닌 것 처럼 보일 수 있지만 여러 함수가 돌아가면서 여러 변수에 영향을 주고 받게 된다면 값을 예측하기 매우 힘들어질 것이다. 

이렇게 외부의 영향을 받고 영향을 주는 현상을 우리는 **Side effect**라 부른다. Side effect 없이 코드를 작성하면 우리는 다음과 같이 작성할 수 있다. 

```swift
func addAndGet(base: Int, add: Int) -> Int {
    return baes + add
}

addAndGet(base: 10, add: 1) // 11
addAndGet(base: 20, add: 1) // 21
```

위와 같이 작성하면 동일한 함수에 동일한 파라미터 값에 대한 결과는 항상 같을 뿐만 아니라 파라미터 값을 제외하곤 외부에 어떠한 영향도 받지 않게 되며 외부의 어떠한 값에도 영향을 주지 않게 된다. 이러한 함수를 우리는 **순수 함수,** **Pure function**이라 부른다.

**Composition**

함수에는 크게 세 가지 종류가 존재한다. 

1. `input`은 없고 `output`만 존재하는 함수.
2. `input`과 `output` 모두 존재하는 함수.
3. `input` 만 존재하는 함수.

> `input`과 `output` 둘 다 존재하지 않는 함수는 함수라기 보단 액션이라 구분해서 부른다.

```swift
func getTen() -> Int { // --- 1
    return 10
}

func addTwo(n1: Int, n2: Int) -> Int { // --- 2
    return n1 + n2
}

func output(value: Int) { // --- 3
    print("\(value)")
}

output(addTwo(n1: getTen(), n2: getTen())) // --- Composition
```

이렇게 위에서 나열한 세 가지 종류의 함수를 합성해서 사용하는 것을 **Composition**이라 부른다. 그리고 위의 세 가지 종류의 함수를 각각 순서대로 `Generator`, `Function` 그리고 `Consumer`라고 구분해서 부른다.

**First Class Citizen(일급 객체), High-Order function(고차 함수)**

프로그래밍 언어에서 변수에 담을 수 있고 인자 값으로 사용할 수 있으며 반환 값에도 사용될 수 있는 것을 우리는 **일급 객체**라고 부른다.

```swift
func getTen() -> Int {
    return 10
}

func output(value: Int) {
    print("\(value)")
}

let adder = { (v1, v2) -> Int in v1 + v2 }
let multiplier = { (v1, v2) -> Int in v1 * v2 }

func calculate(n1: Int, n2: Int, method: (Int, Int) -> Int) -> Int {
    return method(n1, n2)
}

output(value: calculate(n1: getTen(), n2: getTen(), method: adder)) // 20
output(value: calculate(n1: getTen(), n2: getTen(), method: multiplier)) // 100
```

위의 코드를 살펴보면 스위프트에선 함수가 **일급 객체**가 될 수 있다는 것을 확인할 수 있다. 그리고 함수가 함수를 받고 함수 내에서 함수를 사용하는 등의 함수를 우리는 **고차 함수**라고 부를 수 있다. (`filter`, `map` 등등)

> 우리는 스위프트를 `functional`한 언어라고 부를 수 있다. 그리고 스위프트에선 객체, 함수, 프로토콜 그리고 구조체들도 동일하게 사용될 수 있으므로 우리는 스위프트를 **멀티 패러다임 언어**라고 부를 수있다.

**명령형 프로그래밍, 선언형 프로그래밍**

데이터를 정의하고 그것의 변화 과정을 프로그래밍 하는 것을 **명령형 프로그래밍**, 행위를 정의하고 거기에 데이터를 넣는 것이 **선연형 프로그래밍**이다. 생각의 주체가 이동한 것이라고 보면 된다. 생각의 주체를 데이터에 두느냐, 함수에 두느냐의 차이다. **함수형 프로그래밍은 선언형 프로그래밍**에 해당한다. 

위의 내용들을 토대로 정리하자면 다음과 같다. 

함수형 프로그래밍은 

- 데이터는 **Immutable**하게 처리해야 하고
- 데이터의 변경이 필요할 때는 새로 만들며
- **Side effect**를 없애기 위해 **순수 함수(Pure function)**을 사용해야 한다.
- 또한 함수들의 **Composition**과 **High-Order function**을 이용하여
- **데이터**가 아닌 **프로세스**에 집중해서 프로그램을 만들어야 한다. 



**Reactive Programming이란?**

리액티브 프로그래밍이란 비동기적인 상황에서 비동기로 받아오는 데이터를 어떻게 처리할 것이냐에 대한 아이디어로 **스트림(Stream)**이란 것을 이용하고 그곳에 데이터를 흘려보내자라는 아이디어이다. 이는 아이디어로 이들을 구현한 것이 `RxSwift`, `ReactivSwift` 등이다. 

리액티브 프로그래밍의 아이디어를 구현한 RxSwift을 기준으로 기본적인 흐름을 살펴보자. 데이터는 생성되고 처리되어야 한다. 전자는 데이터를 생성하는 제너레이터가 될 것고 후자는 데이터를 처리하는 컨슈머가 될 것이다. 그리고 이 둘을 스트림으로 이어준다. 

RxSwift에서 제너레이터는 **Observable**이 되고 컨슈머는 **Subscriber**가 되며 이 둘 사이의 **스트림**을 통해 데이터가 흘러가는 과정에서 **Operator**(filter, map, reduce)를 통해 데이터를 변형하고 가공할 수 있다.

코드로 예를 살펴보자. 서버로부터 이미지가 저장된 url을 받아오고 다시 해당 url로 부터 이미지를 가져오는 작업을 우리는 RxSwift를 사용해 다음과 같이 사용할 수 있다. 

```swift
import RxSwift

...

func getImageUrl() -> Observable<String> {
    let urlString = "https://api.server/image"
    return Observable.create({ emitter in 
    	let url = URL(string: urlString)
		URLSession().dataTask(with: url) { (data, response, error) in 
			guard error == nil else { return }
			emitter.onNext(String(data: data!), encoding: .utf8)
			emitter.onCompleted()
		}
		return Disposables.create()
	})
}

func requestDownload(_ url: URL) -> Observable<UIImage> {
    return Observable.create({ emitter in 
		URLSession().dataTask(with: url) { (data, response, error) in 
			guard error == nil else { return }
			let image = UIImage(data: data!)
			emitter.onNext(image)
			emitter.onCompleted()
		}
		return Disposables.create()
	})
}
```

위와 같이 `getImageUrl`은 서버로부터 이미지가 저장된 url을 요청하고 그 값을 스트림에 흘려보낸다(`onNext`). 그리고 `requestDownload(_:)`는 `URL` 타입의 url 값을 받아 이미지를 받아내고 이미지를 역시 스트림에 흘려보낸다. 

이 두 메소드를 사용해서 `UIImageView`에 이미지를 할당해보자. 

```swift
class ViewController: UIViewController {
    @IBOutlet weak var imageView: UIImageView!
    let disposeBag = DisposeBag()
    ...
    override func viewDidLoad(){
        super.viewDidLoad()
        
        getImageUrl()
        	.flatMap( { self.requestDownload(URL(string: $0!)) })
        	.subscribe(onNext: { self.imageView.image = $0 })
        	.disposed(by: disposeBag)
    }
}
```

RxSwift도 당연히 비동기적으로 동작한다. 하지만 기존에 델리게이트나 콜백을 사용했을 때와 비교해서 코드를 작성한 순서대로 동작을 하고 읽기에도 수월하다는 장점이 존재한다.

리액티브 프로그래밍을 정리해보면 다음과 같다. 

- Async한 처리를 Functional하게 처리하자.
- 반환 값은 **스트림**인 **Observable**을 반환하자.
- 스트림에 흐르는 Data/Event를 **Operator**로 처리하자. 
- 스트림과 스트림을 연결하자. 
- Data가 아닌 **프로세스에 집중해서 프로그래밍**하자. 

전체적으로 정리해보자면 다음과 같이 정리할 수 있다. 

- **Functional**
  - Concurrency한 시대 환경에서 여러 프로그램들이 함께 돌아가고 외부에 영향을 주지 않아야 하기 때문에 Side effect가 없도록 프로그래밍하는 **패러다임**
- **Reactive**
  - Async한 작업을 Functional하게 처리하는 **아이디어**
- **RxSwift** 
  - Reactive 아이디어를 구현한 Swift **라이브러리**