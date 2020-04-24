---
title: SwiftUI Essentials (2)
subtitle: WWDC2019
date: 2019-12-26 21:10:52
tags: ["WWDC", "SwiftUI"]
category: ["WWDC", "SwiftUI"]
---

## Building custom views 

SwiftUI로 커스텀 뷰를 만드는 방식에 대해 이야기 해보자. 

주문 내역을 보여주는 `OrderHistory`를 살펴보자.

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-26 at 9.20.17 PM.png)

위의 코드에서 먼저 살펴볼 부분은 바로 `View` 프로토콜을 따르고 있는 `OrderHistory`가 구조체로 선언되어 있다는 점이다.

일반적으로 UIKit으로 뷰를 만들면 프로토콜을 따르는 구조체가 아닌 공통 부모 클래스로부터 상속받는 클래스를 작성하곤 한다. `OrderHistory`를 UIKit으로 만든다면 다음과 같은 상속 관계를 가질 것이다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-26 at 9.24.48 PM.png)

`UIView`는 `alpha`나 `backgroundColor` 같은 공통된 저장 프로퍼티(stored property)를 갖고 있다. `OrderHistory`는 자신의 `previousOrders` 프로퍼티와 더불어 부모 클래스의 프로퍼티까지 갖고 있게 된다. 반면에 SwiftUI는 어떨까? 

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-26 at 9.29.19 PM.png)

SwiftUI는 이런 공통된 저장 프로퍼티를 분리된 변경자로 관리하고 각각의 변경자는 자신들만의 뷰를 생성하게 된다. 그러므로 공통된 저장 프로퍼티는 뷰 계층 전반에 걸쳐 분산된다. 이러한 방식으로 뷰를 더 가볍게 해서 각 뷰의 고유 목적에 맞게 최적화한다.

이러한 방식 때문에 SwiftUI에서 뷰가 프로토콜이 된다고 할 수 있는 것이다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-26 at 9.37.57 PM.png)

그럼 뷰는 무엇을 하는 것일까?

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-26 at 9.39.40 PM.png)

뷰는 단지 UI의 한 조각을 정의할 뿐이고 우린 이런 뷰들을 이용하고 재사용하여 뷰 계층을 구성하는 것이다. `View` 프로토콜의 살펴보자. 

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-26 at 9.46.41 PM.png)

위의 코드를 보고 있으면 어떤 생각이 드는가? 재귀적이지 않은가? 

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-26 at 9.46.37 PM.png)

하나의 뷰가 있고 그 뷰의 `body`가 다른 뷰를 나타내고 그 뷰의 `body`가 또 다른 뷰를 나타내는 이런 구조를 보일 수 있다고 생각할 수 있는데 이는 지속되지 않는다. 그 이유는 SwiftUI가 스스로 컨텐츠를 갖지 않고 다른 뷰를 구성하는 아토믹(atomic)한 뷰인 원시 뷰(primitive view)를 제공하고 위와 같은 `body` 사슬의 끝은 결국 이런 원시 뷰이기 때문이다.

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-26 at 9.57.52 PM.png)

우리가 위에서 보았던 `Text`나 `Image`와 더불어 드로잉에 사용되는 `Color`와 `Shape`, 레이아웃에 사용되는 `Spacer`와 같은 다양한 원시 뷰를 제공한다. 

다시 `OrderHistory`로 돌아와 클래스가 아닌 구조체로 정의된 것에 주목해보자. 클래스로 정의한 것이 아니기 때문에 `OrderHistory`는 더 이상 이벤트 기반으로 동작하는 명령형 코드로 갱신되는 영구적인 객체가 아니다.

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-26 at 10.08.44 PM.png)

대신 뷰는 Input에 따라 결과가 달라지는 함수와 같이 선언형 코드로 정의된다. 이 말은 Input이 변경되면 SwiftUI가 `body` 프로퍼티를 다시 호출해서 뷰를 갱신한다.

만일 이벤트 기반의 명령형 코드였다면 Input의 변경(삭제, 삽입 등)에 따른 갱신 코드를 작성해주어야 했는데, SwiftUI에서는 선언형 코드로 인풋이 변경되면 SwiftUI가 내부적으로 이전 데이터와 새 데이터를 비교해서 무엇이 변경되었는지를 비교 후 효율적으로 뷰를 갱신하게 된다.

