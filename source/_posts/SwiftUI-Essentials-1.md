---
title: SwiftUI Essentials (1)
subtitle: WWDC2019
date: 2019-12-21 21:55:57
tags: ["WWDC", "SwiftUI"]
category: ["WWDC", "SwiftUI"]
---

## Views and modifiers

`View`들은 UI를 구성하는 가장 기본적인 블록이다. UIKit의 `UIView`나 AppKit의 `NSView`와 같이 UI를 구성하는 기본적인 단위라고 할 수 있다. 

다음 앱의 UI를 계층 구조로 살펴보자. 

![](https://ehdrjsdlzzzz.github.io/2019/12/21/SwiftUI-Essentials-1/ViewHierarchy.png)

이 계층을 SwiftUI로 작성하면 다음과 같다.

![](https://ehdrjsdlzzzz.github.io/2019/12/21/SwiftUI-Essentials-1/CodeHierarchy.png)

SwiftUI는 이러한 뷰의 계층을 코드로 표현한다. 왼쪽의 코드 구조는 오른쪽의 뷰 계층 구조와 상당히 흡사한 것을 확인할 수 있다. 

또한 코드에서 살펴볼 수 있듯이 뷰 계층을 표현하는데 `addSubview`와 같은 메소드를 사용하지 않는다. SwiftUI는 하나의 계층 구조를 각 뷰 조각들로 구성하는 것이 아니라 계층 전체를 하나의 완전한 구조로 생성한다. 왜냐하면 **SwiftUI는 뷰를** 명령형(imperatively)과 반대인 **선언형(declaratively)으로 정의**하고 있기 때문이다. 

명령형과 선언형의 차이점을 살펴보자. 

- 명령형 코드 : 명시적인 명령(explicit commands)을 통해 결과를 구성
- 선언형 코드 : 묘사(describing) 통해 결과를 구성. 단 이를 어떻게 생성할지는 다른 주체에 의해 결정

둘의 차이가 정의로만은 부족할 수 있다. 상황을 예를 들어 둘의 차이를 살펴보자. 

명령형 코드는 친구에게 아보카도 토스트를 만드는 방법을 알려주는 것과 같다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/21/SwiftUI-Essentials-1/ImperativeToast1.png)

![](https://ehdrjsdlzzzz.github.io/2019/12/21/SwiftUI-Essentials-1/ImperativeToast2.png)

선언형 코드는 아보카도 토스트를 만드는 요리사에게 토스트 주문을 하는 것과 같다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/21/SwiftUI-Essentials-1/DeclarativeToast.png)

친구에게 토스트 만드는 방법을 설명할 때는 `7. 아보카도의 중심을 제거해라.`와 같이 내가 직접 단계별로 필요한 결과를 전달한다. 

반면, 요리사에게는 내가 원하는 토스트의 모습을 묘사하여 전달하고 그것을 어떻게 만드는지는 전적으로 요리사의 몫이다. 그리고 요리사가 전문가라면 우리는 항상 최상의 품질을 보장받을 수 있다.

> 이 두 상황을 통해 명령형 코드와 선언형 코드의 차이점을 보다 쉽게 이해할 수 있었다.

SwiftUI가 요리사의 역할을 하는 것이다. 그럼 이제 SwiftUI의 요소들을 하나씩 살펴보자. 

### View Container Syntax

뷰 컨테이너는 여러 다른 컨텐트 뷰(Content View)들로 구성되어 있다. 뷰 컨테이너에는 `VStack`, `HStack` 등이 존재한다. 뷰 컨테이너의 일반적인 문법은 다음과 같다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/21/SwiftUI-Essentials-1/ViewContainerSyntax.png)

`VStack`을 다음과 같이 사용할 수 있는 것과 같다. 

```swift
VStack { 
  Imgae(...)
  Text(...)
  Text(...)
}
```

`Image`, `Text`와 같은 컨텐트 뷰들은 뷰 빌더(View Builder)라는 클로저 안에 나열된다. 그리고 뷰 컨테이너의 생성자는 이 뷰 빌더 클로저를 인자로 받는다. `addSubview`와 같은 함수를 호출하는 대신 이 클로저 블록 안에 원하는 뷰를 순서대로 나열만 해주면 된다.

