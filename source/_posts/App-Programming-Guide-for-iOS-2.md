---
title: App Programming Guide for iOS (2)
subtitle: The App Life Cycle
date: 2018-08-22 15:33:33
tags: ["iOS", "Swift"]
category: ["Programming Guide"]
---

앱은 개발자가 작성한 코드와 시스템 프레임워크의 간의 정교한 상호작용이라 할 수 있습니다. 시스템 프레임워크는 모든 앱이 돌아가는데 필요한 기본적인 인프라를 제공하고 개발자는 이런 인프라를 바탕으로 코드를 추가, 수정하는 등의 행위를 통해 원하는 앱의 모습을 만들어갑니다. iOS의 인프라에 대한 이해와 이것이 어떻게 동작하는지에 대해 이해하고 있으면 이러한 개발 과정을 보다 효과적으로 수행할 수 있습니다. 

iOS 프레임워크는 MVC와 Delegation 패턴을 기반으로 하여 구현되어 있습니다. 성공적으로 앱을 만들기 위해서는 이러한 구조에 대한 이해가 뒷받침 되어야 합니다. 

---

#### The Main Function

C 언어를 기반으로 하는 모든 앱의 시작점은 바로 `main` 함수이고 iOS도 이와 크게 다를 것이 없습니다. 다른 점이라면 iOS에선 이런 `main` 함수를 직접 작성해줄 필요가 없다는 것입니다. 대신 Xcode가 이를 프로젝트의 기본적인 한 부분으로써 프로젝트 생성시 함께 생성해준 다는 것입니다.  

소수의 몇몇 경우를 제외하고선 Xcode에 의해 자동으로 생성된 `main` 함수를 수정해선 안됩니다.

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

`main` 함수에서 언급할 것은 이 함수의 역할이 바로 UIKit 프레임워크에 권한을 위임한다는 사실입니다. `UIApplicationMain` 함수는 앱의 핵심 객체들을 생성하고, 유효한 스토리보드 파일로부터 UI를 불러오며 개발자가 작성한 코드들을 호출하는데 이렇게 개발자가 작성한 코드들을 불러옴으로써 개발자는 앱의 초기 세팅을 작업해줄 수 있습니다. 또한 앱의 런 루프를 실행시켜 이러한 프로세스들을 처리합니다. 개발자가 제공할 것은 스토리보드 파일과 직접 작성한 코드뿐입니다. 



#### The Structure of an App

앱이 실행되기 시작하면 `UIApplicationMain` 함수는 몇몇 주요한 객체들을 설정하고 앱을 실행시키기 시작합니다. 앱의 심장 역할을 하는 객체는 `UIApplication` 객체로 이는 시스템과 앱의 다른 객체들간의 상호작용을 원할하게 하는 역할을 합니다. 다음의 그림은 대부분의 앱에서 찾아볼 수 있는 공통된 객체들간의 상호작용 흐름도입니다. 

<img src="https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Art/core_objects_2x.png">

먼저 알아야할 것은 위에서도 언급했듯이 iOS 앱은 MVC 구조를 사용한다는 것입니다. 이 구조는 앱의 데이터와 비즈니스 로직을 데이터를 시각화를 담당하는 부분(뷰)와 분리시켜 놓습니다. 이 구조는 화면의 크기가 다른 여러 디바이스에서 실행되야 하는 앱을 만드는데 중요한 역할을 담당합니다. 

다음은 위의 그림에 나와있는 객체들의 역할입니다. 

