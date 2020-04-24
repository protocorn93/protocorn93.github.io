---
title: App Life Cycle
date: 2018-11-06 23:26:09
tags: ["iOS", "Swift", "UIApplication"]
category: ["iOS"]
---

우리는 iOS를 공부하면서 앱 생명주기는 반드시 알아야 하는 주제 중 하나다. 이번 포스팅에선 단순한 앱 상태의 변화, 뷰 컨트롤러의 생명주기가 아닌 앱의 생명주기를 공부해 볼 것이다. 이는 앱 아이콘을 터치하여 앱을 실행시키는 것부터 시작한다. 

C 언어를 기반으로 하는 모든 프로그램의 시작점은 `main` 함수이며 이는 iOS도 다를 것이 없다. 다른 점이라면 iOS에선 이런 `main` 함수를 직접 작성해줄 필요가 없다는 것이다.  Xcode가 이를 프로젝트의 기본적인 한 부분으로써 프로젝트 생성 시 함께 생성해주기 때문이다. 

```objective-c
#import <UIKit/UIKit.h>
#import "AppDelegate.h"
 
int main(int argc, char * argv[])
{
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```

`main` 함수는 내부적으로 `UIApplicationMain`이라는 함수를 한 번 더 호출한다. 이 함수는 앱의 핵심 객체들을 생성하며 앱 런칭을 시작한다. 이렇게 생성되는 중요한 객체들을 몇 가지 살펴보도록 하자. 제일 먼저 [`UIApplication`](https://developer.apple.com/documentation/uikit/uiapplication) 객체이다. 

---

### UIApplication

`UIApplicationMain` 함수는 가장 먼저 `UIApplication` 객체를 생성한다. 이 클래스를 공식문서에선 다음과 같이 설명하고 있다. 

*"The centralized point of control and coordination for apps running in iOS."*

*"iOS에서 실행 중인 앱들의 조정과 제어의 중심적인 역할을 하는 클래스."*

`UIApplicationMain`에 의해 생성된 `UIApplication` 객체는 싱글톤의 형태로 이를 만들고 `shared`를 통해 접근이 가능하다. 이 객체의 가장 중요한 역할은 사용자 이벤트의 초기 라우팅을 처리해주는 것이다. 이는 이벤트 액션 메시지를 컨트롤 객체 ([`UIControl`](https://developer.apple.com/documentation/uikit/uicontrol))들을 사용해 적절한 타겟 객체에게 (이벤트를 처리해야 할) 전달한다. 그리고 이 객체는 윈도우들의 목록을 유지하고 있기 때문에 앱의 어떤 뷰 객체에도 접근이 가능하다. 

<img src="https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Art/event_draw_cycle_a_2x.png">

그리고 `UIApplication` 클래스는 [`UIApplicationDelegate`](https://developer.apple.com/documentation/uikit/uiapplicationdelegate) 프로토콜을 채택한 델리게이트를 정의한다. 그리고 이 델리게이트에 실행 중 발생하는 중요한 이벤트들을 전달하는데 그 종류에는 앱 런치, 로우 메모리 경고 그리고 앱 종료와 같은 앱 상태 변화와 같은 것들이 있고 델리게이트에선 각각의 메소드 안에 적절한 행위들을 작성해주면 된다.

`main` 함수도 프로젝트 생성 시 함께 생성된 것처럼 `UIApplicationDelegate`를 따르는 델리게이트 클래스도 같이 생성되는데 그 앞에는 `@UIApplicationMain`라는 어노테이션이 붙고 UIKit은 이것으로 `UIApplication` 객체의 델리게이트임을 인식하고 둘을 연결한다. 그럼 `UIApplicationDelegate` 프로토콜을 좀 더 자세히 살펴보도록 하자.

---

### UIApplicationDelegate

공식 문서에선 이를 다음과 같이 설명하고 있다. 

*"A set of methods that are called by the singleton [`UIApplication`](https://developer.apple.com/documentation/uikit/uiapplication) object in response to important events in the lifetime of your app."*

여기서 말하는 중요한 이벤트를 대응한다는 것은 앱이 포그라운드에서 백그라운드로 이동하거나 수신되는 노티피케이션에 대응하는 것과 같은 것들을 말한다. 앱의 상태 변화는 다음과 같다.

<img src="https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Art/high_level_flow_2x.png">

앱 델리게이트는 사실상 앱의 루트 객체이다. `UIApplication` 객체가 그랬던 것처럼 이 역시 싱글톤의 형태로 실행 중인 앱의 전역에서 접근이 가능하다. 비록 `UIApplication` 객체가 앱 관리에 있어서 대부분의 작업을 하지만 앱 델리게이트 메소드들의 구현체를 적절히 제공해주어야 한다. 앱 델리게이트는 또한 다음과 같은 몇몇 중요한 역할을 수행한다. 

- 앱의 시작 코드를 포함한다.
- 앱의 상태 전환에 대응한다.
- 앱의 외부에서 들어오는 노티피케이션을 대응한다. 
- 상태의 보존 및 복원이 이루어져야하는지 여부를 결정하고 필요에 따라 보존 및 복원 프로세스를 지원한다.
- 앱의 뷰나 뷰 컨트롤러에서 처리되지 않은 이벤트에 대응한다. 
- 뷰 컨트롤러가 소유하지 않는 앱의 중심 데이터 객체 혹은 다른 컨텐츠들을 저장하는데 사용할 수 있다. 

런치 시점에 앱 델리게이트는 앱을 초기화하는데 필요한 사용자 정의 코드(개발자가 작성한)를 실행시켜야 하는 책임을 갖고 있다. 예를 들어 앱 델리게이트는 일반적으로 앱의 초기 데이터 구조를 생성하고 필요한 서비스를 등록하며 사용 가능한 데이터에 기반하여 앱의 초기 사용자 인터페이스를 조정하기도 한다. (토큰 유무에 따른 로그인 화면 혹은 메인 화면)



이렇게 `UIApplication`과 `UIApplicationDelegate`를 살펴보았는데 이를 도식화하면 다음과 같다. 

<img src="https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Art/core_objects_2x.png">

순서대로 정리해보자면 다음과 같다.

1. 사용자가 앱 아이콘을 탭
2. 앱의 `main` 함수가 호출
3. `main`은 `UIApplicationMain`을 호출
4. `UIApplicationMain`이 호출되어 `UIApplication` 객체를 생성
5. `UIApplicationDelegate` 프로토콜을 체택한 객체도 생성
6. 앱 델리게이트의 `applicationDidFinishLaunching` 메소드 호출 
7. `UIApplication` 객체는 메인 런 루프를 실행시켜 이벤트를 받을 준비
8. 개발자가 작성한 코드 실행

이렇게 앱의 시작 과정과 생명 주기에 대해 간단히 살펴보았다. 이러한 순서와 관계를 알게 되면 어떤 코드를 어디에 위치시켜야 할지에 대한 감이 조금 생기게 된다. 이를 활용해 적절한 곳에 적절한 코드를 위치시켜보자.

> 부족한 부분이나 틀린 내용이 있다면 ehdrjsdlzzzz@gmail.com으로 피드백 부탁드립니다.😊

---

참고자료

1. [UIApplication](https://developer.apple.com/documentation/uikit/uiapplication)

2. [UIApplicationDelegate](https://developer.apple.com/documentation/uikit/uiapplicationdelegate)
3. [App Programming Guide for iOS](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/TheAppLifeCycle/TheAppLifeCycle.html#//apple_ref/doc/uid/TP40007072-CH2-SW1)

