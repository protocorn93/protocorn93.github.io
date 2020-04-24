---
title: 'convert(_:, to:), convert(_:, from:)'
date: 2019-01-13 09:29:03
tags: ["UIView", "iOS", "frame", "bounds", "convert"]
cateogory: ["iOS"]
---

오늘은 뷰를 보다 동적으로 다룰 때 적지 않게 사용하는 `UView`의 메소드인 `convert(_:, to:)`와 `convert(_:, from:)`에 대해 알아보자. 이 둘의 사용법은 그렇게 어려운 것은 아니지만 좌표계에 대한 이해가 부족하다면 직접 그려보거나 머리속으로 적지 않게 생각을 해야 한다. 본인도 좌표계에 대한 이해가 충분하다고 생각을 하였으나 생각보다 익숙해지는 데는 적지 않은 노력을 했던 것 같다. 

그럼 간단히 두 메소들의 정의를 살펴보자.

[`convert(_:, to:)`](https://developer.apple.com/documentation/uikit/uiview/1622442-convert)

- *Converts a point from the receiver’s coordinate system to that of the specified view.*
  - 메소드를 호출하는 뷰의 좌표계 상의 점을 특정 뷰의 좌표계로 변환

[`convert(_:, from:)`](https://developer.apple.com/documentation/uikit/uiview/1622424-convert)

- *Converts a point from the coordinate system of a given view to that of the receiver*
  - 주어진 뷰의 좌표계 상의 점을 메소드를 호출한 뷰의 좌표계로 변환

앞뒤만 다를 뿐 결국 같은 의미이다. 실제로 다음은 같은 결과를 반환한다. 

```swift
a.convert(a.specificPoint, to: b)
b.convert(a.specificPoint, from: a)
```

보다 다양한 관점에서 다양한 방법을 제공해준 것일 뿐이다. 두 메소드 모두 `to`나 `from`에 할당되는 값이 `nil`이라면 모두 윈도우 좌표계를 기반으로 한다.

> 사용하는 점(뷰)와 변환을 위한 기준 좌표계가 무엇이냐에 따라 사용하기 쉬운(사용을 위해 이해하기 쉬운) 메소드들은 그때마다 다른 것 같다. 

두 가지 예를 이용해 두 메소드를 이해해보자.

---

### Copy

화면 전환에 있어서 우리는 전환 과정이 진행되는 동안 사용자에게 눈속임(?)을 하기 위해 뷰를 복사해서 그 뷰 위에 올려놓아야 하는 경우가 있을 수 있다. 

> 실제로 이를 사용한 사용자 정의 뷰 컨트롤러 전환에 대해 추후에 포스팅할 예정이다. 

그리고 전환이 완료되면 복사된 뷰는 뷰 계층에서 제거될 것이다. 

![example1](https://ehdrjsdlzzzz.github.io/2019/01/13/convert-to-convert-from/example1.png)

노란색 뷰는 흰색 뷰에 속하고 흰색 뷰는 파란 뷰, 파란 뷰는 녹색 뷰 그리고 녹색 뷰는 뷰 컨트롤러의 루트 뷰에 속하는 계층 구성을 하고 있다.

노란색 뷰를 복사하여 그 위치 그대로 그 위에 올려보자. 이때 뷰의 부모 뷰는 메소드의 이해를 위해 흰색 뷰가 아닌 윈도우가 될 것이다. 예제에서 노란색 뷰는 `subview`가 된다.

```swift
// ViewController.swift

private var subviewCopy: UIView?

...

@IBAction func copySubview(_ sender: Any) {
    if let subviewCopy = subviewCopy {
        subviewCopy.removeFromSuperview()
        return
    }

    let absoluteFrame = subview.convert(subview.frame, to: nil)

    subviewCopy = UIView(frame: absoluteFrame)
    subviewCopy?.backgroundColor = .cyan
    
    self.view.addSubview(subviewCopy!)
}
```

`to`에 `nil`이 담겼다. 즉 `subview`의 `frame`을 위치와 크기를 모두 유지한 채 윈도우 좌표계 상으로 변환하는 것이다. 현재 `subview`의 `frame`의 `origin` 좌표는 `(0, 0)`이다.

> `frame`은 부모 뷰에 상대적으로 결정되므로 부모 뷰의 좌측 상단에 위치하므로 `(0, 0)`이 된다.

변환된 결과인 `absoluteFrame`은 `(60, 104)`가 된다. `subviewCopy`를 `subview ` 위에 그대로 올리면서 직속 부모 뷰가 윈도우가 되기 위해선 `frame` 좌표가 윈도우에 상대적이어야 하기 때문이다. 즉 `(60, 104)`는 윈도우 좌표계를 기준으로 했을 때의 `subview`의 좌표인 것이다. 

![example2](https://ehdrjsdlzzzz.github.io/2019/01/13/convert-to-convert-from/example2.png)

위에서 언급했던 것 처럼 주체를 반대로 하여 `convert(_:, from:)`으로 이를 표현하면 다음과 같다.

```swift
guard let absoluteFrame = UIApplication.shared.keyWindow?.convert(subview.frame, from: subview) else { return }
```

---

### Move

이번엔 조금 더 복잡하게 현재 좌표계상의 위치와 동일하게 변환에 사용되는 좌표계상에서도 동일한 위치에 오게끔 해보자. 말로 표현하니 복잡한데 간단하게 표현하자면 현재 좌표계의 왼쪽 모서리 끝 상단(`origin`)에 뷰가 위치한다면 변환에 사용되는 좌표계상의 왼쪽 모서리 끝 상단에 뷰가 위치하게끔 해보자는 것이다. 노란색 뷰의 부모 뷰는 흰색 뷰이지만 위치는 파란 뷰 왼쪽 끝 상단에 오게 하는 것이다.

다음과 같이 작성해보면 결과는 원하는 대로 나올까?

```swift
@IBAction func convert(_ sender: UIButton) {
    let convertedFrame = subview.convert(subview.frame, to: blueView)
    subview.frame = convertedFrame
}
```

실행해보면 정상적으로 파란색 뷰의 상단 끝에 위치하지 않는 것을 확인할 수 있다. 이유는 간단하다. 좌표는 정상적으로 변환된다.  하지만 노란색 뷰의 `frame`은 여전히 흰색 뷰의 좌표계를 기준으로 결정되기 때문이다. 파란색 뷰 좌표계 상으로 노란색 뷰의 위치는 `(20, 20)`이다. 하지만 노란색 뷰의 부모 뷰는 흰색 뷰이므로 흰색 뷰를 기준으로 `(20, 20)`이 되므로 보다 오른쪽 아래로 위치하게 될 것이다. 

다음과 같이 코드를 변경해보자.

```swift
@IBAction func convert(_ sender: UIButton) {
    let convertedOrigin = blueView.convert(blueView.bounds.origin, to: subview.superview!)
    subview.frame.origin = convertedOrigin
}
```

무언가 어색하다. 그 이유는 `subview`를 변환을 위한 좌표계로 변환한 것이 아닌 변환할 좌표계를 `subview`를 기준으로 변환시켜준 것이다.

![Example3](https://ehdrjsdlzzzz.github.io/2019/01/13/convert-to-convert-from/example3.png)

파란색 뷰 상의 좌표인 `bounds`의 `origin`을 `subview`의 부모 뷰 즉 흰색 뷰 좌표계 상으로 변환을 한 것이다. 여기서 `bounds`를 사용한 이유는 `bounds`는 자신의 고유 좌표계를 기준으로 하기 때문이다. 파란 뷰의 `origin`은 흰색 뷰에 상대적으로 `(-20, -20)`에 위치하고 이 점을 흰색 뷰의 상대적인 `subview`의 `frame.origin`에 할당해주었기 때문에 노란색 뷰는 흰색 뷰를 부모로 하여 파란색 뷰 `origin`에 위치하게 된다. 노란색 뷰에선 `frame`을 사용한 이유는 노란색 뷰를 상대적으로 이동시켜야 하기 때문이다.

역시 `convert(_:, from:)`으로 작성해보면 다음과 같다.

```swift
let convertedOrigin = subview.superview!.convert(blueView.bounds.origin, from: blueView) 
```

---

### 마무리

오늘은 이렇게 간단하게 좌표계를 기준으로 뷰를 변환시켜보는 메소드들에 대해 공부해보았다. 실제로 좌표계를 자유자재로 사용하기 위해선 조금 더 익숙해져야겠다는 것을 깨달았다. 