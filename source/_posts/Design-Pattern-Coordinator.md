---
title: Design Pattern - Coordinator
date: 2019-01-05 21:19:49
tags: ["iOS", "Swift", "Coordinator", "Design Pattern"]
category: ["iOS", "Design Pattern"]
---

오늘은 **Coordinator(코디네이터)** 패턴에 대해 공부를 해보았다. 기존에 뷰 컨트롤러간의 전환에서는 `segue`를 연결하여 `perform` 과 `prepare` 메소드를 사용하거나 직접 뷰 컨트롤러를 생성해주고 `UINavigationController` 를 통해 `push` 해주거나 현재 뷰 컨트롤러에서 `push`를 해주었다.

하지만 코디네이터 패턴을 우연치 않은 기회에 접하게 되었고, 관련 글을 읽다보니 기존의 방법이 몇 가지 단점을 갖고 있다는 것을 발견하였다.

뷰 컨트롤러는 그들이 앱 내에서 앱의 플로우 내에서 자신의 위치에 대해 인식하지 못하거나, 심지어는 그들이 앱 플로우의 시작점이란 것 조차 인식하지 못하고 독립적으로 존재할 때 가장 좋다. 이는 작성한 코드를 테스트할 때 좋을 뿐만 아니라 뷰 컨트롤러의 재활용을 용이하게 해준다.

```swift
if let vc = storyboard?.instantiateViewController(withIdentifier: "SomeVC") {
    navigationController?.pushViewController(vc, animated: true)
}
```

> 기존에 뷰 컨트롤러를 `push` 해줄 때 자주 볼 수 있는 형태의 코드다.

위와 같은 코드에서 하나의 뷰 컨트롤러는 반드시 다른 뷰 컨트롤러에 대해 알고 있어야 하며 만들고 설정하고 이를 띄어주어야 한다. 이는 앱 내에서 강한 커플링을 만든다. 또한 위와 같이 다른 뷰 컨트롤러의 전환 코드를 하드 코딩한다면 같은 뷰 컨트롤러 내에서 또 다른 곳으로 전환을 원한다면 중복 코드가 발생하게 될 것이다. 

코디네이터 패턴은 뷰 컨트롤러 간의 연결을 느슨하게 해줄 것이고 그러므로 각각의 뷰 컨트롤러는 어떤 뷰 컨트롤러가 이전이었는지 혹은 다음에 올지 알지 못하고 심지어는 이러한 뷰 컨트롤러 전환 체인이 존재하는지도 모를 것이다. 대신 앱의 흐름은 `Coordinator`를 사용해 통제되며 뷰 컨트롤러는 오직 `Coordinator` 객체와만 대화를 하게 될 것이다. 

결과로는 어떤 뷰 컨트롤러를 어떤 순서로도 사용할 수 있으며 필요에 따라 재사용할 수 있게 된다. 

앱이 커진다면 자식 코디네이터(서브 코디네이터)를 사용하여 앱의 내비게이션 파트를 쪼갤 수 있다. 예를 들어 *계정 생성* 흐름과 *제품 구매* 흐름을 서로 다른 코디네이터를 두어 사용할 수 있는 것이다. 만일 더 나은 유연성(*flexibility*)를 원한다면 뷰 컨트롤러가 코디네이터를 직접 갖고 있는 것보단 프로토콜로 참조하는 방법으로 더 나은 유연성을 부여할 수 있다.

코드로 이를 살펴보도록 하자.

```swift
protocol Coordinatable {
    func activate()
}
```

코디네이터 객체가 따라야 할 프로토콜의 기본 모습은 위와 같다. 먼저 어플리케이션 시작을 담당하는 `ApplicationCoordinator` 객체를 만들어보자. 

```swift
class ApplicationCoordinator: Coordinatble {
    private var navigationController: UINavigationController
    private var mainCoordinator: Coordinatable
    init(_ navigationController: UINavigationController) {
        self.navigationController = navigationController
		self.mainCoordinator = MainCoordinator(navigationController) // --- 1
    }
    
    func activate() {
        mainCoordinator.activate()
    }
}

class MainCoordinator: Coordinatable {
    private var navigationController: UINavigationController
    
    init(_ navigationController: UINavigationController) {
        self.navigationController = navigationController
    }
    
    func activate() {
        let mainViewController = MainViewController.instantiate() // --- 2
        navigationController.push(mainViewController, animated: true)
    }
}
```