`OrderHistory` 코드를 계속해서 살펴보자. 조건에 따라 뷰의 유무를 표시할 때 우리는 다음과 같이 뷰 빌더 클로저 안에 조건문을 통해 이를 구현할 수 있다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-26 at 10.19.10 PM.png)

하지만 이런 조건문도 상황에 따라 제대로 사용해야 한다. 다음 상황의 코드를 살펴보자. 

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-26 at 10.22.09 PM.png)

`flipped` 값에 따라 아이콘의 각도를 다르게 보여주고 싶을 때 위와 같이 작성할 수 있다. 하지만 이는 잘못된 방법이다. 이런 코드는 부자연스러운 애니메이션을 만들게 된다. 이 코드는 SwiftUI에게 서로 다른 뷰 중 하나를 선택하게 하는 것이고 이는 곧 뷰의 추가와 삭제를 의미한다. 뷰의 추가와 삭제는 fade 애니메이션이 적용되기에 부자연스러운 애니메이션을 보게 되는 것이다. 

우리가 원하는 자연스러운 애니메이션을 위해선 다음과 같이 코드를 작성해야 한다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-26 at 10.26.52 PM.png)

여기서 얻을 수 있는 교훈은 이런 조건에 따라 다른 값에 의한 뷰의 변화를 부드러운 애니메이션을 통해 제공하기 위해선 최대한 이를 변경자 내부에 위치시켜 SwiftUI가 변화를 감지하여 보다 부드러운 애니메이션을 제공하도록 해야 한다는 것이다.

또한 비대해진 `OrderHistory`를 우린 더 작은 뷰로 나누어 관리할 수도 있다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-26 at 10.35.51 PM.png)

만일 `OrderHistory`에 조건에 따라 또 다른 뷰가 추가되어야 한다면 코드를 어떻게 작성해야할까

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-26 at 10.37.48 PM.png)

위와 같은 방법은 확장성이 매우 떨어진다. 우리는 이런 상황에서 `ForEach` 뷰를 사용할 수 있다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-26 at 10.38.27 PM.png)

`ForEach`는 하나의 뷰로 `List`와 마찬가지로 콜렉션 데이터 타입을 인자로 받는다. 그리고 뷰 빌더 클로저 안에 뷰를 나열하는데 이때 나열된 뷰는 `ForEach`에 추가되지 않고 `ForEach`의 상위 뷰에 추가된다.

지금까지 작성된 코드들을 보면 우리가 직접 작성하지 않고도 SwiftUI가 스스로 그리고 반응하며 갱신하는 것을 확인할 수 있었다. 이것이 바로 선언형 코드의 장점이라 할 수 있다.



## Composing Controls

아보카도 토스트 주문을 넣는 화면을 다시 살펴보자. 이는 우리가 알고 있는 화면과 많이 다르다. 정확히 말하자면 정형화되지 않은 상태다. 이 뷰를 아래와 같이 우리가 익숙한 형태의 뷰로 변경해보자. 

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-26 at 11.45.10 PM.png)

둘의 가장 큰 차이점은 컨테이너가 다르다는 것이다. 

기존 뷰(왼쪽)의 컨테이너가 `VStack`이라면 우리가 익숙한 오른쪽 뷰의 컨테이너는 `Form`이다. `Form` 역시 뷰 컨테이너의 한 종류다. `VStack`과의 차이점에는 헤더, 섹션 등이 있어 보다 정형화된 그룹 스타일의 UI를 보다 쉽게 만들 수 있다.

그리고 이렇게 컨테이너가 바뀜에 따라 그 안에 속하는 컨트롤(버튼, 토글 등)도 그 모습이나 속성이 컨테이너에 따라 변한다. 또한 `Form`을 사용하면 서로 다른 플랫폼에서 다양한 룩앤필(Look and Feel)을 제공할 수 있다. 이렇게 SwiftUI가 UI를 그리기 때문에 우리는 기능에 보다 집중할 수 있다.

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-27 at 8.36.26 PM.png)

위의 화면에서 `Button`을 예로 들면 뷰 컨테이너가 바뀌면서 `Button`의 `padding`, `alignment` 등이 바뀐 것을 확인할 수 있다. 

이번엔 `Button` 코드를 살펴보자. 

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-27 at 8.41.15 PM.png)

