---
title: SwiftUI under the hood
subtitle: by Chris Eidhof 
date: 2020-02-15 17:26:52
tags: ["SwiftUI", "GeometryReader", "PreferenceKey"]
category: ["SwiftUI"]
---

요즘 SwiftUI를 공부하면서 느낀 점은 마냥 쉽지만은 않다는 것이다. 흔히 SwiftUI를 소개할 때 매우 쉽게 UI를 그릴 수 있는 프레임워크로 소개하곤 하는데, 완전히 틀린 말은 아니지만 그렇다고 완전히 맞는 말도 아닌 것 같다. SwiftUI를 공부하면서 처음 iOS를 공부했을 때, 더 나아가 처음 프로그래밍을 공부했을 때가 종종 생각나곤 한다. 

당시 처음으로 프로그래밍 언어를 가르쳐 주신 강사님으로부터 기본적인 C 언어 문법을 배우고 모든 걸 배운 것처럼 기세등등했던 적이 있다. 자신감이 한없이 하늘을 찌르고 있을 때 강사님은 다음과 같은 말씀을 해주셨다. 

*"원래 처음 프로그래밍을 배우고 기본 문법을 마친 사람들이 프로그래밍을 매우 쉽다고 생각하고 자만하게 됩니다. 하지만 공부를 하면 할수록 어렵게 느껴지고 자신감이 떨어질 수 있는 공부가 프로그래밍입니다."*

> 비단 프로그래밍에만 국한되는 이야기는 아닌 것 같다.

당시에는 이해가 가질 않았으나, 본격적으로 학부 공부를 시작하고 취업 준비를 하면서 저 말이 계속해서 떠올랐고 자신감이 자주 떨어지곤 했다. SwiftUI를 공부하면서도 마찬가지였다. 처음 `VStack`, `List` 등을 사용하면서 그 간편함에 놀라고 매우 쉽다고 생각했다. 하지만 데이터의 흐름과 레이아웃이 결정되는 방식 등 깊게 파고들면 파고들수록 머릿속은 여러 개념들이 완전하지 않은 채  뒤엉키게 되었다. 그렇게 혼란스러워하던 중 한 컨퍼런스에서 Chris Eidhof가 "SwiftUI under the hood"란 주제로 발표한 영상을 보고 복잡하고 산발되어 있던 개념들이 어느 정도 정리가 되면서 여러 개념들의 존재 이유와 목적에 대해 감을 잡을 수 있었다. 오늘은 그 영상의 내용을 요약 및 정리해보는 시간을 가지려 한다. 

