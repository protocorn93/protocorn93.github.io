---
title: 'init?(coder:), init(nibName:, bundle:), awakeFromNib()'
date: 2018-11-01 17:06:57
tags: ["iOS", "Swift"]
category: ["iOS"]
---

평소에 뷰 컨트롤러나 뷰를 만들기 위해 스토리보드뿐만 아니라 `.xib`나 코드의 형태로 만들어 사용하기도 한다. 하지만 이들 각각의 방법으로 뷰 컨트롤러를 생성할 때 생성되는 시점이나 불리는 메소드를 명확히 알지 못해 매번 작성한 코드를 이리저리 움직여가며 동작을 확인했다. 오늘은 이렇게 정확하기 알지 못했던 개념을 개념을 더욱 명확하게 하기 위해 공부하고 기록한 내용이다. 

------

#### xib? nib?

- **nib** - NeXT Interface Builder
- **xib** - XML Interface Builder

`.xib`와 `.nib`의 차이점은 무엇일까? 차이점이라기보단 `.xib` 파일은 빌드 시점에 `.nib` 파일의 형태로 바뀐다. 인터페이스 빌더로 작업한 UI는 `.xib` 파일의 형태로 저장되고 빌드 시점에 앱 번들로 복사되고 런타임에 로드된다. 또한 `.xib` 파일은 텍스트 기반의 파일로 바이너리 파일인 `.nib` 파일보다 소스 컨트롤에 용이하며 읽기에 용이하다.



#### init?(coder:)

`xib` 파일로 만든 뷰는 실제로 저장될 때 **아카이빙** 되어 저장된다. 그러므로 해당 뷰를 불러올 때는 **언아카이빙**를 통해 불러와야 한다. 스토리보드도 내부적으론 일종의 `xib` 파일들로 이루어져 있기 때문에 로그를 찍어보면 `init?(coder:)` 생성자가 불리는 것을 확인할 수 있다. 하지만 아직 이 시점에선 `@IBOutlet`이나 `@IBAction`은 준비되어 있지 않다. 



#### init(nibName:, bundle:)

> `nib` 파일명은 결국 우리가 생성해준 `xib` 파일이 바뀐 결과물이기 때문에 이 둘의 이름은 확장자를 제외하곤 동일하다.

`init(nibName:, bundle:)` 생성자는 코드로 `nib` 파일로 만들어진 뷰를 불러올 때 사용하는 생성자이다. 이는 본인도 적지 않게 사용해봤다. 사용법은 `nibName` 아규먼트에는 `nib `파일의 이름에 해당하는 값을 넣어주는데 본인은 언제나 두 아규먼트 모두에 `nil` 값을 할당해주었다. 그 이유가 궁금해서 찾아봤는데 공식 문서는 이를 다음과 같이 설명하고 있었다. 

[**nibName**](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621487-nibname)

이 프로퍼티는 `init(nibName:, bundle:)` 생성자에 의해 생성되는 시점에 지정된 값을 포함한다. 이 프로퍼티는 `nil`일 수 있다. 

뷰 컨트롤러의 뷰를 저장하기 위해 `nib` 파일을 사용한다면 뷰 컨트롤러를 생성하는 시점에 생성자에 해당 이름을 명시해주는 것을 추천한다. 하지만 만일 `nib` 파일 명을 지정해주지 않고 `loadView`를 오버라이딩하지 않는다면 뷰 컨트롤러는 `xib` 파일을 다른 방법을 이용해 이를 찾아낼 것이다. 구체적으로 말하자면 적절한 파일 이름 (`.nib` 확장자를 제외한)의 `nib`를 찾고 뷰를 필요로 할 때 이를 로드하여 사용할 것이다. **적절한** 이름을 찾는 과정은 다음의 순서를 따라 찾는다. 

1. 만일 뷰 컨트롤러 클래스의 이름이 `MyViewController`와 같이 `Controller` 끝난다면 `Controller`를 제외한 `MyView.nib` 파일을 찾을 것이다. 
2. 뷰 컨트롤러와 이름이 동일한 `nib` 파일을 찾는다. `MyViewController`라면 `MyViewController.nib`

위와 같은 이유로 생성 시점에 아규먼트를 넘겨줘야 한다면 다음과 같이 생성자를 정의해줄 수 있다.

```swift
class MyViewController: UIViewController {
    private var prop: Int
    
    init(prop: Int) {
        self.prop = prop
        super.init(nibBundle: nil, bundle: nil)
    }
    
    // 사용자 정의 생성자를 작성해주었기 때문에 반드시 required 생성자를 구현해주어야 한다.
    required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
    }
}
```



#### awakFromNib()

사실 `awakeFromNib()`는 생성자가 아니다. 근데 생각보다 이를 생성자라고 생각하고 사용하는 경우가 적지 않은데 이를 잘못 이해한다면 원하는 결과물을 얻을 수 없을 것이다. 그럼 `awakeFromNib`은 무엇일까?

이 메소드는 `init?(coder:)`를 통해 뷰가 모두 언아카이빙된 후 호출된다. `@IBOulet`과 `@IBAction`이 모두 자리가 잡힌 후 호출되는 것이다. `init?(coder:)`가 언아카이빙의 시작점이라면 `awakeFromNib()`은 끝나는 시점이라고 할 수 있다. 

이 세 메소드들을 공부하면서 각각 로그를 찍어보았는데 한곳에서 알 수 없는 현상이 발생하였다. 이는 다음과 같은 상황이었다. `xib` 파일을 하나 만들고 `UIViewController`를 상속받는 뷰 컨트롤러를 생성하였고 이를 `xib` 파일의 `File's Owner`로 지정하였다. 그리고 뷰 컨트롤러의 `awakeFromNib()` 메소드에 로그를 출력하도록 코드를 추가해보았지만 `awakeFromNib()`은 호출되지 않았다. 

그 이유를 찾아본 결과 `File's Owner`는 이렇게 언아카이빙 되는 뷰 객체와 관련성이 없다. 위에서 언급했듯이 `awakeFromNib()` 언아카이빙이 모두 끝난 후 호출되는 메소드이다. `File's Owner`는 언아카이빙이 시작되기 전에 이미 존재하고 언아카이빙이 모두 끝난 후 해당 객체와 연결되기 때문에 `File's Owner`인 뷰 컨트롤러 안에서의 `awakeFromNib()`의 호출은 무의미하다. 

하지만 스토리보드와 뷰 컨트롤러에선 얘기가 좀 달라진다. 스토리보드에 뷰 컨트롤러를 올리고 로그를 찍어보면 정상적으로 찍히는 것을 확인할 수 있다. 

```
ViewController - init(coder:)
ViewController - awakeFromNib
```

그 이유는 해당 뷰 컨트롤러 객체 클래스를 `File's Owner`가 아닌 해당 뷰 컨트롤러 자체이기 때문이다. 그렇기 때문에 스토리보드 위에 올라가있는 뷰 컨트롤러는 언아키이빙 과정을 거치기 때문에 정상적으로 `awakeFromNib()`이 호출되는 것을 확인할 수 있다.



------

참고자료

1. [iOS: initWithCoder:, initWithNibName:bundle:, awakeFromNib, loadView](http://suho.berlin/engineering/ios/ios-initwithcoder-initwithnibnamebundle-awakefromnib-loadview/)
2. [What's up with the .NIB -> .XIB?](https://stackoverflow.com/questions/1068191/whats-up-with-the-nib-xib)

