---
title: High Performance Auto Layout
subtitle: WWDC 2018
date: 2019-01-04 16:55:38
tags: ["WWDC2018", "Auto Layout", "Render Loop"]
category: ["WWDC2018", "Auto Layout"]
---

## High Performance Auto Layout 

> WWDC 2018 High Performance Auto Layout 영상을 보고 이해한 것을 바탕으로 정리한 내용입니다. 

### Performance Improvements in iOS 12 

<img src="https://ehdrjsdlzzzz.github.io/img/HighPerformanceAutoLayoutAssets/HigherPerformance.png">

첫 번째 세션에선 성능의 향상을 서로 다른 iOS(11과 12) 버전에서의 차이를 육안으로 보여주었고 이를 위와 같은 그래프로 보여주었다. 성능이 전체적으로 iOS 11에 비해 좋아졌다고 한다. 확실히 iOS 11에 비해 12에선 보다 부드러운 스크롤을 확인할 수 있었다.

------

### Internal and Intuition 

두 번째 세션에선 오토레이아웃이 내부적으로 어떻게 동작하는지에 대한 세션이었다. 그리고 내부 동작 원리를 간단하게 설명해주는 동시에 이를 바탕으로 비효율적으로 오토레이아웃을 작성하는 것을 피하고 효율적으로 작성하는 방법들에 대해 설명해주었다. 

먼저 이에 대한 이해를 하려면 `Constraint`를 바탕으로 뷰들의 배치와 이를 출력해주는 과정인 **Render Loop**에 대해 알아야 한다. 이 프로세스는 초당 120번 실행되면서 보여주어야 하는 컨텐츠들을 배치하게 된다. 즉 굉장히 빠른 시간 내에 여러번 실행된다. 즉 `민감한` 프로세스라 할 수 있다.  그렇기 때문에 이와 연관된 메소드들을 사용하거나 메소드를 오버라이드 할 경우에는 효율성에 대해 고민을 하며 사용해야 한다. 

Render Loop의 흐름은 다음과 같다. 

<img src="https://ehdrjsdlzzzz.github.io/img/HighPerformanceAutoLayoutAssets/RenderLoop.png">

