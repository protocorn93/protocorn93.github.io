---
title: Delegation in Swift
date: 2019-02-02 13:54:06
tags: ["Delegate", "Closure", "Design Pattern"]
category: ["Design Pattern"]
---

>SwiftBySundell의 [Delegation in Swift](https://www.swiftbysundell.com/posts/delegation-in-swift)를 공부하고 정리한 내용입니다. 

**Delegate Pattern**

델리게이트 패턴은 애플 플랫폼에서 오랜 기간동안 중요한 역할을 해왔다. 델리게이션은 테이블 뷰의 이벤트를 다루는 `UITableViewDelegate`부터 캐시 동작을 수정하는 `NSCacheDelegate`까지 모든 곳에서 사용되었다. 델리게이트 패턴의 주요 목적은 결함도를 낮추는 방법으로 객체의 소유주와 객체가 통신할 수 있도록 해주는 것이다. 객체가 자신의 소유주의 실제 타입을 알게 하는 것이 아니기 때문에 보다 유지하기 쉽고 재사용하기 쉬운 코드를 작성할 수 있다. 

객체의 동작이나 결정들을 객체의 소유주에 위임하는(*delegating*) 행위의 이점은 하나의 객체가 그들 스스로가 모든 use case를 담는 거대한 객체가 되지 않고 보다 다양한 use case를 지원할 수 있다는 것이다. 

`UITableView` 혹은 `UICollectionView`는 그들이 무엇을 어떻게 보여주냐에 따라 굉장히 다양한 모습을 가질 수 있다. 델리게이션을 통해 우리는 손쉽게 이벤트들을 다룰 수 있고 셀들이 어떻게 만들어지는지, 또한 레이아웃 속성들을 정의해줄 수 있다. 

델리게이션은 많은 상황에서 재사용되어야 하고 모든 상황에서 자신의 소유주 객체를 명확히 알고 있는 타입에 효과적이다. (`UITableView`가 뷰 컨트롤러나 컨테이너 뷰에 속해있는 것을 생각해보자.) *Observable*한 타입과 반대로 델리게이션은 오직 하나의 소유주 객체와만 통신하고 이와 1:1 관계를 맺는다. 

애플의 API에서 델리게이션을 구현하는 가장 흔한 방법은 바로 델리게이션 프로토콜을 사용하는 것이다. `UITableViewDelegate` 프로토콜과 같이 우리는 우리만의 델리게이션 프로토콜을 다음과 같이 정의할 수 있다. 

```swift
protocol FileImporterDelegate: AnyObject {
    func fileImporter(_ importer: FileImporter,
                      shouldImportFile file: File) -> Bool

    func fileImporter(_ importer: FileImporter,
                      didAbortWithError error: Error)

    func fileImporterDidFinish(_ importer: FileImporter)
}

class FileImporter {
    weak var delegate: FileImporterDelegate?
}
```

델리게이션 프로토콜을 정의할 때는 이름을 짓는 방법은 애플 그들의 방식을 이용해 짓는 것이 바람직하다. 

1. 메소드가 델리게이트 메소드라는 것을 명확히 하기 위해 메소드의 이름을 행위를 위임하는 객체의 이름을 사용하는 것이 가장 흔한 방법이다. (`UITableView`의 경우 자신의 행위나 결정을 위임하는 메소드들(`UITableViewDelegate`)의 이름이 `tableView`인 것을 확인할 수 있다. )
2. 델리게이션 메소드의 첫 번째 인자는 행위를 위임하는 객체 자신이 되는 것이 이상적이다. 여러 객체들을 소유하고 있는 객체(*뷰 컨트롤러 or 컨테이너 뷰*)가 이벤트를 처리할 때 그들과 행위를 정의해주어야 하는 객체의 구분을 쉽게 해줄 수 있기 때문이다. 
3. 행위를 위임할 때 중요한 것은 행위의 대리자에게 세부 구현 사항을 노출시켜선 안된다는 것이다.

**Advantage**

프로토콜에 기반한 방법의 이점이라면 스위프트 개발자에게 익숙한 방법이라는 것이다. 또한 타입(예제의 `FileImporter` 혹은 `UITableView`)이 내보낼 수 있는 모든 이벤트를 단일 프로토콜로 묶어주고 이들 중 하나라도 제대로 구현하지 않는다면 컴파일러는 우리에게 에러를 발생시킬 것이다. 

**Downsides**

**델리게이트 프로토콜을 사용하는 것은 애매모호한 상태의 원인이 될 수 있다.**

`FileImporter` 예제에서 주어진 파일을 가져올지의 여부를 결저앟는 행위를 델리게이트에게 어떻게 위임하는지 보자. 델리게이트는 옵셔널이기 때문에 델리게이트가 없다면 이를 결정하는 행위가 애매해질 수 있다. 

```swift
class FileImporter {
    weak var delegate: FileImporterDelegate?

    private func processFileIfNeeded(_ file: File) {
        guard let delegate = delegate else {
            // 델리게이트가 nil이라면 어떠한 행위를 해주어야 할까?
            // 1. assertionFailure()
            // 2. using Default value
            return
        }

        let shouldImport = delegate.fileImporter(self, shouldImportFile: file)

        guard shouldImport else {
            return
        }

        process(file)
    }
}
```

<br>

**Closures**

좀 더 예측 가능한 상태를 만드는 방법은 델리게이트 프로토콜의 decision-making 부분을 클로저로 대체하는 것이다. 

그러면 우리가 만든 API의 사용자는 미리 가져올 파일을 결정하는데 사용되는 로직을 명시해주어야 하고 이는  `FileImporter` 로직의 애매모호함을 없애줄 수 있다.

```swift
class FileImporter {
    weak var delegate: FileImporterDelegate?
    private let predicate: (File) -> Bool

    // Specify the logic 
    init(predicate: @escaping (File) -> Bool) {
        self.predicate = predicate
    }

    private func processFileIfNeeded(_ file: File) {
        let shouldImport = predicate(file)

        guard shouldImport else {
            return
        }

        process(file)
    }
}
```

델리게이트 프로토콜의 모습은 다음과 같아질 것이다. 

```swift
protocol FileImporterDelegate: AnyObject {
    func fileImporter(_ importer: FileImporter,
                      didAbortWithError error: Error)

    func fileImporterDidFinish(_ importer: FileImporter)
}
```

위에서 사용한 방법의 주요 이점은 `FileImporter` 클래스를 잘못 사용하기 어려워졌다는 것이고 이제 델리게이트를 할당하지 않아도 객체가 유효하다는 것이다. 

<br>

**Configuration types**

계속해서 이렇게 클로저를 더해가는 것은 나쁘지 않은 방법이지만 이것이 반복된다면 API는 점점 더 혼잡해질 것이다. (설정을 위한 옵션과 다른 프로터티들의 구분이 힘들어 진다.) 

이러한 문제를 해결하기 위해 *configuration type*을 사용할 수 있다. 이를 통해 우리는 델리게이트 프로토콜에서 그랬던 것 처럼 이벤트들을 묶을 수 있고 여전히 다양한 이벤트들을 구현하는데 있어서 자유로울 수 있다. configuration type에 구조체를 사용하여 각 이벤트에 해당하는 프로퍼티들을 추가주었다.

```swift
struct FileImporterConfiguration {
    var predicate: (File) -> Bool
    var errorHandler: (Error) -> Void
    var completionHandler: () -> Void
}
```

이벤트들을 configuration type의 프로퍼티로 할당했기 때문에 쉽게 접근할 수 있고 `FileImporter`는 하나의 configuration type만을 갖고 있으면 되기 때문에 아래와 같이 보다 깔끔한 API를 유지할 수 있다. 

```swift
class FileImporter {
    private let configuration: FileImporterConfiguration

    init(configuration: FileImporterConfiguration) {
        self.configuration = configuration
    }

    private func processFileIfNeeded(_ file: File) {
        let shouldImport = configuration.predicate(file)

        guard shouldImport else {
            return
        }

        process(file)
    }

    private func handle(_ error: Error) {
        configuration.errorHandler(error)
    }

    private func importDidFinish() {
        configuration.completionHandler()
    }
}
```

위와 같은 방법으로 우리는 다양한 설정(configuration type)들을 사용해 쉽게 다양한 상황에 대응할 수 있다. 예를들어 `convenience` 생성자를 추가하여 단순한 모습의 `FileImporter`를 만들 수 있다.

```swift
extension FileImporterConfiguration {
    init(predicate: @escaping (File) -> Bool) {
        self.predicate = predicate
        errorHandler = { _ in }
        completionHandler = {}
    }
}
```

<br>

**Conclusion**

델리게이트 프로토콜를 사용하면 다양한 use case의 기본값인 친숙하고 고정된 패턴을 제공할 수 있다.  클로저는 이에 유연성(*flexibility*)를 더하지만 좀 더 복잡한 코드로 이어질 수 있다.

Configuration type은 중간 매개체를 제공하지만 역시 약간의 코드를 좀 더 필요로 한다.