실제로 뷰 빌더가 내부적으로 어떻게 동작하는지 확인하기 위해 `VStack` API를 살펴보자. 

![](https://ehdrjsdlzzzz.github.io/2019/12/21/SwiftUI-Essentials-1/ViewContainerSyntaxDetail.png)

생성자 인자 중 `content`에 `@ViewBuilder` 속성(attribute)이 붙어있는 것을 확인할 수 있다. 스위프트 컴파일러는 `@ViewBuilder` 속성이 붙어 있으면 해당 클로저를 우리가 나열한 컨텐트 뷰들이 포함된 단일 뷰를 반환하는 클로저로 변환한다. 

이런 특수한 클로저를 뷰 컨테이너의 생성자에 전달해줌으로써 뷰 컨테이너와 컨텐트 뷰들은 들여 쓰기로 자연스레 구분될 수 있다. 

또한 `VStack`은 `alignment`나 `spacing`과 같은 인자를 추가로 받아 정렬이나 간격을 조정해줄 수 있다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/21/SwiftUI-Essentials-1/ExampleAppCode.png)

SwiftUI에서 `VStack`, `HStack`과 같이 컨트롤(Control)도 뷰 컨테이너의 종류로 다른 뷰를 컨텐트 뷰로 포함할 수 있다. `Control`에는 `Button`, `Toggle` `Slider` 등이 있다. 위의 코드에서처럼 `Text`뿐만 아니라 다른 뷰도 컨텐트 뷰로 포함할 수 있다.

> 컨트롤은 사용자와 상호작용할 수 있는 요소들을 말한다. [공식 문서](https://developer.apple.com/documentation/swiftui/views_and_controls)를 통해 컨트롤에 어떤 것들이 있는지 알 수 있다. 

컨트롤과 뷰 컨테이너는 추후에 더 자세히 살펴보도록 하고 이젠 **`$`** 싸인에 주목해보자. 

### Binding Syntax

`Stepper`를 선언하는 코드를 살펴보자. `order.quantity`를 넘기는데 `$` 싸인이 앞에 붙었다. 이는 단순히. `order.quantity` **값**을 넘기는 것이 아닌 **바인딩**을 넘기는 것이다. 그럼 여기서 말하는 바인딩이란 뭘까?

영상의 예제 앱에서 `Stepper`는 `OrderForm`이란 뷰에 포함되어 있다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/21/SwiftUI-Essentials-1/BindingSyntax.png)

`OrderForm`은 현재 순서를 추적하기 위해 `Order` 타입에 의존하고 있다. 이 프로퍼티를 살펴보면 `@State`란 속성이 붙어있는 것을 확인할 수 있다. `@State` 속성이 붙어있으면 SwiftUI는 이를 보고 내부적으로 지속성 있는 상태(persistent state)를 생성하고 관리하며 상태의 값을 이 프로퍼티를 통해 접근하도록 한다.

우린 이 프로터티에 접근해 상태의 값을 읽거나 쓸 수 있다.

```swift
Text("Quantity: \(order.quantity)")
```

`Stepper`는 정적인 뷰가 아닌 컨트롤이다. 그 말은 사용자가 `Stepper`의 버튼을 누르면 그 상태가 변경될 수 있다는 의미다. 이를 위해선 단순히 읽기 전용인 값을 전달하는 것이 아니라 바인딩을 전달해야 한다. 

바인딩은 일종의 관리되는 참조(managed reference)로 이를 통해 하나의 뷰가 다른 뷰의 상태를 변경할 수 있다. 이 예제에선 `Stepper`가 `OrderForm`의 상태를 `$order.quantity`를 통해 변경하고 있는 것이다. 

> SwiftUI에서의 데이터 흐름에 대한 자세한 내용은 [Data Flow Through SwiftUI](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=2ahUKEwjF6tvttsbmAhVRJaYKHXDyBggQwqsBMAB6BAgKEAQ&url=https%3A%2F%2Fdeveloper.apple.com%2Fvideos%2Fplay%2Fwwdc2019%2F226%2F&usg=AOvVaw0HLHjam6QvNklY9QlJyvIA) 영상을 참고하자. 

다시 예제 앱으로 돌아와 우리가 아직 살펴보지 못한 문법을 살펴보자. 

### Modifier

