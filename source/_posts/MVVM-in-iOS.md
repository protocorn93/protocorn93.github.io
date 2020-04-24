---
title: MVVM in iOS - 번역
subtitle: Introduction
date: 2019-02-17 16:31:53
tags: ["MVVM", "Binding", "iOS"]
category: ["Architecture", "MVVM"]
---

> MVVM에 대해서 공부를 하면서 읽은 여러 글 중 개인적으로 괜찮았다고 느낀 글을 번역하였습니다. 
>
> 출처 : [MVVM in iOS](https://medium.com/@azamsharp/mvvm-in-ios-from-net-perspective-580eb7f4f129) by [Mohammad Azam](https://medium.com/@azamsharp)

관련된 자료

- [View Model Validation in iOS](https://medium.com/@azamsharp/validation-in-mvvm-for-ios-5a819be221c8)
- [View Model and Networking](https://medium.com/@azamsharp/mvvm-in-ios-viewmodel-and-networking-5bbe1d768c7f) 
- [Udemy Course - Mastering MVVM for iOS](https://www.udemy.com/mastering-mvvm-for-ios/?couponCode=ILOVEMVVM)

최근 들어 iOS에서의 MVVM과 이 아키텍처가 MVC(Massive View Controller)의 문제를 해결하는 방법에 대한 많은 이야기가 오가곤 한다. iOS를 만드는데 사용되는 아키텍처는 MVC(Model View Controller)이고 비록 MVC 패턴에는 아무 문제가 없더라도 대부분의 뷰 컨트롤러에 너무 많은 코드가 쌓이게 된다. 

MVVM은 어디선가 새로 튀어나온 개념이 아니다. 이는 특히 .NET 커뮤니티에서 오랜 세월 동안 사용되어왔다. 본인(저자)은 MVVM이 성공적으로 사용되었던 스케일이 큰 많은 WPF(Windows Presentation Foundation) 어플리케이션을 작업해왔다. 

WPF는 *XAML*과 "*[Dependency Properties](https://docs.microsoft.com/en-us/dotnet/framework/wpf/advanced/dependency-properties-overview)*" 를 활용해 양방향의 원활한 바인딩을 수행하기 때문에 iOS 프레임워크와 .NET WPF 프레임워크를 비교하는 것은 공평하지 않다. iOS에서 우리는 "*Dependency Properties*" 와 유사한 어떠한 개념도 존재하지 않고 심지어는 *XAML*과 같은 선언적 UI 코드도 존재하지 않는다. 

**MVVM 패턴의 이면에 존재하는 아이디어는 화면의 각 <u>뷰에 대한 데이터를 나타내는 뷰 모델(View Model)이 각 뷰(View)를 지원</u>한다는 것이다. **이는 만일 우리가 뉴스 어플리케이션을 만든다면 뷰 뉴스 아이템을 보여주는 `UITableView`로 구성되어 있을 것이고 뷰 모델은 제목, 설명, 출간 날짜, 저자 그리고 출처 등과 같이 뉴스를 표현해주는 데이터로 이루어져 있을 것이다.

샘플 뷰는 다음과 같이 기본 테이블 뷰 셀을 사용한는 간단한 테이블 뷰로 이루어져 있다. 

![](https://cdn-images-1.medium.com/max/1600/1*ZywEyTPYcrKEPyS8pkO-hg.png)

이제 이 뷰 위에 어떤 데이터를 보여주고 싶은지 생각해보자. 간단하게 제목과 설명을 보여주자. 그렇다면 **뷰 모델**은 제목과 설명을 제공하는 프로퍼티들로 이루어져 있을 것이다. 

```swift
struct ArticleViewModel {
    var title: String 
    var description: String
}

extension ArticleViewModel {
    init(article: Article) {
        self.title = article.title
        self.description = article.description
    }
}
```

이는 우리가 테이블 뷰에 보여주고 싶은 데이터를 나타내기는 하지만 이는 단지 테이블 뷰의 일부만을 나타낸다. (단일 테이블 뷰 셀) 뷰 모델은 완전한 뷰를 지원할 수 있어야 한다. 다음의 코드를 살펴보자. 

```swift
struct ArticleListViewModel {
    var title: String? = "Articles"
    var articles: [ArticleViewModel] = []
}

extension ArticleListViewModel {
    init(articles: [ArticleViewModel]) {
        self.articles = articles
    }
}
```

부모 격 뷰 모델인 **`ArticleListViewModel`**을 추가함으로써 우리는 화면에 나타나는 아이템들을 제어하기 위한 유연성을 만들어낸 것이다. 만일 추후에 Search bar를 추가한다면 단순히 `SearhViewModel` 안에 `ArticleListViewModel`의 레퍼런스를 추가해주면 된다. 



### View Models + Networking = ? 

------

iOS에서의 MVVM 구현에 대해 조사하는 동안 뷰 모델 안에 네트워킹 코드와 데이터 접근 코드를 구현한 많은 리소스를 발견하였다. 다음은 이에 대한 하나의 사례 코드이다. 

```swift
struct ArticleListViewModel {
    var title: String? = "Articles"
    var articles: [ArticleViewModel] = []
}

extension ArticleListViewModel {
    init(articles: [ArticleViewModel]) {
        self.articles = articles 
    }
    
    func loadArticles(callback: (([Article] -> Void)) {
        URLSession.shared.dataTask(with: url) { data, response, error in
            
            if let data = data {
                let json = try! JSONSerialization.jsonObject(with: data, options: [])
                articles = articleDictionaries.flatMap { Article(dictionary: $0) }
                
            }
            
            DispatchQueue.main.async {
                callback(articles)
            }
        }.resume()
    }
}
```

뷰 모델은 최대한 **멍청한(dumb)** 상태이어야 한다. 뷰 모델은 네트워킹 코드나 데이터 접근 코드를 포함해선 안된다. 뷰 모델 안에서 네트워크 작업을 수행한다면 뷰 모델 계층과 웹 서비스 계층 사이에 강한 커플링이 생기게 된다. 

> 몇몇의 독자들이 ***뷰 모델은 네트워킹 코드나 데이터 접근 코드를 포함해선 안된다***에 대해 지적을 해주었다. 심지어 iOS 커뮤니티에서도 각자 다른 취향의 MVVM을 사용하는 개발자들을 보았다. 각자의 상황에 맞게 MVVM 아키텍처를 선택해서 사용하면 될 것 같다. 

네크워킹와 데이터 접근 작업 수행은 뷰 모델과 독립적으로 존재해야 하고 분리된 클래스의 일부이거나 혹은 분리된 프레임워크/라이브러리의 일부가 되어야 한다. 당신의 어플리케이션에 따라 네트워킹과 데이터 접근 코드는 뷰 컨트롤러의 생성자에서 의존성 주입으로 전달될 수 있다. 

다음은 뷰 컨트롤러 내에서 웹 서비스 호출이 일어나지만 별개의 웹 서비스 클래스에 의해 제어되는 구현 코드이다. 

```swift
private func loadArticles() {
    let url URL(string: "url")
    
    Webservice().getArticles(url: url) { articles in
        print(articles)
    	
        let articles = articles.map { article in 
            return ArticleViewModel(article: article)
        }
        
        self.viewModel = ArticleListViewModel(articles: articles)
    }
}
```

웹 서비스 클래스는 뷰 모델 객체에 매핑되는 도메인 객체를 반환한다. 이러한 매핑은 직접 혹은 .NET의 [AutoMapper](http://automapper.org/)와 유사한 iOS 자동 매핑 툴을 사용하여 수행될 수 있다.

매핑 작업이 끝나면 뷰 모델을 할당하고 이는 테이블 뷰의 리로드로 이어진다. 이 작업은 프로퍼티의 `didSet` 행위에 의해 자동으로 이루어진다. `didSet` 행위는 클래스의 생성자 내부에서 일어나는 할당에는 실행되지 않는다는 것을 명심하자. 

```swift
private var viewModel: ArticleListViewModel = ArticleListViewModel() {
    didSet {
        self.tableView.reloadData()
    }
}
```



### MVVM Bindings 

------

**바인딩(Binding)**은 뷰와 뷰 모델 사이의 데이터의 흐름을 말한다. 회원가입 화면을 만들어야 하는 상황을 고려해보자. 텍스트 필드에 값을 입력하면 이는 `RegistrantionViewModel`에도 즉시 반영되어야 한다.  바인딩을 사용한다면 다음과 같은 코드를 입력할 필요가 없다. 

```swift
self.viewModel.title = self.titleTextField.text!
self.viewModel.description = self.descriptionTextField.text!
```

바인딩은 양방향 통신이다. 이 말은 뷰 모델의 프로퍼티를 변경하면 이 역시 뷰에 반영된다는 것을 의미한다. 위에서 언급했듯이 WPF 프레임워크에선 바인딩은 dependency properites에 의해 처리된다. 하지만 불행하게도 스위프트는 이러한 기능을 갖고 있지 않고 우리는 이러한 작업을 위해 추가적인 무언가를 해주어야 한다. 

문자열 타입에는 옵저버를 사용할 수 없기 때문에 우리는 우리만의 사용자 정의 클래스를 만들어줄 것이고 다른 객체가 자신을 관찰할 수 있도록 해야 한다.

```swift
import Foundation 

class Dynamic<T> {
    
    var bind: (T) -> Void = { _ in }
    
    var value: T? { 
        didSet {
            bind(value!)
        }
    }
    
    init(_ v: T) {
        value = v
    }
}
```

[Dynamic<T>](http://rasic.info/bindings-generics-swift-and-mvvm/)는 사용자 정의 클래스로 T 타입의 값을 갖고 있다. 만일 값이 변경되면 didSet은 바인드 함수를 호출하는데, 이 함수에는 변경된 값이 담기고 호출한 객체에게 이를 다시 전달한다. 그리고 이렇게 전달된 값을 통해 값이 변경되었을 때의 행위를 정의해줄 수 있는 것이다. 

```swift
class AddArticleViewController: UIViewController {
    
    @IBOutlet weak var titleTextField: BindingTextField!
    @IBOutlet weak var descriptionTextField: BindingTextField!
    
    var viewModel: AddArtcielsViewModel! {
        didSet {
            viewModel.title.bind = { [unowned self] in self.titleTextField.text = $0 }
            viewModel.description.bind { [unowned self] in self.descriptionTextField.text = $0 }
        }
    }
    
    override viewDidLoad() {
        super.viewDidLoad()
        self.viewModel = AddArticleViewModel()
    }
    
    @IBAction func addArticleButtonPressed(_ sender: Any) {
        self.viewModel.title.value = "hello world"
        self.viewModel.description.value = "description"
    }
}
```

위의 코드는 뷰 모델에서 UI 요소로의 단 반향 바인딩(*one-directional binding*)을 구축한다. 이는 즉 뷰 모델의 프로퍼티가 변경되면 UI 요소에도 반영된다는 것을 의미한다. 

양방향 바인딩(*bi-directional binding*)을 구축하기 위해선 사용자 정의 `UITextField`를 구현해야 한다. 

```swift
import UIKit 

class BindingTextField: UITextField {
    
    var textChanged: (String) -> Void = { _ in }
    
    func bind(callback: @escaping (String) -> Void) {
        self.textChanged = callback
        self.addTarget(self, action: #selector(textFieldDidChanged), for: editingChanged)
    }
    
    @objc func textFieldDidChanged(_ textField: UITextField) {
        self.textChanged(textField.text!)
    }
}
```

`UITextField`에 `.editingChanged` 이벤트에 대한 행위를 정의해주었고 이는 사용자 정의 콜백 함수를 호출한다. 이제 이를 사용해 `IBOutlets`을 새로 작성해보자. 

```swift
class AddArticleViewController: UIViewController {
    
    @IBOutlet weak var titleTextField: BindingTextField! {
        didSet {
            titleTextField.bind { self.viewModel.title.value = $0 }
        }
    }
    
    @IBOutlet weak var descriptionTextField: BindingTextField! {
        didSet {
            descriptionTextField.bind { self.viewModel.description.value = $0 }
        }
    }
    
    var viewModel: AddArticleViewModel! {
        // From View Model to View
    }
    
    ...
    
}
```

이 글은 단순히 MVVM을 소개하는 수준이다. 스위프트가 사용자 정의 속성을 만드는 기능을 허용하게 되면 iOS의 MVVM 접근 방식은 크게 향상될 것이다. 이를 통해 우리는 각 뷰 모델이 쉽게 뷰와 관련된 규칙을 검증하고 어긋난 규칙을 반환하는 동적 검증 프레임워크를 만들 수 있다. 

Michael Long이 댓글에서 언급한 것처럼 MVVM은 뷰 관련 로직의 테스트를 쉽게 해준다. 

뷰 컨트롤러의 비즈니스 로직을 뷰 모델로 옮기는 것의 또 다른 이점은 뷰 모델을 사용하면 어플리케이션의 해당 구성 요소에 대한 단위 테스트를 훨씬 쉽게 작성할 수 있다는 것이다.