> 영상 : [Chris Eidhof - SwiftUI under the hood](https://youtu.be/GuK6wwX8M0E)

영상에서 다루는 큰 주제는 레이아웃 알고리즘이다. 즉 SwiftUI가 어떤 방식으로 레이아웃을 그리는지에 대해 다룬다. 이를 설명하면서 `GeometryReader`, `Preference` 등 중요한 개념들이 자연스럽게 등장하며 설명을 돕는다.

서론이 너무 길었다. 바로 시작해보자.

---

### Layout Algorithm

파란색 배경의 원 안에 텍스트가 존재하는 뷰를 만든다고 상상해보자. 원이 아닌 다양한 도형이 될 수 있다. 즉 매우 흔한 상황이다. 우리가 원하는 결과물은 다음과 같을 것이다.

<img src="https://ehdrjsdlzzzz.github.io/2020/02/15/SwiftUI-under-the-hood/0.5.png" width=400>

매우 쉬워 보인다! 그럼 코드로 이를 만들어보자. 

```swift
Text("Reset")
	.background(
    Circle()
    .fill(Color.blue)
  )
```

기대하던 모습이 나올까? 그렇지 않다. 그 모습은 우리가 기대하던 모습과 거리가 멀다.

<img src="https://ehdrjsdlzzzz.github.io/2020/02/15/SwiftUI-under-the-hood/1.png" width=400>

이유가 무엇일까? 그 이유를 알기 위해선 SwiftUI의 레이아웃 알고리즘에 대한 이해가 있어야 한다. 모든 레이아웃은 다음 4가지 단계를 통해 그려진다. 

1. 컨테이너 뷰(직계 상위 뷰)가 사이즈를 제안한다. 
2. 하위 뷰가 자신의 사이즈를 결정한다. (하위 뷰는 상위 뷰가 제안한 사이즈를 그대로 사용하거나, 본인이 자신의 사이즈를 결정하기도 한다.)
3. 하위 뷰가 컨테이너 뷰에 자신의 사이즈를 알린다. (2단계에서 결정된 사이즈로 컨테이너 뷰의 사이즈가 정해진다.)
4. 컨테이너 뷰가 하위 뷰를 가운데 정렬 시킨다. (`alignment`)

이는 아주 기본적인 절차이고 여러 값들의 재정의를 통해 변경될 수 있다. 그럼 이 알고리즘을 바탕으로 위의 코드를 분석해보자. 먼저 위의 코드로 그려진 뷰의 계층을 뷰 디버깅을 통해 살펴보고 아래 설명을 따라가면서 이해해보자.

{% asset_img 3.png %}

1. 먼저 루트 뷰(root view)가 화면 전체 사이즈를 `background`에게 제안한다. 그리고 `background`는 이를 다시 `Text`에게 전달한다. 

   > Modifier는 새로운 뷰를 만들고 이전 뷰를 감싸서 반환한다는 것을 기억한다면 `background`가 `Text`를 감싸고 있다는 것을 이해할 수 있다. 

2. 하지만 `Text`는 자신이 담고 있는 내용만큼만 필요하기 때문에 `"Reset"`만큼만 사용할 것을 `backgroud`에 알린다. 

3. 그럼 다시 `background`는 이를 하위 뷰인 `ShapeView`에 알리고 `ShapeView`는 이를 `Circle`에게 알린다. (`ShapeView`와 `Text`는 형제-자매(sibling) 관계다.) `Circle`는 받은 크기(`Text`의 크기)를 그대로 사용해서 그 크기에 딱 맞는 원을 그린다.

4. 그리고 이를 `background`에게 전달하고 `background`는 다시 이 크기를 루트 뷰에 전달하고 루트 뷰는 해당 뷰를 가운데 정렬한다.

뷰 계층과 알고리즘을 따라가보면 왜 우리가 원하는 레이아웃이 나오지 않았는지에 대해 이해할 수 있다. 그럼 우리가 원하는 레이아웃을 그리기 위해선 어떻게 해야 할까?

가장 먼저 떠오르는 생각은 `frame` 변경자를 사용하는 것이다. 사용하기에 앞서 `frame` 변경자의 특징을 알아야 한다. **`frame` 변경자는 상위 뷰에서 오는 크기 정보나, 하위 뷰에서 알려주는 크기 정보를 모두 무시하고 자신의 인자로 넘어온 크기 정보만 사용한다.** 이 점을 기억하고 아래 코드를 살펴보자.

```swift
Text("Reset")
//.frame(width: 75, height: 75) // --- 1
	.background(
    Circle()
    	.fill(Color.blue)
//    .frame(width: 75, height: 75) // --- 2
  )
//.frame(width: 75, height: 75) // --- 3
```

`frame` 연산자를 1, 2번 두 곳 중 한 곳에만 위치시켜도 원하는 레이아웃을 그릴 수 있다. 하지만 두 곳 모두 문제점을 갖고 있다. 

1. 위에서 언급했듯이 `frame`은 하위 뷰에서 알려준 크기 정보도 무시한다. 즉 `Text`가 정한 크기도 무시한다는 뜻이다. 그렇기 때문에 이곳에 `frame`을 위치시면 길어진 문자열 크기를 `Text`가 아무리 `frame`에 알려도 `frame`은 자신의 정보로 크기를 결정하기 때문에 문자열은 75x75 영역 안에서 벗어날 수 없고, 해당 영역 안에 표시되지 못한 부분은 말 줄임표(`...`)로 나타나게 된다.

   <img src="https://ehdrjsdlzzzz.github.io/2020/02/15/SwiftUI-under-the-hood/5.png" width=400>

2. `frame` 영역은 `Circle`에만 영향을 주기 때문에 문자열이 길어지면 원의 영역을 벗어난다. 

   <img src="https://ehdrjsdlzzzz.github.io/2020/02/15/SwiftUI-under-the-hood/6.png" width=400>

`frame`을 3번에 위치시키면 어떻게 될까? `frame`이 이 크기 정보(75x75)를 `background`에게 `background`가 다시 `Text`에 전달하지만 `Text`는 자신이 담고 있는 내용만큼만 사용한다고 `background`에 알리고 `background`는 이 정보를 다시 `Circle`에 전달하기 때문에 결과적으로 위에서 봤던 레이아웃과 동일한 것을 육안으로 확인할 수 있다.

<img src="https://ehdrjsdlzzzz.github.io/2020/02/15/SwiftUI-under-the-hood/4.png" width=400>

육안으로 확인한 레이아웃이 같다고 실제로 그 둘이 같은 것은 아니다. 전체를 감싸고 있는 뷰의 크기 차이가 존재한다(파란 테두리가 감싸고 있는 영역). 그 이유는 위에서 언급했듯이 `frame`은 하위 뷰에서 보낸 정보도 무시하기 때문에 자신의 크기 정보(75x75)를 사용한다. 그렇기 때문에 전체를 감싸고 있는 뷰의 크기에 차이가 생기는 것이다.

문제를 해결하기 위해선 `frame` 안에 들어갈 값은 문자열의 길이에 따라 동적으로 변경되어야 한다. 정확히는 `Text`의 크기에 따라 변경되어야 한다. 그럼 `Text`의 크기는 어떻게 알 수 있을까? UIKit을 사용할 땐 객체에 직접 접근해 값을 가져올 수 있었다. 하지만 SwiftUI에선 불가능하다. 우리는 이 문제점을 `GeometryReader`를 사용해 해결해보려 한다. 

### GeometryReader

`GeometryReader`는 SwiftUI에서 중요한 개념이지만 이번 포스팅에선 현재 상황에서 `GeometryReader`가 해결할 수 있는 부분에 대해서만 간단하게 설명해보고자 한다. 

`GeometryReader`는 컨테이너 뷰의 한 종류로 자신의 직계 상위 뷰의 기하학(Geometry) 정보(좌표, 크기 등)를 자신이 포함하는 자식 뷰에게 제공하는 역할을 한다. 그럼 바로 사용해보자. 

```swift
Text("Reset")
	.padding()
	.fixedSize()
	.background(
  	GeometryReader { proxy in 
    	// proxy.size.width == Text("Reset")'s width               
		}
  )
	.frame(width: 75, height: 75)
	.background(
  	Circle()
    	.fill(Color.blue)
  )
```

> `fixedSize` 변경자는 `Text`에만 존재하는 변경자로 아무리 문자열이 길어져도 1줄에 보여주도록 강제한다. 이를 사용하면 위에서 보았던 이미지처럼 주어진 크기 안에서 문자열이 길어질 때 여러 줄과 함께 말 줄임표로 보여주는 것이 아니라 영역을 벗어나더라도 문자열을 1줄에 보여줄 수 있다.

단순히 직계 상위 뷰의 정보를 받아온다고 생각하지 말고 위에서 살펴보았던 레이아웃 결정 과정을 대입해서 생각해보자.

1. `background`는 루트 뷰로부터 크기를 제안받는다. 
2. `background`는 그 크기를 `Text`에게 제안한다.
3. `Text`는 자신이 포함하는 내용만을 담을 수 있는 크기를 사용하기로 결정하고 이를 `background`에게 알린다.
4. `background`는 그 정보를 `GeometryReader`에게 알린다. 

이런 순서로 `GeometryReader`는 직계 상위 뷰의 정보를 받아올 수 있는 것이다. 정확히 표현하자면 `Text`에 의해 결정된 크기를 사용하는 `background`의 크기 정보를 받아온 것이다. 그럼 우린 이 크기 정보를 사용하면 된다. 

```swift
@State private var width: CGFloat? = nil

Text("Reset")
	.padding()
	.fixedSize()
	.background(
  	GeometryReader { proxy in 
      self.width = proxy.size.width
		}
  )
	.frame(width: self.width, height: self.width)
	.background(
  	Circle()
    	.fill(Color.blue)
  )
```

위와 같이 코드를 작성할 수 있을 것 같지만 실제로 이렇게 작성하면 에러가 발생한다. 왜냐하면 `GeometryReader`는 생성자로 `@ViewBuilder`를 받기 때문에 `self.width = proxy.size.width`와 같은 코드는 `@ViewBuilder` 안에 작성할 수 없다.

우린 이 시점에서 프록시(`GeometryProxy`)에 담긴 크기 정보를 바로 `width` 프로퍼티에 할당할 수 없다. 우린 이 정보를 뷰 계층 위로 전달해야 한다. 이렇게 하위 뷰에서 상위 뷰로 정보를 전달하기 위해서 사용하는 것이 바로 `Preference`다.

### Preference

`Preference` 역시 `GeometryReader`와 마찬가지로 굉장히 중요한 개념 중 하나이다. 하지만 마찬가지로 현재 상황에서 `Preference`가 해결할 수 있는 부분에 대해서만 간단하게 설명해보고자 한다. 

`Preference`는 키-밸류 메커니즘으로 하위 뷰 정보를 상위 뷰에 전달할 수 있는 수단이다. 이를 위해선 먼저 `PreferenceKey` 프로토콜을 따르는 키를 정의해주어야 한다. 

```swift
struct SizeKey: PreferenceKey {
  static func reduce(value: inout CGFloat?, nextValue: () -> CGFloat?) {
    value = nextValue()
  }
}
```

간단하게 설명하자면 `reduce` 메소드는 `SizeKey`를 사용하는 하위 뷰들을 순회하면서 상위 뷰가 접근할 수 있는 값을 만들기 위해 이들의 값(`SizeKey`를 사용하는 하위 뷰의 값) 취합하는 역할을 한다.

> 이번 포스팅에선 단순히  `PreferenceKey`의 기능만을 소개하지만 추후 포스팅에서 더욱 자세히 다룰 예정이다. 하지만 당장 궁금하다면 [이 글](https://swiftui-lab.com/communicating-with-the-view-tree-part-1/)을 참고하면 좋을 것이다.

그리고 간단한 트릭(?)을 사용해서 우린 뷰 계층 위로 `proxy` 정보를 전달할 수 있다. 

```swift
struct ContentView: View {
  @State private var width: CGFloat? = nil

  var body: some View {
    Text("Reset")
    .padding()
    .fixedSize()
    .background(
      GeometryReader { proxy in
				Color.clear.preference(key: SizeKey.self, value: proxy.size.width) // --- 1
			}
    )
    .frame(width: self.width, height: self.width)
    .background(
      Circle()
      .fill(Color.blue)
    )
    .onPreferenceChange(SizeKey.self) { value in // --- 2
			self.width = value
		}
  }
}
```

1. `Color`도 `View` 프로토콜을 따른다. 그렇기 때문에 사용자는 볼 수 없는 `Color.clear`에 `preference`를 통해 `proxy.size.width`를 전달하고 있다. 
2. 상위 뷰(`Text`)에선 하위 뷰에서 전달한 정보를 `onPreferenceChange`를 통해 받을 수 있다. 

이제 진짜 우리가 원하던 레이아웃을 확인할 수 있을 것이다. 

---

### 정리하며 

간단한 레이아웃(?)을 그려보면서 SwiftUI를 관통하는 여러 개념들을 자연스럽게 접해볼 수 있던 영상이었다. SwiftUI가 레이아웃을 그리는 알고리즘에 대해 이해하고 나니 왜 그렇게 그려지는지 이해가 가지 않았던 부분들이 어느정 도 머릿속에 정리가 되는 시간이었다.

> 영상 말미에 한 가지 과제(?)를 주는데 이 부분은 [github](https://github.com/ehdrjsdlzzzz/SwiftUI-practice/tree/master/SwiftUIUnderTheHood)에 올려놓았습니다.