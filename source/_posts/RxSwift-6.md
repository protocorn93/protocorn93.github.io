---
title: RxSwift (6)
subtitle: 테이블 뷰 데이터 바인딩 이해하기 - 1
date: 2019-05-06 22:41:24
tags: ["RxSwift", "bind"]
category: ["RxSwift"]
---

오늘은 RxSwift를 공부하면서 이해하기 힘들었던 메소드를 정말 간단한 분석을 통해 그 외형을 이해해보려 한다.

> 함수의 인자와 반환값을 이해하기 위한 정말 간단한 외형 분석이다. 내부 로직 분석에 관한 글이 아니다.😅

---

본인이 제대로 이해하지 못했던 부분 데이터를 테이블 뷰 셀에 바인딩하는 다음의 코드이다. 

```swift
viewModel.data.asObservable()
	.observeOn(MainScheduler.instance)
	.bind(to: tableView.rx.items(cellIdentifier: "Cell")) { _, data, cell in
		cell.titleLabel.text = data.title
    cell.descriptionLabel.text = data.description
  }
	.disposed(by: disposeBag)
```

바로 위에서 `bind(to : tableView.rx.items(cellIdentifier: "Cell")) { … }` 부분이 본인이 이해하지 못한 코드이다. 그래서 `bind<R1, R2>(to: curriedArgument:) -> R2` 함수가 어떻게 작성되어 있는지 살펴보았다. 그 코드가 바로 아래의 코드다.

이 메소드를 다음과 같이 설명하고 있다

*사용자 정의 바인더 함수를 이용해 Observable 시퀀스를 구독하고 `self` (`ObservableType` 자신)를 바인더 함수에 넣은 결과물에 마지막 파라미터를 바인더 함수에 넘긴다.* 

자세한 내용과 사용법은 추후에 다루도록 하고 오늘은 단순히 **외형**에만 집중해보자!

```swift
extension ObservableType { 
  /**
  Subscribes to observable sequence using custom binder function and final parameter passed to binder function
  after `self` is passed.

  public func bind<R1, R2>(to binder: Self -> R1 -> R2, curriedArgument: R1) -> R2 {
  return binder(self)(curriedArgument)
}

  - parameter to: Function used to bind elements from `self`.
  - parameter curriedArgument: Final argument passed to `binder` to finish binding process.
  - returns: Object representing subscription.
  */
  public func bind<R1, R2>(to binder: (Self) -> (R1) -> R2, curriedArgument: R1) -> R2 {
    return binder(self)(curriedArgument)
  }
}
```

제네릭 타입 `R1`과 `R2`를 사용한다. 기본적으로 이 메소드는 두개의 인자와 한 개의 값을 반환하는 외형을 갖는다. 

첫 번째 인자는  `Self`, 즉 `ObservableType`을 따르는 타입을 인자로 받고 그 결과로 `R1`을 인자로 받아 `R2`를 반환하는 메소드이다. (`(Self) -> ((R1) -> R2)`)

두 번째 인자는 `R1`이며 반환값은 `R2`이다. 

그리고 내부적으론 첫 번째 인자로 들어온 함수 `binder`에 자기 자신을 인자로 넣어 `binder` 함수를 호출한다. 그 결과의 반환값은 함수로(`R1 -> R2`) 그 함수에 우리는 두 번째 인자로 들어간 `R1`을 넣어준다. 그리고 그 메소드의 결과값은 `R2`이므로 `bind(to: )` 메소드의 반환값에 맞게 값을 반환해준다.

자 대충 파악이 끝났다. 이번에 살펴볼 부분은 바로 `tableView.rx.items(cellIdentifier: "Cell")` 이 부분이다. 역시 어떻게 작성되어 있는지 살펴보자. 

```swift
extension Reactive where Base: UITableView { 
    public func items<S: Sequence, Cell: UITableViewCell, O : ObservableType>
        (cellIdentifier: String, cellType: Cell.Type = Cell.self)
        -> (_ source: O)
        -> (_ configureCell: @escaping (Int, S.Iterator.Element, Cell) -> Void)
        -> Disposable
        where O.E == S {
        return { source in
            return { configureCell in
                let dataSource = RxTableViewReactiveArrayDataSourceSequenceWrapper<S> { (tv, i, item) in
                    let indexPath = IndexPath(item: i, section: 0)
                    let cell = tv.dequeueReusableCell(withIdentifier: cellIdentifier, for: indexPath) as! Cell
                    configureCell(i, item, cell)
                    return cell
                }
                return self.items(dataSource: dataSource)(source)
            }
        }
    }
}
```

음…🤔 당..당황스럽다..

일단 내부의 여러 코드들은 무시하고 그 외형에만 집중해보자. 먼저 `items(cellIdentifier:String, cellType: Cell.Type)`에선 세 개의 타입을 사용하는데 여기서 주목해볼 것은 `O`로 `ObservableType` 프로토콜을 따르는 타입이다. 

먼저 `items(cellIdentifier: "Cell")`을 호출하면 어떤 값이 반환되는지 보자.

바로 `(_ source: O) -> (_ configureCell: @escaping(Int, S.Iterator.Element Cell) -> Void) -> Disposable`의 외형을 갖는 **함수가 반환**된다. 이 함수는 `O` 타입의 인자를 받아 `(_ configureCell: @escaping(Int, S.Iterator.Element Cell) -> Void) -> Disposable`을 반환하는 함수의 형태를 갖는다.

자 이제 거의 다왔다!🤯

이제 위에서 살펴본 `bind(to:, curriedArgument:) -> R2`와 함께 살펴보자. 

첫 번째 인자는 `(Self) -> ((R1) -> R2)` 형태의 함수를 받는다 `Self`는 `ObservableType` 프로토콜을 따르는 타입을 말한다. 

그리고 `items(cellIdentifier: "Cell")`의 호출된 결과값은 함수로 그 형태는 `(_ source: O) -> (_ configureCell: @escaping(Int, S.Iterator.Element Cell) -> Void) -> Disposable`로 `bind(to: curriedArgument:) -> R2`의 첫 번째 인자로 들어갈 값의 외형과 동일하다!! 

왜냐하면 `O`를 `O : ObservableType`를 통해 한정해주었기 때문이다. 

> 이해했다고 생각하고 글을 작성하는 본인도 하나하나 따져보고 비교해가며 글을 작성하고 있다.😭

그렇다면 `R1`은  `@escaping(Int, S.Iterator.Element Cell) -> Void)`가 되고 `R2`는 `Disposable`이 된다. 그렇기 때문에 `bind(to:, curriedArgument:) -> R2`에서 두 번째 인자를 후행 클로저로 작성해줄 수 있는 것이다.

그럼 이제 테이블 뷰에 데이터를 바인딩하는 과정을 작성한 위의 코드가 어느 정도가 이해가 될 것이다.



