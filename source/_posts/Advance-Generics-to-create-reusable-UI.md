---
title: Advance Generics
subtitle: Advance Generics to create reusable UI
date: 2018-09-14 03:49:45
tags: ["Swift","iOS","Generics"]
category: ["iOS","번역"]
---

제네릭은 스위프트의 강력한 기능 중 하나이며 스위프트 스탠다드 라이브러리 대부분의 코드는 제네릭 코드로 이루져 있어 이는 우리로 하여금 클린하고 재사용성이 높은 코드를 작성하게끔 해준다. 제네릭의 로직을 이해하는 것은 제네릭의 효과를 이해하는 것보다 덜 직관적이므로 이 포스팅에서는 제네릭의 로직을 자세하게 설명하기 위한 의도로 작성되지 않았다. 그래야 이 글을 읽는 당신이 보다 쉽게 당신의 앱에서 제네릭을 사용하는 방법을 익힐 수 있을 것이기 때문이다. 그렇기 때문에 어느 타입(type)에서도 재사용할 수 있는 제네릭 기본 구현 코드를 공유하고자 한다. 

우리는 제네릭과 프로토콜의 다형성을 활용하여 기본 검색 컨트롤러를 구현할 것이고 이를 통해 당신은 다신의 코드에서 중복된 코드를 제거하는 것이 얼마나 쉬운지에 대해 확인할 수 있을 것이다. 먼저 프로토콜을 사용하여 뷰 컨트롤러에서 재사용 식별자를 문자열로 다루는 행위를 피하는 것부터 시작해보자. (*테이블 뷰의 셀 식별자와 같이 재사용을 목적으로 사용되는 식별자를 문자열로 작성하는 것은 좋은 방법은 아니다.*) `Reusable` 스위프트 파일을 하나 생성하고 다음과 같이 코드를 작성해보자.

```swift
protocol Reusable {}

extension Reusable where Self: UITableViewCell {
    static var reuseIdentifier: String {
        return String(describing: self)
    }
}

extension UITableViewCell: Reusable {}

extension UITableView {
    
    func register<T: UITableViewCell>(_:T.Type) {
        register(T.self, forCellReuseIdentifier: T.reuseIdentifier)
    }
    
    func dequeueReusableCell<T: UITableViewCell>(for indexPath: IndexPath) -> T {
        guard let cell = dequeueReusableCell(withIdentifier: T.reuseIdentifier) as? T else {
            fatalError("Could not deque cell with identifier")
        }
        
        return cell
    }
}
```

이 코드가 셀의 id 값을 문자열로 사용하는 것을 방지하도록 도와줄 것입니다. 이렇게 첫 번째 프로토콜이 끝났다. UI를 위한 코드를 작성해보도록 하자. 먼저 우리는 제네릭 타입 프로퍼티를 갖는 셀을 만들 것이다. `BaseTableViewCell` 스위프트 파일을 하나 생성하고 다음과 같이 코드를 작성해보자

```swift
class BaseTableViewCell<V>: UITableViewCell {
    
    var item:V!
    
}
```