![](https://ehdrjsdlzzzz.github.io/2019/12/21/SwiftUI-Essentials-1/ModifierText.png)

`Text("Avocado Toast")`에서 우린 `font(.title)`과 같은 메소드를 호출할 수 있다. 이 메소드가 하는 작업은 간단하다. 호출한 뷰로부터 새 뷰를 만들어내는 것이다. SwiftUI에서 이런 메소드를 변경자(Modifier)라 부른다. 

이런 변경자에 의해 뷰 계층이 어떻게 변경되는지 살펴보자.  `Text("Avocado Toast")`을 포함하는 `VStack`은 다음과 같은 계층 구조를 가진다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/21/SwiftUI-Essentials-1/NoModifierViewHierarchy.png)

하지만 여기에 `font(.title)` 변경자를 적용하면 뷰 계층은 다음과 같이 변경된다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/21/SwiftUI-Essentials-1/ModifierViewHierarchy.png)

이렇게 변경자로 생성된 뷰는 기존의 뷰를 감싸고 뷰 계층에 포함된다. 이런 변경자는 다수의 변경자들과 함께 체이닝될 수 있다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/21/SwiftUI-Essentials-1/MultipleModifierViewHierarchy.png)

이렇게 변경자를 추가하게 되면 계층 구조는 빠른 속도로 비대해진다. 우리는 이전까지 이렇게 뷰 계층이 비대해지면 성능 이슈에 대해서 고민하곤 했다. 기존의 뷰 계층은 최대한 작고 가벼워야 했다.

하지만 SwiftUI는 이러한 부분에 대한 걱정을 덜어도 된다. 위에서 언급했듯이 우리는 선언형 코드를 작성한다. 우리는 단지 원하는 모습을 묘사할 뿐이고, SwiftUI가 이를 최적화한다. 우리가 아무리 많은 변경자를 사용해 `Text`를 여러 뷰로 감싸도 SwiftUI가 이를 보다 효율적인 자료구조로 최적화한다. 그리고 이렇게 최적화된 자료구조는 렌더링 시스템이 렌더링 하는데 사용한다. 

이렇게 변경자 체이닝 문법은 성능 이슈에 대해 걱정할 필요 없이 많은 이점을 제공한다. 그중 하나로 변경자 체이닝은 시각적 요소의 직관적인 순서를 강제한다. 즉 체이닝에 참여하는 변경자의 순서에 따라 최종 렌더링 되는 모습이 달라진다는 것이다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/21/SwiftUI-Essentials-1/PaddingLast.png)

![](https://ehdrjsdlzzzz.github.io/2019/12/21/SwiftUI-Essentials-1/PaddingFirst.png)

만약 이런 속성들을 변경자 체이닝으로 변경하는 것이 아닌 `Text`의 내부에 포함된 속성이라고 가정해보자. 우린 시행착오와 문서 없이는 각각의 속성들이 어떤 순서로 적용되는지 알 수 없을 것이다. 이런 속성들을 변경자를 통해 적용함으로써 우린 순서를 명시적으로 지정할 수 있다. 

또한 이런 변경자들은 여러 뷰들에서 공유될 수 있다.

![](https://ehdrjsdlzzzz.github.io/2019/12/21/SwiftUI-Essentials-1/ModifierWithoutSharing.png)

![](https://ehdrjsdlzzzz.github.io/2019/12/21/SwiftUI-Essentials-1/ModifierWithSharing.png)

이렇게 변경자를 공유함으로써 각각의 뷰들은 보다 단순해질 수 있고 자신들만의 인터페이스에 집중할 수 있다. 이것이 SwiftUI의 기본 원칙이다.

![](https://ehdrjsdlzzzz.github.io/2019/12/21/SwiftUI-Essentials-1/SwiftUIPrinciple.png)

더 작고 단일 목적의 뷰라는 원칙을 따름으로써 우리는 보다 이해하기 쉽고, 유지 보수가 쉬운 뷰를 만들 수 있다. 

> 재사용성 역시 증가한다.

![](https://ehdrjsdlzzzz.github.io/2019/12/21/SwiftUI-Essentials-1/SwiftUIPrinciple2.png)

그리고 이렇게 각자의 역할별로 작게 나누어진 뷰들을 통해 보다 큰 뷰를 효과적으로 구성할 수 있다. 

