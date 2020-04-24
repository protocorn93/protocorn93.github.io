---
title: About Scene
subtitle: Scene과 관련된 클래스와 프로토콜 살펴보기
date: 2019-08-21 21:48:02
tags: ["UIScene", "UIWindowScene", "UISceneDelegate", "UIWindowSceneDelegate", "UISceneSession", "UISceneConfiguration"]
category: ["iOS 13"]
---

> iOS 13부터 등장한 새로운 생명주기 개념을 이해하기 위해 `Scene`에 대해 공식 문서와 함께 관련 WWDC 영상을 보면서 정리한 내용입니다. 
>
> **Scene과 관련 클래스들을 이해하는데 집중했으며 생명주기는 다루고 있지 않습니다.**
>
> 영상 : [Introducing Multiple Windows on iPad](https://developer.apple.com/videos/play/wwdc2019/212/)
>
> 예제 코드 : [Supporting Multiple Windows on iPad](https://developer.apple.com/documentation/uikit/app_and_environment/scenes/supporting_multiple_windows_on_ipad)



먼저 `Scene`의 등장으로 앱 델리게이트에서부터 UI 관련 부분을 분리할 수 있었다. 

![](https://ehdrjsdlzzzz.github.io/2019/08/21/About-Scene/7.png)



### UIScene

---

앱의 UI의 인스턴스 하나를 나타내는 객체 

UIKit은 사용자 또는 앱이 요청하는 앱 UI의 각 인스턴스에 해당하는 `Scene`(이하 씬) 객체를 생성한다. 일반적으로 UIKit은 `UIScene` 객체 대신 `UIWindowScene` 객체를 생성하지만 이 클래스의 메소드와 프로퍼티를 통해 씬에 대한 정보에 접근할 수 있다. 

모든 씬 객체는 연관된 델리게이트 객체를 갖고 있으며, 그 델리게이트 객체 [`UISceneDelegate`](https://developer.apple.com/documentation/uikit/uiscenedelegate) 프로토콜을 따르고 있다. 씬의 상태가 변경되면 씬 객체는 이를 델리게이트 객체에 알리고 등록된 옵저버 객체에 적절한 노티피케이션을 보낸다. 델리게이트 객체와 노티피케이션을 이용해서 씬 상태 변화에 적절히 대응해야 한다. 예를 들어 이들을 사용해서 씬이 백그라운드로 이동한 시점에 대응해줄 수 있다. 

씬 객체를 직접 생성해선 안된다. `UIApplication`의 메소드인 [`requestSceneSessionActivation(_:userActivity:options:errorHandler:)`](https://developer.apple.com/documentation/uikit/uiapplication/3197901-requestscenesessionactivation)를 호출해서 UIKit에게 앱의 새로운 씬의 생성을 요청할 수 있다. 또한 UIKit은 사용자의 상호작용에 반응하여 씬을 생성하기도 한다. 앱의 씬 지원을 구성할 때 `UIScene` 객체 대신 [`UIWindowScene`](https://developer.apple.com/documentation/uikit/uiwindowscene)을 명시해야 한다. 

>  위에서 언급한 앱과 사용자와의 상호작용의 예는 다음과 같다. 
>
> - Dock에서 앱 아이콘을 끌어다가 현재 보여지고 있는 앱의 테두리에 가져다 놓는다. 
> - 앱 내 링크를 앱 테두리에 가져다 놓는다. (Drap and Drop)
>
> 등등

`UIScene`에 대해 WWDC 영상에서 추가적으로 설명하고 있는 부분은 다음과 같다. 

![](https://ehdrjsdlzzzz.github.io/2019/08/21/About-Scene/6.png)



### UISceneDelegate

---

씬에서 발생하는 생명주기 이벤트에 대응하는데 사용하는 핵심 메소드를 포함하고 있는 프로토콜

`UISceneDelegate` 객체를 사용해서 앱 UI 중 하나의 인스턴스의 생명주기 이벤트를 관리한다. 이 인터페이스는 씬에 영향을 주는 상태 전환에 대응하는 메소드들을 정의하고 있다. 이러한 상태 전환에는 포그라운드로 이동, 활성 상태로의 변화 그리고 백그라운드로의 이동 등이 존재한다. 이러한 상태 전환이 발생했을 때의 적절한 행동을 이 델리게이트를 사용해 정의해준다. 예를 들어 앱이 백그라운드로 들어갔을 때 중요한 작업을 끝내고 앱을 종료시키는 것과 같은 행동을 정의해줄 수 있다. 

`UISceneDelegate` 객체를 직접 생성해선 안된다. 대신 씬을 구성하는데 사용하는 구성 정보의 일부로 사용자 정의 델리게이트 클래스 이름을 명시해야 한다. `Info.plist`에 명시하거나

![2.png](https://ehdrjsdlzzzz.github.io/2019/08/21/About-Scene/2.png)

앱 델리게이트 메소드인 [`application(_:configurationForConnecting:options:)`](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/3197905-application)가 반환하는 [`UISceneConfiguration`](https://developer.apple.com/documentation/uikit/uisceneconfiguration) 객체에 명시할 수 있다. 



### UIWindowScene

---

`UIWindowScene` 객체는 씬이 보여주고 있는 하나 혹은 그 이상의 윈도우를 포함한 앱의 UI 인스턴스 하나를 관리한다. 

![1.png](https://ehdrjsdlzzzz.github.io/2019/08/21/About-Scene/1.png)

씬 객체는 사용자 디바이스 상에서 보여주고 있는 윈도우와 사용자와 상호작용하는 그 씬의 생명주기를 관리한다. 씬의 상태가 변하면, 씬 객체는 이를 [`UIWindowSceneDelegate`](https://developer.apple.com/documentation/uikit/uiwindowscenedelegate) 프로토콜을 따르고 있는 델리게이트 객체에 알린다. 또한 씬은 등록된 옵저버들에 적절한 노티피케이션을 보내기도 한다. 이 옵저버 객체들을 사용해서 이런 상태 변화에 대응할 수 있다.



윈도우 객체를 직접 생성해선 안된다. 대신 원하는 `UIWindowScene` 객체의 클래스 이름을 구성 시점에 `Info.plist` 파일의 씬 구성 상세 항목에 포함시키는 방법을 사용해야 한다. 

![2.png](https://ehdrjsdlzzzz.github.io/2019/08/21/About-Scene/2.png)

또한 앱 델리게이트 메소드인 [`application(_:configurationForConnecting:options:)`](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/3197905-application) 안에서 [`UISceneConfiguration`](https://developer.apple.com/documentation/uikit/uisceneconfiguration)을 생성할 때 클래스 명을 명시해줄 수 있다.

![3.png](https://ehdrjsdlzzzz.github.io/2019/08/21/About-Scene/3.png)

사용자가 앱과 상호작용할 때, 시스템은 개발자가 제공한 구성 정보를 바탕으로 적절한 씬을 생성한다. 씬을 코드로 생성하기 위해선 UIApplication 메소드인 [`requestSceneSessionActivation(_:userActivity:options:errorHandler:)`](https://developer.apple.com/documentation/uikit/uiapplication/3197901-requestscenesessionactivation)를 호출해야 한다. 



### UIWindowSceneDelegate

---

씬에서 발생하는 앱에 특정한 작업을 관리하기 위해 사용하는 추가적인 메소드를 포함하는 프로토콜로 `UISceneDelegate`를 따르고 있다.

`UIWindowSceneDelegate` 객체를 사용해서 앱 UI 중 하나의 인스턴스의 생명주기 이벤트를 관리한다. 이는 `UISceneDelegate`를 따르고 있어 씬이 앱에 연결되거나, 포그라운드에 들어가는 것과 같은 이벤트에 대한 노티피케이션에 대응하는데 사용할 수 있다. 또한 이를 사용해서 씬의 기본 환경에서 발생하는 변화에 대응할 수 있다. 예를 들어 사용자가 씬의 크기를 재조정하면, 이 델리게이트를 사용해서 새로운 씬 사이즈에 맞게 컨텐츠를 조정하는 등의 행위를 할 수 있다.

`UIWindowSceneDelegate` 객체를 직접 생성해선 안된다. 대신 씬을 구성하는데 사용하는 구성 정보의 일부로 사용자 정의 델리게이트 클래스 이름을 명시해야 한다.  Info.plist에 명시하거나 앱 델리게이트 메소드인 [`application(_:configurationForConnecting:options:)`](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/3197905-application)가 반환하는 [`UISceneConfiguration`](https://developer.apple.com/documentation/uikit/uisceneconfiguration) 객체에 명시할 수 있다. 

어떻게 씬을 구성하는지에 대한 자세한 정보는 [Specifying the Scenes Your App Supports](https://developer.apple.com/documentation/uikit/app_and_environment/scenes/specifying_the_scenes_your_app_supports) 문서를 참고하길 바란다. 

>  이에 대해서도 과거(?)에 정리해보았다. 
>
> [정리한 글](https://ehdrjsdlzzzz.github.io/2019/06/06/Specifying-the-Scenes-Your-App-Support/)



### UISceneConfiguration

---

UIKit이 특정 씬을 생성할 때 사용하는 객체와 스토리보드에 관한 정보를 담고 있다. 

UIKit이 앱의 새로운 씬을 생성할 때 사용할 수 있는 정보를 명시하기 위해선 `UISceneConfiguration` 객체를 사용해야 한다. 특히 생성을 원하는 씬의 클래스, 앱이 그 씬을 관리하는데 사용하는 씬 델리게이트 클래스 그리고 씬의 초기 뷰 컨트롤러를 포함하고 있는 스토리보드를 반드시 포함해야 한다. 

>  스토리보드를 사용하지 않고 코드를 구성하는 방법 
>
> [Apple’s New UIScene API: A Programmatic Approach](https://medium.com/@ZkHaider/apples-new-uiscene-api-a-programmatic-approach-52d05e382cf2)

사용자가 앱의 새로운 UI를 요청할 때, UIKit은 해당하는 씬 객체를 생성하는데 필요한 구성 정보를 `Info.plist`에서 찾는다. 그리고 찾았다면 그 정보들을 `UISceneConfiguration` 객체로 포장해서 세션의 일부로 앱 델리게이트의 메소드인 [`application(_:configurationForConnecting:options:)`](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/3197905-application)에 전달한다.

![4.png](https://ehdrjsdlzzzz.github.io/2019/08/21/About-Scene/4.png)

세번째 이미지와 비교해서보면 세번째 이미지는 문자열로 이름을 명시했고, 위의 이미지에선 `connectingSceneSession.configuration.name`으로 전달하였다. 실행시켰을 때 동일한 씬을 확인할 수 있는데 그 이유는 두 번째 이미지에서 확인할 수 있는 `Info.plist` 파일에서 문자열과 동일한 이름을 명시했기 때문이다. 

이렇게 전달된 구성 정보를 그대로 사용할 수 있지만 다른 상세 구성 정보를 더해 새로운 `UISceneConfiguration` 객체를 생성해 반환할 수도 있다. 



### UISceneSession

---

앱의 여러 씬들 중 하나의 씬에 대한 정보를 담고 있는 객체 

`UISceneSession`은 씬의 고유한 런타임 인스턴스를 관리한다. 사용자가 앱에 새로운 씬을 추가하거나, 개발자가 코드로 새로운 씬을 요청했을 때 시스템은 그 씬을 추적하기 위해 세션 객체 하나를 생성한다. 이렇게 생성된 세션은 고유한 식별자 값과 그 씬의 상세 구성 정보를 포함하고 있다. UIKit은 세션이 추적하고 있는 그 씬이 살아있는 동안은 세션을 유지하고, 사용자가 앱 스위처에서 해당 씬을 닫으면 그에 따라 세션을 제거한다. 

세션 객체를 직접 생성해선 안된다. 앱과 사용자의 상호작용에 반응하여 UIKit이 세션을 생성한다. 또한 코드로 `UIApplication`의 메소드인  [`requestSceneSessionActivation(_:userActivity:options:errorHandler:)`](https://developer.apple.com/documentation/uikit/uiapplication/3197901-requestscenesessionactivation)를 호출해서 UIKit에게 새로운 씬과 세션을 생성하도록 요청할 수 있다. UIKit Info.plist의 내용을 바탕으로 기본 구성 데이터를 사용해 세션을 생성한다. 

추가적으로 세션은 인터페이스의 상태를 저장하며, 정해진 시스템상에서의 역할이 존재한다. 다음은 이에 대해서 설명하고 있는 WWDC 영상에서 사용하고 있는 슬라이드 내용의 일부다. 

상태 저장은 씬 생명주기에서 굉장히 중요한 역할이다. 

![5.png](https://ehdrjsdlzzzz.github.io/2019/08/21/About-Scene/5.png)

그리고 다음과 같이 앱 스위처에서 보이는 동일한 앱의 씬들은 서로 다른 세션에 의해 관리된다.

![8.png](https://ehdrjsdlzzzz.github.io/2019/08/21/About-Scene/8.png)