1. `ApplicationCoordinator`는 `MainCoordinator`를 조정해주는 역할이라고 보면된다. 이 과정에서 뷰 컨트롤러에 필요한 데이터를 넣어줄 수 있다.
2. 실제로 `MainViewController`를 띄우는 것은 `MainCoordinator` 에서 진행되고 `.instantiate()` 메소드는 뷰를 어떻게 구성하느냐(스토리보드 or 코드 or xib)에 따라 달라질 것이다.

이를 앱 델리게이트에서 사용해보자. 

>  우리는 지금 앱 시작을 위한 코디네이터 객체를 만들었기 때문에 프로젝트 설정의 타겟(`Targets`)에서 `Main Interface` 부분을 빈 칸으로 만들어야 한다.

```swift
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?
    var coordinator: Coordinatable?
    
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
        let navigationController = UINavigationController()
        
        coordinator = MainCoordinator(navigationController)
        coordinator?.activate()
        
        window = UIWindow()
        window?.rootViewController = navigationController
        window?.makeKeyAndVisible()
        
        return true
    }
}
```

프로젝트를 실행시켜보면 첫 뷰 컨트롤러(`MainViewController`)가 정상적으로 보이는 것을 확인할 수 있다. 그리고 `MainViewController`에서 계정 생성 흐름과 제품 구매 흐름을 두 개의 코디네이터로 진행해주고 싶다면 우리는 다음과 같이 코드를 작성해볼 수 있다. 

```swift
protocol MainViewFlowDelegate: class {
    func signUp()
    func buyProduct()
}

class MainViewController: UIViewController {
    weak var delegate: MainViewFlowDelegate?
    
    override func viewDidLoad() {
        super.viewDidLoad()
    }
    
    @IBAction func handleSignUp(_ sender: UIButton) {
        delegate?.signUp()
    }
    
    @IBAction func handleBuyProduct(_ sender: UIButton) {
        delegate?.buyProduct()
    }
}
```

위와 같이 새로 시작하는 흐름을 프로토콜 메소드와 델리게이트 패턴으로 구현해주었다. 다음은 `MainViewFlowDelegate` 프로토콜을 준수하고 있는 `MainCoordinator`의 대략적인 모습이라고 볼 수 있다. 

```swift
class MainCoordinator: Coordinatable {
    private var navigationController: UINavigationController
    
    init(_ navigationController: UINavigationController) {
        self.navigationController = navigationController
    }
    
    func activate() {
        let mainViewController = MainViewController.instantiate()
        mainViewController.delegate = self // --- 1
        navigationController.push(mainViewController, animated: true)
    }
}

extension MainCoordinator: MainViewFlowDelegate {
    func singUp() {
        let singUpCoordinator = SingUpCoordinate(navigationController)
        singUpCoordinate.activate()
    }
    
    func buyProduct() {
        let buyProductCoordinator = SingUpCoordinate(navigationController)
        buyProductCoordinator.activate()    
    }
}
```

1. `activate`를 통해 `MainViewController`를 생성하고 `push`를 할 때 `delegate`를 지정해준다.

위와 같이 코디네이터로 뷰 컨트롤러의 전환을 하게 되면 뷰 컨트롤러에서 목적지 뷰 컨트롤러에 대한 코드를 작성해 줄 필요도 없으며 직접 `UINavigationController`에 요청을 하지 않아도 된다. 즉 커플링이 느슨해지고 뷰 컨트롤런는 전환에 대해선 코디네이터를 제외하곤 어떠한 코드도 갖고 있지 않게 된다. 또한 코디네이터 역시 하나의 콘크리트 타입으로 참조하고 있는 것이 아닌 프로토콜을 참조하고 있기 때문에 좀 더 유연하다고 볼 수 있다.

---

**출처**

- [How to use Coordinator Pattern in iOS apps - Hacking with Swift](https://www.hackingwithswift.com/articles/71/how-to-use-the-coordinator-pattern-in-ios-apps)
- [Coordinator Tutorial for iOS : Getting Started - Raywenderlich](https://www.raywenderlich.com/158-coordinator-tutorial-for-ios-getting-started)