- [`UIApplication`](https://developer.apple.com/documentation/uikit/uiapplication) - `UIApplication` 객체는 event loop와 고수준의 앱 동작들을 관리합니다. 이 객체는 또한 주요 앱 전환과 일부 특별한 이벤트 (푸시 알림) 발생을 개발자가 정의한 delegate객체에 전달합니다. `UIApplication` 객체는 상속 없이 그대로 사용해야 합니다.

- App delegate 객체 - app delegate는 개발자가 작성한 코드 중 핵심입니다. 이 객체는 `UIApplication` 객체와 연결되어 앱의 초기화, 상태 전환 그리고 많은 고 수준의 앱의 이벤트들을 처리하는 일을 합니다. 또한 이 객체는 모든 앱에 존재하는 유일한 객체이기 때문에 앱의 초기 데이터 구조 설정을 하는데 자주 사용되곤 합니다.

- Documents and data model 객체 - Data model 객체들은 앱의 컨텐츠를 저장하고 있으며 해당 앱에만 한정되어 저장됩니다. 예를 들어 뱅킹 앱은 금융 거래가 포함된 데이터베이스를 저장하고 있을 수 있는 반면, 패인팅 앱은 이미지 객체나 이를 그리는 일련의 그리기 명령어들을 포함할 수 있습니다. ( 후자의 경우 이미지 객체 역시 이미지 데이터를 감싸고 있는 데이터 객체에 해당합니다.) 
  앱은 또한 Document 객체(`UIDocument`의 사용자 정의 서브 클래스)를 사용하여 일부 혹은 전체 데이터 객체들을 관리할 수 있습니다. Document 객체는 반드시 필요한 객체는 아니지만 단일 파일이나 패키지에 속하는 데이터를 그룹으로 묶어 관리하는데 편리한 기능을 제공합니다. Document에 대해 자세히 알아보시고 싶으시다면 [Document-Based App Programming Guide for iOS](https://developer.apple.com/library/archive/documentation/DataManagement/Conceptual/DocumentBasedAppPGiOS/Introduction/Introduction.html#//apple_ref/doc/uid/TP40011149) 문서를 참고하세요.

- View controller 객체 - 뷰 컨트롤러 객체는 화면 위 컨테츠의 시각화를 담당합니다. 뷰 컨트롤러는 단일 뷰와 해당 뷰의 자식 뷰들 집합을 관리합니다. 컨텐츠가 보여질 때 뷰 컨트롤러는 뷰들을 앱의 윈도우에 설치하여 이들이 화면상에서 보여질 수 있게 해줍니다. 
  `UIViewController` 클래스는 모든 뷰 컨트롤러 객체의 기본 클래스입니다. 이는 뷰들을 불러오고 보여주며 디바이스의 방향에 따라 회전시키는 기본적인 기능들을 제공하며 몇몇 시스템의 기본적인 행위들도 제공합니다. UIKit과 다른 프레임워크들은 이미지 피커, 탭 바 인터페이스, 네비게이션 인터페이스와 같은 기본 시스템 인터페이스를 구현하기 위해 뷰 컨트롤러 클래스들을 추가적으로 정의하여 제공하고 있습니다. 뷰 컨트롤러에 대한 보다 자세한 사용법은 [View Controller Programming Guide for iOS](https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/index.html#//apple_ref/doc/uid/TP40007457)를 참고하세요.

- [`UIWindow`](https://developer.apple.com/documentation/uikit/uiwindow) 객체 - `UIWindow` 객체는 화면 상의 하나 이상의 뷰들의 표시를 조정합니다. 대부분의 앱은 메인 화면에 컨텐츠들을 표시하는 오로지 하나의 윈도우만을 갖고 있습니다. 하지만 외부 디스플레이에서 컨텐츠를 표시한다면 추가적인 윈도우를 가질 수 있습니다.
  앱의 컨텐츠를 바꾸기 위해선 뷰 컨트롤러를 사용하여 해당 윈도우에 존재하는 뷰들을 변경해주어야 합니다. 절대 윈도우 그 자체를 변경해선 안됩니다. 뷰를 위치시키는 것 외에도 윈도우는 `UIApplication`과 상호작용하여 뷰와 뷰 컨트롤러에 이벤트를 전달하기도 합니다.

- View 객체, Control 객체 그리고 Layer 객체 - 뷰와 컨트롤들은 앱의 컨텐츠의 시각화를 담당합니다. 뷰는 하나의 객체로 지정된 사각형의 영역에 컨텐츠를 그리며 해당 영역에서 발생하는 이벤트를 처리해야 하는 역할을 갖고 있습니다. 컨트롤은 특별한 타입의 뷰로 버튼, 텍스트 필드, 토글 등을 구현하는 역할을 갖고 있습니다. 
  UIKit 프레임워크는 여러 다른 타입의 컨텐츠들을 표시하기 위한 기본적인 뷰들을 제공합니다. 직접적으로 `UIView` (혹은 `UIView`의 서브클래스)를 상속하여 자신만의 사용자 정의 뷰를 정의할 수 있습니다. 
  게다가 뷰와 컨트롤러를 통합하는 것 이외에도 앱은 Core Animation 레이어를 뷰와 컨트롤러의 계층에 통합시킬 수 있습니다. 레이어 객체는 시각적인 컨텐츠의 표시를 담당하는 데이터 객체입니다. 뷰는 레이어 객체를 집중적으로 사용하여 컨텐츠들을 화면에 보여주기 위해 렌더링 작업을 진행합니다. 개발자는 사용자 정의 레이어 객체를 추가하여 복잡한 애니메이션과 다른 타입의 정교한 시각적인 효과를 구현할 수 있습니다. 

앱이 관리하는 데이터와 이를 어떻게 사용자에게 보여주냐에 따라 하나의 iOS 앱을 다른 앱들과 구분할 수 있습니다. UIKit 객체와의 상호작용 그 자체만으로는 앱을 정의할 수 없지만 이러한 행위들의 도움으로 개발자가 앱을 재정의할 수 있습니다. 예를 들어 app delegate 메소드들은 앱의 상태가 전환될 때마다 이를 개발자에게 알려주고 이로인해 개발자가 작성한 코드들이 이에 적절히 대응할 수 있습니다.



#### The Main Run Loop

앱의 메인 런 루프(run loop)는 사용자와 관련된 모든 이벤트들을 처리합니다. `UIApplication` 객체는 런치 시점에 메인 런 루프를 실행시키고 이를 이벤트나 뷰에 기반한 인터페이스 갱신을 처리하는데 사용합니다. 이름에서 알 수 있듯이 메인 런 루프는 앱의 메인 스레드에서 동작합니다. 이러한 행위는 사용자의 이벤트가 도착한 순서대로 처리되는 것을 보장합니다. 

다음의 그림은 메인 런 루프의 구조를 나타내며 사용자의 이벤트를 앱이 어떻게 처리하는지를 보여줍니다. 사용자가 디바이스와 상호작용하게 되면 시스템은 이러한 상호작용과 연관된 이벤트를 생성하고 이를 UIKit에 의해 생성된 특수 포트를 통해 앱에 전달합니다. 이벤트들은 앱에 의해 내부적으로 큐에 쌓이게 되고 메인 런 루프에 의해 하나씩 처리됩니다. `UIApplication` 객체는 이러한 이벤트들을 받게 되는 가장 첫 번째 객체가 되고 이를 어떻게 처리해야할지 결정합니다. 터치 이벤트는 보통은 메인 윈도우가 처리하게 되고 메인 윈도우는 터치가 발생한 뷰로 해당 이벤트를 전달합니다. 다른 이벤트들은 앱의 다양한 객체들에 의해 약간은 다른 이벤트 전달 경로를 갖습니다. 

<img src="https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Art/event_draw_cycle_a_2x.png">

다양한 형태의 이벤트들이 iOS 앱에 전달됩니다. 다음의 리스트는 가장 흔한 이벤트들입니다. 이러한 타입의 이벤트들은 메인 런 루프를 통해 전달합니다. 하지만 몇몇은 그렇지 않습니다. 몇몇은 델리게이트 객체에 전달되거나 사용자 정의 블록에 전달될 수 있습니다. 

터치, 리모트 컨트롤, 모션, 가속도계 그리고 자이로스코프 이벤트들과 같이 이벤트들이 어떻게 처리되는지 궁금하시다면 [Event Handling Guide for iOS](https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/using_responders_and_the_responder_chain_to_handle_events) 문서를 참고하세요. 

| 이벤트 타입                            | 전달받는 곳             | 설명                                                         |
| -------------------------------------- | ----------------------- | ------------------------------------------------------------ |
| Touch                                  | 이벤트가 발생한 뷰 객체 | 뷰들은 **응답자(Responder)** 객체입니다. 뷰에 의해 처리되지 않은 이벤트들은 응답자 체인을 따라 처리될 때 까지 아래로 전달 됩니다. |
| Remote Control, Shake motion events    | First responder object  | 리모트 컨트롤 이벤트는 미디어 재생 장치를 제어하기 위함이고 헤드폰이나 다른 악세서리에 의해 생성됩니다. |
| Accelerometer, Magnetometer, Gyroscope | 사용자 지정 객체        | 가속도게, 자력계 그리고 자이로스코프 하드웨어와 관련된 이벤트들은 사용자가 지정한 객체로 전달됩니다. |
| Location                               | 사용자 지정 객체        | 개발자는 Core Location 프레임워크를 이용해 위치 이벤트를 받을 객체를 등록해야 합니다. [Location and Maps Programming Guide](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/LocationAwarenessPG/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009497)를 참고하세요. |
| Redraw                                 | 갱신되어야 하는 뷰      | 뷰 갱신 이벤트는 이벤트 객체를 포함하고 있진 않지만 뷰로 하여금 스스로 갱신을 하도록 합니다. iOS의 드로잉 구조는 [Drawing and Printing Guide for iOS](https://developer.apple.com/library/archive/documentation/2DDrawing/Conceptual/DrawingPrintingiOS/Introduction/Introduction.html#//apple_ref/doc/uid/TP40010156)를 참고하세요. |

터치 이벤트나 리모트 컨트롤 같은 몇몇의 이벤트들은 앱의 응답자 체인에 의해 처리됩니다. 응답자 객체들은 앱 내 어디든 존재할 수 있습니다. (`UIApplication` 객체, 뷰 객체 그리고 뷰 컨트롤러 객체들이 이러한 응답자 객체의 일부입니다.) 대부분의 이벤트들은 특정 응답자 객체를 기본 응답자 객체로 타겟팅하고 있지만 다른 응답자 객체로 이벤트를 넘길 수 있습니다. (응답자 체인에 의해) 예를 들어, 이벤트가 발생한 뷰에서 이벤트를 처리하지 않고 싶으면 해당 뷰의 부모 뷰나 자신이 속한 뷰 컨트롤러에 이벤트를 넘겨 처리하도록 할 수 있습니다. 

컨트롤(버튼과 같은)에서 발생한 이벤트는 여러 다른 유형의 뷰에서 발생한 터치 이벤트와는 다르게 처리 됩니다. 컨트롤과 성호작용할 수 있는 방법은 한정되어 있고 이렇게 한정된 상호작용은 액션 메시지로 포장되어 적절한 타겟 객체로 전달됩니다. 이러한 Target-Action 패턴은 컨트롤을 사용하여 보다 쉽게 앱 내의 사용자 정의 코드를 실행시킬 수 있습니다. 



#### Execution States for Apps

어느 순간에도 앱은 다음 상태 목록 중 하나의 상태를 갖습니다. 시스템은 시스템의 전반에 걸쳐 일어나는 특정 순간에 대한 대응으로 앱의 상태를 변경합니다. 예를 들어 사용자가 홈 버튼을 누르거나, 전화가 들어오거나 혹은 다른 기타 방해가 생겨났을 때 그에 대한 대응으로 앱은 상태를 변경합니다. 

- **Not running** - 앱이 아직 실행되지 않았거나 혹은 실행되고 있었지만 시스템에 의해 종료된 상태

- **Inactive** - 앱이 foreground에서 실행되고 있지만 현재 어느 이벤트도 받고 있지 않는 상태 (다른 코드를 실행하고 있을 수도 있다.) 앱은 대개 현재 상태에서 다른 상태로 전환 될 때 잠깐 동안만 이 상태(Inactive)에 머뭅니다. 

- **Active** - 앱이 foreground에서 실행되고 있고 이벤트를 전달 받고 있는 상태. foreground에서 실행되고 있는 앱이 갖는 상태입니다. 

- **Background** - 앱이 background에 있으면서 코드를 실행시키고 있는 상태. 대부분의 앱이 Suspended 상태로 들어가기 전 이 상태(background)로 먼저 진입합니다. 하지만 추가 실행 시간을 요구하는 앱은 일정 시간 동안 이 상태를 유지할 수 있습니다. 추가적으로 background 상태로 직접 실행되는 앱은 Inactive 상태 대신 바로 background 상태로 진입합니다. background 상태에서 코드를 실행시키는 방법에 대해 궁금하시다면 [Background Execution](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/BackgroundExecution/BackgroundExecution.html#//apple_ref/doc/uid/TP40007072-CH4-SW1) 문서를 참고하세요. 

- **Suspended** - 앱이 background 상태에 있지만 코드를 실행시키고 있지 않은 상태. 시스템은 자동으로 앱을 이 상태로 전환시키고 이러한 행위를 하기 전에 앱에 어떠한 것도 알리지 않습니다. Suspended 상태에서 앱은 메모리에 머물고 있긴 하지만 아무 코드도 실행하지 않습니다. 

다음의 그림은 상태 변화 흐름도입니다. 

<img src="https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Art/high_level_flow_2x.png">

대부분의 상태 전환은 해당 변환과 대응되는 app delegate 객체의 메소드 호출을 동반합니다. 이러한 메소드들에 각각의 상태 전환에 따라 대응할 수 있는 코드를 작성해줄 수 있습니다. 

- `application(_:willFinishLaunchingWithOptions:)` - 실행 시점에 실행 될 코드를 작성해 줄 수 있는 메소드입니다. 

- `application(_:didFinishLaunchingWithOptions:)` - 앱이 사용자에게 보여지기 전 최종 초기화 작업을 해줄 수 있는 메소드입니다. 

- `applicationDidBecomeActive(_:)` - foreground로 진입할 것이라고 앱에게 알려줍니다. 사용자에게 보여지기 전 마지막 작업을 이곳에서 작성해줄 수 있습니다. `application(_:didFinishLaunchingWithOptions:)`는 앱 시작 후 오로지 한 번만 호출되지만 이 메소드는 경우에 따라 여러 번 호출 될 수 있습니다.

- `applicationWillResignActive(_:)` - 앱이 foreground 상태로 부터 상태가 변환 될 것이라고 알립니다. 이 메소드를 사용하여 앱을 정지 상태로 만들 수 있습니다. 

- `applicationDidEnterBackground(_:) ` - 앱이 background 상태로 전환되었다는 것을 알리고 언제든 suspended 상태로 전환될 수 있습니다. 

- `applicationWillEnterForeground(_:)` - 앱이 background 벗어났고 forground 상태로 전환되고 있음을 알립니다. 하지만 아직 active한 상태는 아닙니다. 

- `applicationWillTerminate(_:)` - 앱이 종료될 것을 알립니다. 앱이 suspended 상태라면 이 메소드는 호출되지 않습니다. 



#### App Termination

앱은 언제든지 종료될 준비가 되어 있어야 하며 사용자의 정보를 저장하거나 다른 중요한 작업을 하기 위해 기다려서는 안됩니다. 시스템이 앱을 종료시키는 것은 앱의 생명 주기에 있어서 평범한 부분입니다. 시스템은 종종 앱을 종료시켜 메모리를 회수하고 다른 앱들이 실행될 공간을 제공합니다. 하지만 오작동하거나 이벤트에 대해 적절하게 응답하지 않는다면 역시 앱을 종료시킬 수 있습니다.

suspended 상태의 앱은 종료될 때 시스템이 프로세스를 죽이고 메모리를 회수하는 행위에 대한 어떤 알림도 받을 수 없습니다. 앱이 현재 background 상태에서 동작하고 있고 suspended 상태가 아니라면 시스템은 종료 전에 app delegate의  `applicationWillTerminate(_:)`를 호출하여 앱에 이 사실을 알립니다. 앱이 재시작할때는 이 메소드를 호출하지 않습니다.

추가적으로 시스템에 의한 종료가 아닌 사용자에 의한 종료는 시스템이 suspended 상태의 앱을 종료하는 프로세스와 동일합니다. 어떠한 알림도 앱에 전달되지 않습니다.



#### Threads and Concurrency

시스템은 앱의 메인 스레드를 생성하고 개발자는 다른 작업을 하기 위한 필요에 의해 스레드를 추가적으로 생성할 수 있습니다. iOS 앱에서는 이렇게 직접적으로 스레드를 생성하는 것 보단 Grand Central Dispatch(GCD), Operation 객체 그리고 비동기적 프로그래밍 인터페이스와 같은 기술을 사용하는 것이 선호됩니다. GCD와 같은 기술은 개발자가 수행 할 작업과 수행 할 작업의 순서를 정의할 수 있지만 가능한 CPU에 해당 작업을 가장 효과적으로 실행시키는 것은 시스템이 결정합니다. 시스템이 스레드를 관리하게 함으로써 개발자가 반드시 작성해야하는 코드는 단순해지고 코드의 정확성을 보장하기 쉬워지며 전반적인 성능이 향상됩니다. 

스레드와 동시성을 생각할 때 다음의 사항들을 고려하세요.

- 뷰, Core Animation 그리고 많은 UIKit 클래스가 포함된 작업은 반드시 앱의 메인 스레드에서 동작해야 합니다. 이 규칙에는 몇 가지 예외가 있습니다. 예를 들어 이미지 기반 조작은 백그라운드 스레드에서 일어날 수 있지만 확실하지 않다면 메인 스레드에서 발생해야 한다 가정해야 합니다. 

- 작업 시간이 긴 작업은 백 그라운드 스레드에서 동작해야 합니다. 네트워크 통신, 파일 접근 혹은 거대한 양의 데잍 처리 작업 모두 GCD나 Operation 객체를 사용하여 비동기적으로 처리되어야 합니다. 

- 앱 실행 시 가능한한 메인 스레드에서 작업을 수행해선 안됩니다. 앱 실행 시 앱은 가능한 빠르게 UI를 설정하는데 집중해야 하는데 UI를 설정하는데 필요한 작업들은 메인 스레드에서 수행되어야 하기 때문입니다. 그렇기 때문에 다른 작업들은 비동기적으로 수행되어야 하고 결과가 준비되는 즉시 이를 사용자에게 알려야 합니다.


GCD와 Operation 객체를 사용하여 작업을 처리하는 방법은 [Concurrency Programming Guide](https://developer.apple.com/library/archive/documentation/General/Conceptual/ConcurrencyProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008091) 문서를 참고하세요.

