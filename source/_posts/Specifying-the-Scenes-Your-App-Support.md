---
title: Specifying the Scenes Your App Support
date: 2019-06-06 17:02:43
tags: ["Scene", "WWDC19", "iOS 13"]
category: ["iOS"]
---

WWDC19에서 iOS 13이 발표되고 새로운 것들이 다수 생겼다. 바인딩을 지원하는 `Combine`, 선언형 UI 방식으로 UI를 구현할 수 있는 `SwiftUI` 프레임워크까지 많은 것들이 나왔다. 

이와 함께 앱 생명주기와 관련된 새 공식 문서가 등장했는데 여기서 `Scene`이라는 개념이 등장한다. [Managing Your App's Life Cycle](https://developer.apple.com/documentation/uikit/app_and_scenes/managing_your_app_s_life_cycle) 문서를 살펴보기 전에 [Specifying the Scenes Your App Support](https://developer.apple.com/documentation/uikit/app_and_scenes/specifying_the_scenes_your_app_supports) 문서를 먼저 살펴보며 Scene의 등장에 대한 이유를 이해하고자 한다. 

> 참고로 아직은 베타 문서라 추후에 내용이 변경되거나 사용되는 클래스, 프로토콜 그리고 메소드 등의 이름이 변경될 수 있다.



### Overview

------

iOS 13과 그 이상에서는 사용자가 앱 UI의 사본을 여러 개 만들어 앱 스위처 내에서 서로 전환할 수 있다. iPad에서 사용자는 또한 앱 UI의 사본과 사본을 나란히 디스플레이할 수 있다. 각각의 앱 UI 사본에 대해 Scene 객체를 사용하여 UI를 화면에 띄우는 윈도우, 뷰 그리고 뷰 컨트롤러를 관리한다. 

> WWDC19 키노트에서는 이를 Multi-Window Capability라 설명하며 동일한 메모 앱 두 개를 나란히 실행시키는 모습을 보여주었다. 아래는 키노트의 일부를 캡쳐한 영상이다.

![scenebasedkeynote](https://ehdrjsdlzzzz.github.io/2019/06/06/Specifying-the-Scenes-Your-App-Support/scenebasedkeynote.gif)



사용자가 새 Scene을 요청할 때, UIKit은 이에 해당하는 Scene 객체를 만들고 이것의 초기화 설정을 다룬다. 이를 위해 UIKit은 당신이 제공한 정보에 의존한다. 앱은 반드시 자신이 지원하는 Scene의 유형과 해당 Scene을 관리하는데 사용하는 객체를 선언해야 한다. 이 작업은 앱의 `Info.plist`에 정적으로 정의하거나, 런타임에 동적으로 정의할 수 있다.

> **중요**
>
> Scene을 앱에서 지원하는 것은 선택이지만, 앱의 UI 사본들을 동시에 보여주고 싶다면 반드시 지원해야 한다. 



### Enable Scene Support in Your Project Settings

------

앱은 앱의 구성 설정을 갱신하여 Scene을 명시적으로 선택해야 한다. 

1. Xcode 프로젝트를 연다.
2. `General Settings`로 이동한다.
3. `Deployment Info`  섹션에서 "Support multiple windows" 체크박스를 활성화한다. 

멀티플 윈도우 옵션을 활성화하면 Xcode는 앱의 `Info.plist` 파일에 [UIApplicationSceneManifest](https://developer.apple.com/documentation/bundleresources/information_property_list/uiapplicationscenemanifest) 키를 추가한다. 이 키의 존재는 시스템에게 당신의 앱이 Scene을 지원한다는 사실을 전달한다. 이 키의 값은 딕셔너리로 초기에는 오직 [`UIApplicationSupportsMultipleScenes`](https://developer.apple.com/documentation/bundleresources/information_property_list/uiapplicationscenemanifest/uiapplicationsupportsmultiplescenes) 키만 포함하고 있는 상태다. 

`UIApplicationSupportMultipleScenes` 키의 값은 당신의 앱이 실제로 동시에 여러 Scene을 지원하는지를 시스템에게 알린다. Xcode는 이 값을 기본적으로 `true`로 지정한다. 하지만 한 번에 하나의 Scene만 보여주고 싶으면 비활성화하면 된다. 멀티플 Scene을 지원하기 위해선 서로 다른 Scene들이 서로를 침범하지 않도록 방지하기 위한 추가 작업이 필요하다. 예를 들어 Scene에서 동일한 공유 데이터 구조를 사용하는 경우 앱 데이터 무결성을 유지하기 위해 해당 구조에 대한 접근을 조정해야 한다. 



### Configure the Details for Each Scene 

------

UIKit은 당신이 제공한 정보를 사용해 앱의 Scene 생성을 담당한다. 가장 간단한 방법은 이 정보를 앱의 `Info.plist`를 사용하는 것이다. 

1. Xcode를 열고 `Info.plist` 파일을 선택한다. 
2. `Application Scene Manifest` 항목의 (+) 버튼을 누른다. 이 항목은 `UIApplicationSceneManifest` 키에 해당한다. 없는 경우 프로젝트 설정에서 위에서 언급한 대로 이를 추가하면 된다. 
3. 메뉴가 등장하면 `Scene Configuration`을 선택한다. 
4. `Scene Configuration` 항목에서 (+) 버튼을 클릭한다.
5. 당신의 앱에 메인 Scene을 추가하기 위해 `Application Session Role`을 선택한다. 
6. 제공된 항목에 Scene의 상세 정보를 기입한다. 

대부분의 앱은 오직 하나의 메인 Scene만 필요하지만 멀티플 Scene을 추가하고 각각을 다르게 구성할 수 있다. 예를 들어 알림과 관련된 컨텐츠를 보여주기 위한 두 번째 Scene을 포함시킬 수 있다. UIKit은 각 Scene에 대해 다음 정보를 필요로 한다.

- [`UIWindowScene`](https://developer.apple.com/documentation/uikit/uiwindowscene) 클래스의 이름
- 당신의 앱이 Scene을 관리하는데 사용하는 사용자 정의 델리게이트 객체 클래스의 이름. 이 클래스는 반드시  [`UIWindowSceneDelegate`](https://developer.apple.com/documentation/uikit/uiwindowscenedelegate) 프로토콜을 따라야 한다. 
- 앱에서 Scene을 내부적으로 식별하는데 사용하는 고유한 이름
- Scene의 초기 UI를 포함하는 스토리보드의 이름. `.storyboard` 파일 확장자를 제외한 이름을 명시한다. 

Scene을 구성하는데 필요한 정보 추가적인 정보는 [UISceneConfigurations](https://developer.apple.com/documentation/bundleresources/information_property_list/uiapplicationscenemanifest/uisceneconfigurations)를 참고하라.



### Create the Interface for Your Scene 

------

스토리보드를 사용해 Scene의 UI를 지정한다.  [`UISceneStoryboardFile`](https://developer.apple.com/documentation/bundleresources/information_property_list/uiapplicationscenemanifest/uisceneconfigurations/uiwindowscenesessionroleapplication/uiscenestoryboardfile) 키에 지정한 스토리보드는 Scene을 보여주는데 사용되는 초기 뷰 컨트롤러를 포함한다. Scene 객체를 생성하는 것 외에도 UIKit은 Scene에 대한 윈도우를 생성하고 Scene의 스토리보드에서 초기 뷰 컨트롤러 지정한다. [`UIWindowSceneDelegate`](https://developer.apple.com/documentation/uikit/uiwindowscenedelegate) 객체의 메소드를 사용해서 코드로 이 뷰 컨트롤러를 교체할 수 있다. 

> **중요**
>
> 스토리보드의 초기 뷰 컨트롤러를 지정하는 것을 잊지 말아야 한다. UIKit는 UI를 구성할 때 이 뷰 컨트롤러의 존재에 의존한다. 



### Change Your Scene's Configuration Dynamically

------

실제로 Scene 객체를 생성하기 전에 UIKit은 앱 델리게이트의 메소드인 [`application(_:configurationForConnecting:options:)`](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/3197905-application)를 호출하여 당신이 Scene과 연관된 상세 정보를 수정할 수 있도록 한다. 이 방법을 사용하여 UIKit에서 제공하는 옵션에 따라 Scene 구성을 조정할 수 있다. 예를 들어 시스템이 Scene에 알림 응답(Notification response)을 전달할 때 알림과 연관된 인터페이스와 함께 다른 스토리보드를 지정할 수 있다. 

동적으로 Scene 구성을 구현하지 않으면 UIKit은 Scene을 생성하는데 `Info.plist`의 정보를 사용한다. 



### Adopt Scene-Based Life-Cycle Semantices

------

Scene에 대한 지원을 추가하면 앱이 생명 주기 이벤트에 대응하는 방식이 변경된다. Scene을 사용하지 않는 앱에선 앱 델리게이트 객체가 포그라운드 혹은 백그라운드로의 전환을 담당한다. 앱에서 Scene을 지원하게 되면 UIKit은 이러한 책임을 당신이 지정한 Scene 델리게이트 객체에 위임한다. Scene 생명 주기는 다른 Scene에 독립적이고, 앱 자체와도 독립적이다. 그러므로 당신이 지정한 Scene 델리게이트 객체가 이러한 전환을 담당해야 한다. 

만일 앱이 iOS 12를 지원한다면 앱 델리게이트와 Scene 델리게이트 객체 모두에서 생명 주기 전환을 처리할 수 있다. UIKit은 오로지 하나의 델리게이트 객체에만 생명 주기 관련 이벤트 알림을 보낸다. iOS 13 이상에선 UIKit은 Scene 델리게이트 객체에 알림을 보내고, iOS 12 이하에선 UIKit은 앱 델리게이트에 해당 알림을 보낸다. 

생명 주기 이벤트를 어떻게 다루는지에 대한 추가적인 정보는 [Managing Your App's Life Cycle](https://developer.apple.com/documentation/uikit/app_and_scenes/managing_your_app_s_life_cycle)를 참고하라.