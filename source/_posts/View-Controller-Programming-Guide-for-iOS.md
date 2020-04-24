---
title: View Controller Programming Guide for iOS
subtitle: Overview - The Role of View Controllers
date: 2018-09-24 17:28:32
tags: ["iOS", "UIViewController"]
category: ["iOS", "Programming Guide"]
---

### The Role of View Controllers

뷰 컨트롤러는 앱 내부 구조의 기반이 된다. 모든 앱은 최소 한 개 이상의 뷰 컨트롤러를 갖고, 많은 앱들이 여러 뷰 컨트롤러를 갖는다. 각각의 뷰 컨트롤러는 앱 UI의 부분을 관리뿐만 아니라 인터페이스와 그 밑의 데이터들 간의 상호작용을 관리하는 역할을 한다. 또한 뷰 컨트롤러는 UI 간의 전환을 수월하게 한다. 

뷰 컨트롤러는 앱에서 중요한 역할을 하기 때문에 당신이 하는 모든 작업의 중심이 된다. `UIViewController` 클래스는 뷰 관리, 이벤트 처리, 다른 뷰 컨트롤러의 전환 그리고 앱의 다른 부분과의 조합을 위한 메소드와 프로퍼티들을 정의하고 있다. 당신은 `UIViewController` (혹은 이를 상속받은 클래스)를 상속받아 원하는 앱의 동작을 구현하기 위해 사용자 정의 코드를 추가해줄 수 있다. 

두 가지 유형의 뷰 컨트롤러가 존재한다. 

- **컨텐트 뷰 컨트롤러**(*Content view controllers*)는 앱내의 분리된 컨텐츠 조각들을 관리하며 당신이 생성하는 뷰 컨트롤러의 주요 타입이 된다.
- **컨테이너 뷰 컨트롤러**(*Container view controllers*) 다른 뷰 컨트롤러들(자식 뷰 컨트롤러라 알려진)로부터 정보를 모아 탐색을 용이하게 하거나 컨텐츠를 각각의 뷰 컨트롤러에 다르게 보여주는 역할을 한다. 

대부분의 앱이 이 두 가지 유형의 뷰 컨트롤러들로 이루어져 있다. 



#### View Management

뷰 컨트롤러의 가장 중요한 역할은 뷰들의 계층을 관리하는 것이다. 모든 뷰 컨트롤러들은 뷰 컨트롤러 위의 컨텐츠들을 감싸고 있는 하나의 루트 뷰를 갖고 있다. 당신은 이 루트 뷰에 컨텐츠를 보여주는데 필요한 뷰를 추가해줄 수 있다. 다음 그림은 뷰 컨트롤러와 그의 뷰 사이의 관계를 그리고 있다. 

<img src="https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_ControllerHierarchy_fig_1-1_2x.png">