- **Update Constraints**
  -  `Constraint`를 갖고 있는 모든 뷰는 자식 뷰로부터 부모 뷰로 올라가며  `updateConstraints` 메소드를 호출해가며 `Constraint`를 기반으로 뷰 배치에 필요한 위치와 크기에 대한 계산을 진행한다. 
  - [updateConstraints()](https://developer.apple.com/documentation/uikit/uiview/1622512-updateconstraints)
- **Layout**
  -  이 과정은 부모 뷰로부터 자식 뷰로 내려가며 `layoutSubviews` 메소드를 호출한다. 이 과정은 `Update Constraints` 단계에서 계산된 결과를 바탕으로 뷰를 배치하는 과정이다. 
- **Display** 
  - 이 과정 역시 부모 뷰로부터 자식 뷰로 내려가며 `draw(_:)` 메소드를 호출한다. 이 과정을 통해 뷰가 보여주어야 할 컨텐츠를 출력해준다. 

위에서 언급했듯이 이 과정들은 1초에 120번 진행된다. 그러므로 `updateConstraints` 내부에서 `Constraint`를 비활성화하고 활성화하는 코드를 작성하게 된다면 비활성화와 활성화가 1초에 120번 진행되는 것이다. 즉 상당히 비효율적인 코드라 할 수 있다. 이런 코드는 지양해야할 것이다. 

Render Loop와 연관되어 있는 메소드들은 다음과 같다. 

> Main Run Loop와 Update Cycle 그리고 `layoutIfNeeded`와 `setNeedsLayout`의 차이를 공부해보신 분이라면 익숙할 표입니다. 

|      Update Constraints       |       Layout       |       Display       |
| :---------------------------: | :----------------: | :-----------------: |
|     `updateConstraints()`     | `layoutSubviews()` |     `draw(_:)`      |
| `setNeedsUpdateConstraints()` | `setNeedsLayout()` | `setNeedsDisplay()` |
| `updateConstraintsIfNeeded()` | `layoutIfNeeded()` |          _          |

이 각각의 메소드들은 실제로 Render Loop가 1초당 120번 실행될 때 마다 실행되는 작업을 최소화하는데 주로 사용된다. 즉 Render Loop에서 작업을 진행할 때 불필요한 비효율적인 작업을 지양하기 위해 존재하는 메소드들이다. 

좀 더 안으로 깊게 들어가보자. `Constraint`가 활성화 될 때 내부적으로 어떤 구조를 갖고 있으며 어떤 과정을 거쳐서 `Constraint`가 활성화되는지 살펴보자

<img src="https://ehdrjsdlzzzz.github.io/img/HighPerformanceAutoLayoutAssets/InternalStructure.png">

먼저 뷰에 `Constraints`가 할당되면 그에 대한 방정식(**Equation**)이 생기게 된다. 뷰의 위치와 크기(**x, ,y, width, height**)를 추론할 수 있는 방정식을 엔진이 계산하여 반환한다. 이 결과는 **layout** 단계에서 사용된다.

다음과 같은 뷰를 예로 들어보자. 

> 과정을 살펴보기 위함으로 예제에선 `Horizontal` 배치에 관해서만 살펴보자

<img src="https://ehdrjsdlzzzz.github.io/img/HighPerformanceAutoLayoutAssets/LayoutExample.png">

위와 같이 하나의 뷰 안에 두 개의 `UILabel`을 다음과 같이 배치하였을 때 우리는 위와 같은 방정식들을 얻게 된다. 그렇다면 엔진은 이들을 이용하여 값을 계산하게 된다.

```
text1.minX = 20
text1.width = 100
text2.minX = text1.minX + text1.width + 20 
-->	text2.minX = 20 + 100 + 20 = 140
text2.width = 100 
```

그렇다면 두 `UILabel`의 배치에 필요한 `Horizontal` 요소들은 모두 결정이 된 것이다. 이렇게 값이 결정되면 엔진은 뷰에게 값이 모두 계산되었다는 것을 알린다. 그러면 뷰는 자신의 부모 뷰에게 `setNeedsLayout`을 호출할 것을 요청한다. (뷰의 배치를 다시 해야한다는 것을 부모 뷰에 알리는 것) 여기까지가 Render Loop의 세 단계 중 **Update Constraints** 단계이다.

**Layout** 단계에선 엔진이 계산한 값을 복사하여 부모 뷰에서 `layoutSubviews` 메소드를 호출하여 본격적으로 뷰들을 배치하기 시작한다. 이러한 과정을 통해 우리는 뷰들을 `Constraint`에 기반하여 배치하고 출력해주게 된다. 

> 부모 뷰에서 `layoutSubviews`가 호출되면 계산된 값을 바탕으로 뷰를 배치하고 자식 뷰들의 `layoutSubviews`도 차례로 호출되며 같은 과정을 반복한다. 

이 예제의 경우는 굉장히 간단한 경우의 예이다. 그렇다면 요소들을 추가하여 비효율적으로 작성된 오토레이아웃을 살펴보자. 위의 예제에서 같은 부모 뷰를 하나 더 추가하였다. 

<img src="https://ehdrjsdlzzzz.github.io/img/HighPerformanceAutoLayoutAssets/Unrelated.png">

위에서와 `leading`을 지정해주기 위해 우리는 가끔 서로 같은 계층에 속하지 않은 뷰끼리 `Constraint` 관계를 맺을 때가 있다. 이러한 경우는 내부적으로 효율적이지 못하고 성능에 영향을 주게 된다. 그 이유는 기본적으로 엔진에서 값들을 계산할 때 하나의 뷰 계층은 다른 뷰 계층과 독립적으로 계산이 된다. 

<img src="https://ehdrjsdlzzzz.github.io/img/HighPerformanceAutoLayoutAssets/UnrelatedEngine.png">

만일 이렇게 독립적으로 계산되는 과정에서 내부적으로 서로 다른 계층의 내부에서 관계를 갖게 되면 그 계산 과정은 복잡해지며 이는 성능 저하로 이어진다. 같은 계층 내부에서만 관계를 갖는다면 성능은 선형의 그래프를 그리고 이러한 그래프는 이상적인 그래프라 할 수 있다. 

<img src="https://ehdrjsdlzzzz.github.io/img/HighPerformanceAutoLayoutAssets/LinearGraph.png">

영상을 보면 다음의 문장을 굉장히 강조한다. 

***The engine is a layout cache and dependency tracker***

엔진은 어떤 `Constraint`가 어떤 뷰에 영향을 주는지 알고 있으며 무언가를 변경하면 해당 변경에 따라 이를 갱신한다. 즉 각각의 `Constraint`의 의존 관계를 모두 파악하고 있는 것이다. 그러므로  `Constraint`를 간단하게 표현할 수록 성능은 복잡한 관계일 때 보다 좋아질 것이다. 

> 사실 *The engine is a layout cache and dependency tracker*의 의미가 확실히 와닿지는 않습니다. 이에 대한 이해가 더욱 필요할 것 같습니다.

------

### Building Efficient Layouts

영상에 자주 등장하는데  `Constraint Chrun`이란 무엇일까? 세 번째 세션의 시작에서 이를 간단하게 정의해주고 있다. `Constraints`를 변경했지만 보여지는 뷰는 변화가 없을 때를 말하며 엔진에게 추가 작업을 할당하고 성능에 영향을 주는 행위라고 할 수 있다. 우리는 이러한 `Constraint Churn`을 지양해야 한다. 이러한 현상을 지양하는 것이 효율적으로 레이아웃을 만든다라고 할 수 있다. 

> 이렇게 비효율적인 레이아웃 빌드 과정을 검사할 수 있는 기능을 Instruments를 통해 확인할 수 있다고 한다. 

이 세션에선 `Constraint Churn`을 피하는 방법으로 다음의 것들을 나열하고 있다. 

- **Avoid removing all Constraints**
  - 뷰에 연관되어 있는 모든 `Constraint`를 제거하는 행위를 지양하자 (3번째와 관련되어 있음)

- **Add static constraints once**
  - `Constraint`가 변하지 않는 뷰는 최초 한 번만 추가하자.

- **Only change the constraints that need changing** 
  - 변화가 필요한 `Constraint`에만 변화를 주자.

- **Hide views instead of removing them** 
  - 필요 없는 뷰를 뷰 계층에서 제거하는 것보다 숨기는 것이 훨씬 효율적이다.
  - 궁금한 점 : 뷰를 제거하는 것과 숨기는 것에는 메모리 차원에서 차이가 있지 않을까?

#### Intrinsic Content Size

고유 컨텐츠 사이즈를 갖는 뷰가 몇몇 있다. `UILabel`, `UIImageView` 그리고 `UIButton` 등이 그 예인데 이들은 크기에 관한 `Constraint`를 지정해주지 않아도 고유 컨텐츠 사이즈를 갖고 있기 때문에 에러가 발생하지 않는다.

 `UILabel`을 예를 들어 `UILabel`은 내부적으로 자신이 갖고 있는 텍스트의 사이즈를 계산한다. 이 과정은 텍스트의 양이 많아지면 그 비용은 비싸진다. 즉 텍스트의 양에 따라 추가적으로 작업을 해줄 필요가 있을 수 있다. (**텍스트 집약적 앱**)

예를 들어 많은 양의 텍스트를 포함하는 `UILabel`의 사이즈를 텍스트의 양을 측정하는 과정없이 알고 있다면 해당 사이즈를 이용하여 `Constraint`를 지정해주고 `intrinsicContentSize`를 다음과 같이 오버라이드한다.

```swift
override var intrinsicContentSize: CGSize {
    return CGSize(width: UIView.noIntrinsicMetric, height: UIView.noIntrinsicMetric)
}
```

이런 과정을 통해 대량의 텍스트를 포함하는 `UILabel`이나 `UITextView`를 사용하는데 있어 텍스트 양의 측정 과정을 생략함으로써 성능을 조금 더 향상시킬 수 있다. 

#### systemLayoutSizeFitting(_:)

사실 이 메소드에 대한 이해는 영상을 보고 문서를 읽어보아도 아직은 많이 부족한 상황이다. 단지 영상에 나온 내용을 요약하자면 이 메소드를 호출할 때마다 엔진이 생성되고 프레임을 반환한 뒤 엔진이 해제되기 때문에 비용이 많이 들어가는 작업이라는 것이다. 