이러한 제네릭 문법에 익숙하지 않다면 먼저 이 [글](https://www.raywenderlich.com/154371/swift-generics-tutorial-getting-started)을 읽어보고 진해하는 것을 추천한다. 당신은 `UITableViewCell`을 상속받고 제네릭 타입 프로퍼티를 갖고 있는 것을 확인할 수 있다. 우린 이제 제네릭 데이터 소스 객체를 생성할 것이다. `GenericTableViewDataSource` 스위프트 파일을 생성하고 다음과 같이 코드를 작성해보자.

```swift
final class GenericTableViewDataSource<V, T: Searchable>: NSObject, UITableViewDataSource where V: BaseTableViewCell<T> {
    
    typealias CellConfiguration = (V,T) -> V
    
    // 1
    private var models: [T]
    private let configureCell: CellConfiguration
    private var searchResults: [T] = []
    private var isSearchActive: Bool = false
    
    init(models: [T], configureCell: @escaping CellConfiguration) {
        self.models = models
        self.configureCell = configureCell
    }
    
    // 2
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return isSearchActive ? searchResults.count : models.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell: V = tableView.dequeueReusableCell(for: indexPath)
        let model = getModel(at: indexPath)
        return configureCell(cell, model)
    }
    
    private func getModel(at indexPath: IndexPath) -> T {
        return isSearchActive ? searchResults[indexPath.item] :  models[indexPath.item]
    }
    
    // 3 External function for Searching
    func search(for query: String) {
        isSearchActive = !query.isEmpty
        searchResults = models.filter {
            let queryToFind = $0.query.range(of: query, options: NSString.CompareOptions.caseInsensitive)
            return (queryToFind != nil)
        }
    }
}
```

이를 좀 더 자세히 살펴보자. 여기 `UITableViewDatasSource` 프로토콜을 따르는 제네릭 서브 클래스가 있다. 이는 `V`와 `T`,  두 제네릭 타입의 제약을 갖고 있다. `V`는 단순히 `BaseTableViewCell`의 서브클래스를 의미하며 `T`는 `Searchable` 프로토콜을 따르는 객체를 의미한다. (`Searchable` 프로토콜을 밑에서 만들 갓이다.) 하지만 여기서 본인이 강조하고 싶은 것은 이를 사용한다면 `BaseTableViewSearchController` 위에서 보여지는 어떠한 타입의 모델이든 쿼리를 기반으로 검색 행위를 수행할 수 잇다는 사실이다. 좀 더 자세히 살펴보자. 

1. 테이블 뷰 위에서 보여질 객체 프로퍼티
2. `UITableViewDataSource` 프로토콜 메소드
3. 검색을 도와주는 함수로 앱에서 사용되는 모델이 `Person` 이라면 이들의 이름을 기반으로 검색을 가능케 하는 함수.

이제 위에서 언급한 `Searchable` 프로토콜을 작성해보자. 마찬가지로 `Searchable` 스위프트 파일을 하나 생성하고 다음과 같이 작성해보자.

```swift
protocol Searchable {
    var query: String { get }
}
```

이 프로토콜은 `getter` 프로퍼티만을 갖고 이 프로토콜을 따르게 되는 모든 모델은 해당 모델 객체가 검색되기 위한 쿼리를 이 곳에 작성해주어야 한다. 그리고 위에서 작성한 `search(for:)` 함수는 이 쿼리를  기반으로 모델들을 필터링한다. 

마지막으로 `BaseTableViewSearchController` 파일을 생성하고 다음과 같이 코드를 작성해보자.

```swift
class BaseTableViewSearchController<T: BaseTableViewCell<V>, V>: UITableViewController, UISearchBarDelegate where V: Searchable {
    
    // 1
    private var dataSource: GenericTableViewDataSource<T, V>?
    private let searchController = UISearchController(searchResultsController: nil)
    
    // 2
    var models: [V] = [] {
        didSet {
            DispatchQueue.main.async {
                self.dataSource = GenericTableViewDataSource(models: self.models, configureCell: { cell, model in
                    cell.item = model
                    return cell
                })
                
                self.tableView.dataSource = self.dataSource
                self.tableView.reloadData()
            }
        }
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        tableView.register(T.self)
        setUpSearchBar()
    }
    
    // 3
    private func setUpSearchBar() {
        navigationItem.searchController = searchController
        navigationItem.hidesSearchBarWhenScrolling = false
        searchController.dimsBackgroundDuringPresentation = false
        searchController.searchBar.delegate = self
    }
    
    // 4
    func searchBar(_ searchBar: UISearchBar, textDidChange searchText: String) {
        dataSource?.search(for: searchText)
        tableView.reloadData()
    }
}
```

하나씩 살펴보자. 먼저 제네릭을 사용한 타입의 제약이 들어간 컨트롤러를 확인할 수 있다. 그렇다면 여기서 `V`와 `T`는 무엇일까? 위에서와 마찬가지로 `BaseTableViewCell`과 `Searchable`을 따르고 있는 모델들에 불과하다. 좀 더 자세히 살펴보자.

1. 데이터 소스 객체는 프로퍼티로써 선언되어져야 한다. 
2. `public` 프로퍼티 모델에 값을 할당하면 제네릭 데이터 소스 객체가 초기화하고 이를 테이블 뷰 데이터 소스에 할당한다.
3. 검색 바의 UI 구성
4. `UISearchBarDelegate` 메소드로 제네릭 데이터 소스의 검색 함수를 호출하고 검색 결과를 바탕으로 테이블 뷰를 갱신한다. 

이게 전부다. 이 모든 코드를 이해하는 것은 어려울 것이다. 하지만 이를 몇 번 읽어보면 금방 자연스럽게 코드를 이해할 수 있을 것이다. 그럼 이를 예제에 적용시켜 보자. 우리의 예제에서는 축구 선수들을 검색할 수 있도록 `SoccerPlayer` 모델을 생성할 것이다. 

```swift
struct SoccerPlayer: Searchable {
    var name: String
    var query: String {
        return name
    }
}
```

이 모델 구조체는 이름 프로퍼티를 갖고 있으며 `Searchable` 프로토콜을 따르고 있기 때문에 이름을 반환해주는 `getter` 프로퍼티를 갖고 있다. 물론 이 프로퍼티에는 구조체의 프로퍼티 구성에 따라 주소나 성(*Last name*)과 같은 속성이 들어갈 수 있다. 즉 검색을 위해 사용되어야 하는 프로터리를 구성하는 것이다. 

그럼 이제 `BaseTableViewCell`을 상속받는 `SoccerCell`을 만들어보자.

```swift
class SoccerCell: BaseTableViewCell<SoccerPlayer> {

    override var item: SoccerPlayer? {
        didSet {
            textLabel?.text = item?.name
        }
    }

}
```

이 제네릭 셀은 `SoccerPlayer`를 제네릭 객체 타입으로 갖고 있다. 그리고 이 제네릭 셀을 제네릭 검색 컨트롤러에서 사용해보자. 이미 구현되어 있는 코드로 인해 코드 양은 놀라울 정도로 적을 수 있다. 이를 위해 마지막으로 `ListTableViewController` 파일을 생서하고 다음과 같이 작성해보자. 

```swift
// 1
class ListTableViewController: BaseTableViewSearchController<SoccerCell, SoccerPlayer> {

    override func viewDidLoad() {
        super.viewDidLoad()
        
        // 2
        let soccerPlayers = [
            SoccerPlayer(name: "Messi"),
            SoccerPlayer(name: "Ronaldo"),
            SoccerPlayer(name: "Modric"),
            SoccerPlayer(name: "Guerrero"),
            SoccerPlayer(name: "Rodriguez"),
            SoccerPlayer(name: "Kane"),
            SoccerPlayer(name: "Ramos"),
            SoccerPlayer(name: "Pique"),
            SoccerPlayer(name: "Mbape"),
            SoccerPlayer(name: "Pogba"),
            SoccerPlayer(name: "Zidane"),
            SoccerPlayer(name: "Kross"),
            SoccerPlayer(name: "Puyol"),
            SoccerPlayer(name: "Beckham")
        ]
        
        // 3
        self.models = soccerPlayers
    }
}
```

이것이 전부다. 하나씩 설명하자면

1. 제네릭 객체를 상속받을 때 해주어야 할 것은 제네릭 코드가 타입을 추론할 수 있도록 타입을 선언해주는 것 뿐이다. `T`는 `SoccerCell`로 `V`는 `SoccerPlayer`로 대체되는 것을 확인 할 수 있다. 
2. 더미 데이터로 데이터 소스를 구성하는데 사용된다. 서버로부터 데이터를 받아올 수도 있다.

앱을 실행시키고 원하는 선수 이름을 검색하면 해당하는 선수가 출력되는 것을 확인할 수 있을 것이다. 

이 포스팅이 제네릭의 힘과 프로토콜을 이용해 클린하고 재사용성이 좋은 코드를 작성하는 방법을 이해하는데 도움이 되었기를 희망한다.

완성된 예제 프로젝트는 이 링크를 [참고](https://github.com/jamesrochabrun/GenericBaseContollers)하고 이 포스팅을 작성하는데 영감을 받은 제네릭에 대한 설명이 담긴 훌륭한 글은 이 [링크](https://www.youtube.com/watch?v=K0IO4QvFxnM)를 참고하길 바란다.

---

출처 : [Medium - Advance Generics to create reusable UI](https://medium.com/@jamesrochabrun/advance-generics-to-create-reusable-ui-f0b8b8934895)