> [outlets](https://developer.apple.com/library/archive/documentation/General/Conceptual/Devpedia-CocoaApp/Outlet.html#//apple_ref/doc/uid/TP40009071-CH4)를 사용하는 것은 뷰 컨트롤러의 뷰 계층에 존재하는 다른 뷰들에 접근하는 일반적인 방법이다. 뷰 컨트롤러가 모든 뷰들의 컨텐츠를 관리하기 때문에 outlets을 사용한다면 당신이 필요한 뷰들의 참조를 저장할 수 있다. 뷰들이 스토리보드에서 로드되었을 때 outlets들은 그들 스스로 실제 뷰 객체들과 자동으로 연결된다. 

컨텐트 뷰 컨트롤러는 스스로 자신의 모든 뷰들을 관리한다. 컨테이터 뷰 컨트롤러는 자신이 소유하고 있는 뷰뿐 아니라 한 개 혹은 그 이상의 뷰 컨트롤러들을 관리한다. 컨테이너 뷰 컨트롤러는 자식 뷰 컨트롤러의 컨텐츠들을 관리하진 않는다. 오로지 자신의 루트 뷰와 컨테이너 디자인에 따라 사이즈와 위치만을 관리한다. 다음 그림은 스플릿 뷰 컨트롤러와 그 자식들간의 관계를 그리고 있다. 스플릿 뷰 컨트롤러는 자식 뷰들의 위치와 사이즈의 전반적인 부분을 관리하지만 하지만 자식 뷰 컨트롤러들만이 자신들의 뷰 위의 컨텐츠들을 관리할 수 있다.

<img src="https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_ContainerViewController_fig_1-2_2x.png"/>

뷰 컨트롤러의 뷰들을 관리하는 방법은 [Managing View Layout](https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/DefiningYourSubclass.html#//apple_ref/doc/uid/TP40007457-CH7-SW6) 문서를 참고



#### Data Marshaling 

뷰 컨트롤러는 앱의 데이터와 뷰들 사이의 중재자 역할을 한다. `UIViewController` 클래스의 메소드와 프로퍼티들을 사용한다면 앱의 시각적인 표현을 관리할 수 있다. `UIViewController`를 상속받을 때 서브클래스에 관리할 데이터들의 변수를 추가할 수 있다. 사용자 정의 변수를 추가하는 것은 다음 그림과 같은 관계가 생성되는데 다음 그림은 데이터들에 대한 참조와 데이터를 보여주기 위한 뷰들의 참조를 갖는 뷰 컨트롤러를 나타낸다. 

<img src="https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_CustomSubclasses_fig_1-3_2x.png" />

당신은 항상 뷰 컨트롤러와 데이터 객체들 내에서 책임을 깨끗하게 분리해야 한다. 데이터 구조의 무결성을 보장하기 위한 대부분의 로직은 데이터 객체들 내에 위치해야 한다. 뷰 컨트롤러는 뷰들로부터 들어오는 인풋을 검증하고 이 인풋을 데이터 객체가 필요로 하는 포맷으로 포장하여 넘기는데 여기서 실제 데이터를 관리하기 위한 뷰 컨트롤러의 역할을 최소화해야 한다.

`UIDocument` 객체는 뷰 컨트롤로부터 데이터를 분리하여 관리할 수 있는 방법 중 하나이다. 도큐먼트 객체는 컨트롤러 객체로 영구 저장소에 데이터를 쓰고 읽을 수 있는 방법을 알고 있다. `UIDocument` 클래스를 상속받을 때 데이터를 추출하고 이를 뷰 컨트롤러에서 앱의 다른 부분으로 넘기는 어떠한 로직이나 메소드들을 이곳에 추가해줄 수 있다. 뷰 컨트롤러는 뷰를 쉽게 갱신하도록 입력받은 데이터의 복사본을 저장할 수 있지만 도큐먼트 객체는 실제 데이터를 소유한다. 



#### User Interactions

뷰 컨트롤러는 응답자 객체로 응답자 체인을 통해 내려오는 이벤트들을 처리할 수 있다. 비록 그들이 그럴 수 있다 하더라도 뷰 컨트롤러가 직접 터치 이벤트를 처리하는 것은 드물다. 대신 터치 이벤트가 발생한 뷰가 이벤트를 처리하고 그 결과를 관련 델리게이트나 타겟 객체의 메소드에 전달되는데 주로 뷰 컨트롤러가 그 대상이 된다. 그렇기 때문에 뷰 컨트롤러의 대부분의 이벤트들은 델리게이트 메소드나 액션 메소드에 의해 처리된다. 뷰 컨트롤러에서 액션 메소드를 구현하는 방법에 대한 자세한 내용은 [Handling User Interactions](https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/DefiningYourSubclass.html#//apple_ref/doc/uid/TP40007457-CH7-SW11) 문서를 참고하라. 다른 유형의 이벤트를 처리하는 방법에 대한 자세한 내용은 [Using Responders and the Responder Chain to Handle Events](https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/using_responders_and_the_responder_chain_to_handle_events) 문서를 참고하라.



#### Resource Management

뷰 컨트롤러는 자신이 만든 모든 뷰나 객체들에 대한 책임을 떠맡는다. `UIViewController` 클래스는 뷰 관리의 대부분의 측면을 자동으로 처리한다. 예를 들어 UIKit은 자동으로 뷰와 관련된 자원 중 더 이상 필요하지 않은 자원을 릴리즈 한다. `UIViewController`의 서브 클래스에선 당신이 직접 만든 모든 객체를  관리해야 한다. 

만일 가용 메모리 용량이 작다면 UIKit은 앱에 더 이상 필요하지 않은 메모리를 릴리즈할 것을 요청한다. 이런 요청 행위를 명시해줄 수 있는 방법 중 하나는 뷰 컨트롤러 메소드인  `didReceiveMemoryWarning`에 작성하는 것이다. 이 메소드를 사용하여 더 이상 필요 없는 객체 참조를 제거할 수 있다. 예를 들어 캐시 데이터를 지우는 코드를 이곳에 작성해줄 수 있다. 메모리 부족 현상이 발생하면 최대한 많은 메모리들을 릴리즈해주는 것이 중요하다. 너무 많은 메모리를 사용하는 앱은 시스템에 의해 강제로 종료될 수 있다.



#### Adaptivity

뷰 컨트롤러는 자신의 뷰를 표현할 책임이 있고 기본으로 바탕이 되는 환경에 따라 그 표현을 조정할 책임이 있다. 모든 iOS 앱은 아이패드와 서로 다른 사이즈의 아이폰에서 돌아가야 한다. 각각의 디바이스마다 다른 뷰 계층과 뷰 컨트롤러를 제공하는 것보다 단일 뷰 컨트롤러와 변하는 공간의 요구 사항에 맞추어 뷰를 조정하는 것이 보다 간단하다.

iOS에서 뷰 컨트롤러는 급격한 변화와 미미한 변화 모두를 처리해야 할 필요가 있다. 급격한 변화는 뷰 컨트롤러의 **Traits**(특징)가 바뀌었을 때 발생한다. 특징은 디스플레이 스케일과 같이 전체적인 환경을 표현하는 속성들을 의미한다. 가장 중요한 두 가지 특징은 뷰 컨트롤러의 가로와 세로 크기로 뷰 컨트롤러가 주어진 공간 안에서 얼마나 많은 공간을 가지고 있는지 나타낸다. 다음 그림과 같이 당신은 크기의 변화를 이용하여 뷰들을 나열하는 방법을 바꿀 수 있다. 가로 크기가 *Regular*라면 뷰 컨트롤러는 컨텐츠를 나열할 추가적인 가로 공간을 확보할 수 있는 이점을 갖는다. 가로 크기가 *Compact*라면 뷰 컨트롤러는 컨텐츠를 수직 방향으로 나열한다. 

<img src="https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_SizeClassChanges_fig_1-4_2x.png">



주어진 크기 분류 내에서 언제든지 추가적인 미묘한 사이즈 변화가 발생할 수 있다. 사용자가 아이폰을 portrait에서 landscape로 회전시켰을 때 이 크기 분류는 변하지 않지만 화면의 공간은 변할 수 있다. 오토레이아웃을 사용한다면 UIKit은 자동으로 뷰의 사이즈와 위치를 새로운 공간에 맞추어 조정한다. 뷰 컨트롤러는 필요에 따라 추가로 조정할 수 있다.

Adaptivity에 대한 자세한 정보는 [The Adaptive Model](https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/TheAdaptiveModel.html#//apple_ref/doc/uid/TP40007457-CH19-SW1) 문서를 참고하시오.