위의 단일 코드로 여러 플랫폼에서 다양한 룩앤필을 제공할 수 있다. 

`Button`은 눌렸을 때 액션을 인자로 넣어주고 버튼의 상태와 목적을 나타내는 `label`을 뷰 빌더 클로저를 통해 제공해줄 수 있다. 그리고 앞에서 봐왔듯이 여러 변경자들을 통해 보다 쉽게 커스터마이징을 할 수 있다. 이를 통해 우리는 다양한 플랫폼의 다양한 버튼을 사용자에게 제공해줄 수 있다.

그렇기 때문에 SwiftUI에서 컨트롤은 적응형(adaptive) 컨트롤이라 할 수 있다. 적응형 컨트롤은 다음과 같은 특성을 갖는다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-27 at 8.48.55 PM.png)

컨트롤은 그 자체로 모양이 아닌 역할을 나타낸다. 이렇게 컨트롤이 역할을 의미하기 때문에 여러 플랫폼에 거쳐 재사용될 수 있는 것이다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-27 at 8.51.27 PM.png)

이렇게 컨트롤들은 역할이 있고 이런 역할은 목적에 의해 생겨나기 때문에 `Toggle`이나 `Button`들은 그들 각자의 목적이 존재한다. 그리고 이들은 사람이 읽을 수 있는 레이블을 포함하기 때문에 기본적으로 VoiceOver 기능을 지원한다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-27 at 8.55.38 PM.png)

그리고 레이블이 `Text`가 아니라 `Image`어도 `Image`에 설명을 위한 `Text`를 함께 제공하여 VoiceOver 기능을 제공할 수 있다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-27 at 8.57.43 PM.png)

또한 커스텀 뷰는 `accessbility` 변경자를 통해 이런 기능을 제공할 수 있다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-27 at 8.58.14 PM.png)

이렇게 컨트롤은 플랫폼에 따라 모양은 다를 수 있지만 본연의 목적을 수행하는 데 이는 SwiftUI의 핵심이라고 할 수 있다. SwiftUI는 한 번만 작성하고 어디에서나 실행할 수 있는 수단 일뿐만 아니라 이러한 핵심 개념을 배우고 다양한 컨텍스트와 플랫폼에서 사용할 수 있는 프레임 워크다.

그리고 우리가 뷰에서처럼 컨트롤에서도 변경자를 사용할 수 있다. 그리고 이는 뷰에서와 동일한 특성을 갖는다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-27 at 9.19.51 PM.png)

예를 들어 다음과 같이 컨트롤 계층 전반에 걸쳐 변경자를 공유할 수 있다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-27 at 9.20.48 PM.png)

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-27 at 9.20.52 PM.png)

다음으로 살펴볼 것은 환경(Environment)이다. 이는 일종의 모든 뷰에서 접근할 수 있는 특성의 집합으로 볼 수 있다. 그리고 자식 뷰는 부모 뷰의 환경 특성을 상속 받는다. 물론 필요에 따라 자식 뷰에서 이를 오버라이딩할 수 있다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-27 at 9.26.53 PM.png)

그리고 이 환경은 프리뷰에서 유용하게 사용되는데, 동일한 UI를 여러 문맥에 따라 다르게 보여주는 기능을 제공한다. 이를 통해 환경, 문맥에 따라 UI가 어떻게 바뀌는지 쉽게 확인할 수 있다. 

## Navigating your app

iOS에선 기본적으로 `NavigationView`를 통해 기본 내비게이션 스타일을 사용할 수 있으며 `navigationBarTitle` 변경자를 통해 타이틀을 지정할 수 있다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-27 at 9.35.39 PM.png)

`navigationBarTitle`은 다른 변경자와 같이 아래를 향하지 않고 위를 향하는 특성을 갖는다. `OrderForm`에 변경자를 적용했지만, `NavigationView`에 반영된다는 것을 의미한다. 

그리고 `NavigationButton`를 목적지와 함께 만들어 실질적인 화면 전환을 구현할 수 있다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-27 at 9.40.37 PM.png)

`TabbedView`를 통해 성격이 다른 두 뷰를 탭 뷰로 묶어 관리할 수도 있다. 

![](https://ehdrjsdlzzzz.github.io/2019/12/26/SwiftUI-Essentials-2/Screen Shot 2019-12-27 at 9.41.37 PM.png)