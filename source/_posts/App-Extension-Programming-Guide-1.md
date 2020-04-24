---
title: App Extension Programming Guide
subtitle: App Extension Essentials
date: 2018-10-03 10:59:07
tags: ["iOS", "App Extension"]
category: ["iOS","Programming Guide"]
---

> 투데이 익스텐션(위젯)을 개발하면서 필요했고 궁금했던 부분을 공부하고 이를 번역해보았습니다.

### App Extensions Increase Your Impact

앱 익스텐션을 사용하면 사용자 정의 기능과 컨텐츠를 앱 이외의 영역으로 확장하여 사용자가 다른 앱이나 시스템과 상호작용할 때 앱 익스텐션을 통해 당신의 앱의 기능을 사용할 수 있다. 

당신은 특수한 작업을   위해 앱 익스텐션을 만든다. 예를 들어 웹 브라우저로부터 소셜 서비스에 포스팅을 하기 위해 **공유(Share)** 익스텐션을 사용할 수 있다. 혹은 사용자에게 좋아하는 팀의 소식을 제공하기 위해 **투데이(Today** 위젯을 제공하여 알람 센터에서 현재 스포츠 경기의 점수를 보여줄 수 있다. 심지어 사용자 정의 키보드를 만들어 제공하여 iOS 시스템 키보드를 대체할 수도 있다. 



#### There Are Several Types of App Extensions

iOS와 macOS는 여러 유형의 앱 익스텐션을 정의하고 각각은 시스템 영역의 단일 범위 내에 묶여있다. (예: 공유, 알람 센터 그리고 iOS 키보드) 이러한 확장을 가능케 하는 시스템 영역을 **확장 지점(extension point)**이라 부른다. 각각의 확장 지점은 사용 정책을 정의하고 해당 영역에 대한 앱 익스텐션을 생성했을 때 사용할 수 있는 API를 제공한다. 당신은 제공하고 싶은 기능의 확장 지점을 선택하면 된다. 

다음 표는 iOS와 macOS의 확장 지점 목록과 각각의 확장 지점에 대해 앱 익스텐션에서 사용할 수 있는 작업의 예를 보여준다. 

| Extension Point                        | Typical app extension functionality                          |
| -------------------------------------- | ------------------------------------------------------------ |
| **Action** (iOS/macOS, UI/non-UI)      | 호스트 앱의 컨텐츠를 조정하거나 관찰한다.                    |
| **Audio Unit** (iOS/macOS, UI/non-UI)  | 오디오 스트림을 생성하여 호스트 앱에 전송하거나 호스트 앱으로부터 오는 오디오 스트림을 수정하며 이를 다시 호스트 앱에 전송한다. |
| **Broadcast UI** (iOS/tvOS)            |                                                              |
| **Broadcaset Upload** (iOS/tvOS)       |                                                              |
| **Call Directory** (iOS)               | 전화번호를 통해 발신자를 식별하거나 차단한다. 이에 대한 자세한 내용은 [CallKit Framework Reference](https://developer.apple.com/documentation/callkit)를 참고하라. |
| **Content Blocker** (iOS/macOS)        | 웹키트에 당신의 컨텐츠 차단 앱이 규정을 갱신했다고 알린다. (이 앱 익스텐션은 사용자 인터페이스를 제공하지 않는다.) |
| **Document Provider** (iOS, UI/non-UI) | 파일 저장소의 접근과 관리 권한을 제공한다.                   |
| **Finder Sync** (macOS)                | 파일 동기화 상태에 대한 정보를 파인더에서 직접적으로 보여준다. (이 앱 익스텐션은 사용자 인터페이스를 제공하지 않는다.) |
| **Game app** (watchOS)                 | [App Programming Guide for watchOS](https://developer.apple.com/library/archive/documentation/General/Conceptual/WatchKitProgrammingGuide/index.html#//apple_ref/doc/uid/TP40014969)에 설명되어 있는 대로 애플 워치 전용 게임 앱을 제공한다. (게임 앱 템플릿은 컨텐츠 전용으로 구성된 WatchKit 앱 템플릿 버전이다. ) |
| **iMessage** (iOS)                     | 메시지 앱과 상호작용한다. 이에 대한 자세한 내용은 [Messages](https://developer.apple.com/reference/messages) 문서를 참고하라 |
| **Intents** (iOS)                      | 앱과의 시리 통합 지원에 관련된 작업을 처리한다. 이에 대한 자세한 내용은 SiriKit Programming Guide 문서를 참고하라. |
| **Intents UI** (iOS)                   | 앱과의 통합 지원과 관련된 작업을 처리한 후 통시리 혹은 지도(Maps) 앱 인터페이스를 사용자 정의한다. 이에 대한 자세한 내용은 SiriKit Programming Guide 문서를 참고하라. |
| **Notification Content** (iOS)         |                                                              |
| **Notification Service** (iOS)         |                                                              |
| **Photo Editing** (iOS/macOS)          | 사진 앱 내의 사진이나 비디오를 편집한다.                     |
| **Share** (iOS/macOS)                  | 다른 앱에 웹사이트나 컨텐츠를 공유한다.                      |
| **Smart Card Token** (macOS)           |                                                              |
| **Spotlight Index** (iOS)              | 앱이 실행중이 아닐 때 앱의 컨텐츠의 인덱스를 지정한다. 이에 대한 자세한 내용은 [Index App Content](https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/AppContent.html#//apple_ref/doc/uid/TP40016308-CH7) 문서를 참고하라. |
| **Sticker Pack** (iOS)                 | 사용자가 메시지 앱에서 사용할 수 있는 스티커 묶음을 제공한다. 이에 대한 자세한 내용은 [Messages](https://developer.apple.com/reference/messages) 문서를 참고하라. |
| **Today** (iOS/macOS)                  | 알람 센터의 Today 뷰에서 빠른 갱신 혹은 빠른 작업 수행을 한다. (Today 익스텐션은 위젯이라 불린다.) |
| **TV Services** (tvOS)                 |                                                              |
| **VPN** (iOS/macOS)                    | Packet Tunnel Provider 혹은 App Proxy Provider 확정 지점을 사용하여 비즈니스 사용자 정의 원격 엑세스 VPN 서버용 클라이언트를 생성한다. Filter Control Provider과 Filter Data Provider 확장 지점을 사용하여 학교 환경과 같은 관리되는 디바이스를 위한 컨텐츠 필터링을 생성한다. |
| **WatchKit App** (watchOS)             | [App Programming Guide for watchOS](https://developer.apple.com/library/archive/documentation/General/Conceptual/WatchKitProgrammingGuide/index.html#//apple_ref/doc/uid/TP40014969)에 설명되어 있는대로 애플 워치를 위한 알람 UI 혹은 앱을 제공한다 |
| **Xcode Source Editor** (macOS)        |                                                              |

앱 익스텐션을 위한 특수 영역을 시스템이 정의하기 때문에 당신이 제공하고 싶은 기능에 제일 알맞는 영역을 선택하는 것은 매우 중요하다. 예를 들어 공유 기능을 제공하고 싶다면 Share 익스텐션 지점과 Share Extension Xcode template을 사용하라. 

> 당신이 생성하는 각각의 앱 익스텐션은 위의 표에 확장 지점에 정확히 하나씩 대응된다. 하나 이상의 확장 지점과 대응되는 제네릭 익스텐션은 만들 수 없다.



#### Xcode and the App Store Help You Create and Deliver App Extensions

앱에 따라 앱 익스텐션은 다르다. 비록 익스텐션을 전달하기 위해선 앱을 반드시 사용해야 하지만 각각의 익스텐션은 제공하는 앱과 별개의 바이너리 나누어져 있으며 앱과 독립적으로 실행된다. 

앱에 새로운 타겟을 생성하여 앱 익스텐션을 생성할 수 있다. 다른 타겟과 마찬가지로 확장된 타겟은 앱 프로젝트 내에서 제품을 빌드하기 위해 결합하는 설정과 파일을 지정한다. 단일 앱(*한 개 혹은 그 이상의 익스텐션을 포함하고 있는 앱을 컨테이닝 앱이라 부른다.*)에 대해 여러 개의 익스텐션을 추가할 수 있다. 

앱 익스테션 개발을 시작할 수 있는 가장 좋은 방법은 Xcode가 제공하는 각각의 확장 지점에 대한 템플릿을 사용하는 것이다. 각각의 템플릿은 확장 지점 별 구현 파일 및 설정이 포함되어 있으며 당신의 컨테이닝 앱 번들에 추가되는 별도의 바이너리 파일을 생성한다. 

앱 익스텐션을 배포하기 위해선 앱 스토어에 컨테이닝 앱을 제출하면 된다. 사용자가 컨테이닝 앱을 설치하면 포함되어 있는 익스텐션 역시 설치된다. 

앱 익스텐션을 설치한 후 사용자는 이를 활성화시키기 위해 반드시 행동을 취해야 한다. 대체로 사용자는 현재 작업 흐름 내에서 익스텐션을 활성화할 수 있다. 만일 당신의 익스텐션이 Today 위젯이라 예를 들면 사용자는 알람 센터의 Today 화면을 편집하여 이를 활성화시킬 수 있다. 다른 경우로는 사용자가 Settings(iOS의)나 System Preferences(macOS의)에서 그들이 설치한 익스텐션을 활성화하고 관리할 수 있다.



#### User Experience App Extensions in Different Contexts

비록 각각의 유형의 앱 익스텐션이 서로 다른 작업을 수행하지만 대부분의 익스텐션에 공통되는 몇몇의 사용자 인터페이스가 있다. 앱 익스텐션을 설계할 때 당신이 선택한 확장 지점이 의도하는 사용자 경험을 이해하는 것은 중요하다. 높은 수준에서 모든 익스텐션 기능에 대한 최상의 사용자 경험은 신속하고 간소화되어야 하며 단일 작업에 집중해야 한다. 

사용자는 당신의 앱 익스텐션을 열기 위해 시스템이 제공하는 사용자 인터페이스와 상호작용한다. 예를 들어 사용자는 Share 익스텐션에 접근하기 위해 시스템이 제공하는 앱 내의 Share 버튼을 사용하여 보이는 익스텐션 목록 중 하나를 선택한다.

비록 대부분의 앱 익스텐션이 최소한의 사용자 정의 UI 요소를 제공하지만 사용자는 당신의 익스텐션에 들어가기 전에는 이를 보지 않는다. 사용자가 익스텐션에 들어왔을 때 당신이 정의한 사용자 정의 UI는 사용자로 하여금 새로운 컨텍스트로 이동할 수 있도록 도와준다. 사용자는 현재 앱과 당신의 익스텐션을 구분할 수 있기 때문에 당신이 제공하는 고유한 기능을 활용할 수 있다. 사용자가 익스텐션을 별도의 개체로 인식한다는 것은 잘못 구현되거나 제대로 수행되지 않는 익스텐션을 알아차리고 이를 제거할 수 있음을 의미한다. 

부드럽게 당신의 앱 익스텐션으로 사용자를 전환시키기 위해서 일반적으로 당신은 확정 지점과 관련된 UI와 당신의 사용자 정의 UI의 균형을 원할 것이다. 예를 들어 위젯을 Today 화면에 속하는 것처럼 보이게 만드는 것은 좋은 아이디어다. Photo Editing 익스텐션의 UI를 iOS의 사진 앱과 조화롭게 만드는 것도 비슷하다.

> 당신의 앱 익스텐션이 화면에 보이는 사용자 정의 UI를 제공하지 않더라도 사용자는 이를 활성화시키기 위해 특정 액션을 취했기 때문에 여전히 당신의 익스텐션이 현재 앱과는 다르다는 것을 구분할 수 있다.

---

### Understand How an App Extension Works

앱 익스텐션은 앱이 아니다. 이는 특정 확장 지점에 정의 된 정책을 준수하는 명확하고 잘 정의된 작업을 구현한다. 

#### An App Extension's Life Cycle

앱 익스텐션은 앱이 아니기 때문에 이의 생명 주기와 환경은 다르다. 대부분의 경우에서 앱 익스텐션은 사용자가 앱의 UI를 선택하거나 액티비티 뷰 컨트롤러로부터 실행된다. 앱 익스텐션을 실행시키는데 사용되는 앱을 호스트 앱이라 부른다. 호스트 앱은 익스텐션에 제공되는 컨텍스트를 생성하고 익스텐션이 사용자 액션에 대한 응답으로 요청을 보낼 때 익스텐션의 생명 주기를 시작한다. 앱 익스텐션은 호스트 앱으로부터 전달받은 요청을 완료하자마자 종료된다.

사용자가 OS X 호스트 앱의 어떤 텍스트를 선택하고 공유 버튼을 활성화하고 공유 목록에서 앱 익스텐션을 선택하여 텍스트를 소셜 공유 웹사이트에 게시하는 것을 돕는 경우를 예를 들 수 있다. 호스트 앱은 선택한 텍스트가 포함된 요청을 익스텐션에 발행함으로써 사용자의 선택에 응답한다. 이러한 상황을 일반화한다면 다음 그림과 같다. 

<img src="https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/Art/app_extensions_lifecycle_2x.png">

위 그림의 2단계에서 호스트 앱의 요청 안에서 식별된 앱 익스텐션을 실행시키고 그들 사이의 통신 채널을 구성한다. 이 익스텐션은 호스트 앱의 컨텍스트 내에서 자신의 뷰를 보여주고 호스트 앱의 요청으로부터 받아온 아이템들을 사용하여 자신의 작업을 수행한다. (이 예에서는 익스텐션은 선택된 텍스트를 받는다.)

위 그림의 3단계에서 사용자는 앱 익스텐션 안에서 작업을 수행하거나 취소하고 익스텐션을 사라지게 한다. 이러한 행동에 대한 응답으로 익스텐션은 사용자의 작업을 즉시 수행하거나 혹은 필요하다면 작업을 수행하기 위해 백그라운드 프로세스를 실행시킴으로써 호스트 앱의 요청을 완료한다. 즉시 완료되거나 나중에 완료되거나 익스텐션의 작업이 완료되면 그 결과는 호스트 앱으로 반환된다. 

앱 익스텐션이 자신의 작업을 수행한 직후 (혹은 작업을 수행하기 위해 백그라운드 세션을 시작한 경우) 시스템은 위 그림 4단계에서 설명된 것과 같이 익스텐션을 종료한다.



#### How an App Extension Communicates

앱 익스텐션은 주로 자신의 앱과 통신하고 트랜잭션 처리와 관련하여 다음과 같이 작동한다. 호스트 앱으로부터의 요청과 익스텐션으로부터의 응답이 존재한다. 다음 그림은 호스트 앱에 의해 실행된 익스텐션과 컨테이닝 앱과의 관계를 간략하게 보여주고 있다. 

<img src="https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/Art/simple_communication_2x.png">

> 앱 익스텐션은 호스트 앱과 유일하게 직접적으로 통신한다. 

앱 익스텐션과 그의 컨테이닝 앱 사이에선 직접적인 통신은 없다. 심지어 일반적으로 컨테이닝 앱은 포함된 익스텐션이 실행 중일 때 실행되고 있지 않는다. 앱 익스텐션의 컨테이닝 앱과 호스트 앱은 전혀 통신하지 않는다. 

일반적인 요청/응답 트랜잭션에서 시스템은 호스트 앱을 대신하여 앱 익스텐션을 열고 호스트에 의해 제공된 데이터를 익스텐션 컨텍스트에 전달한다. 익스텐션은 사용자 인터페이스를 보여주며 몇몇 작업을 수행하고 익스텐션 의도에 적합하다면 데이터를 호스트 앱에 전달한다. 

위 그림의 점선은 앱 익스텐션과 그의 컨테이닝 앱 사이의 상호작용이 제한되어 있다는 것을 의미한다. Today 위젯은 `NSExtensionContext` 클래스의 `open(_:completionHandler:)` 메소드를 사용하여 시스템에 자신의 컨테이닝 앱을 열어달라고 요청할 수 있다. 

> 이를 위해선 Xcode 프로젝트에서 호스트 앱 타겟을 선택하고 `Info` 탭의 URL Types에서 `identifier`와 `URL Schemes`를 정의해주어야 한다. `open(_:completionHandler:)`에 넘겨주어야 하는 값은 `URL Schemes`인데 `"YOUR_URL_SCHEME://"`의 형태로 넘겨주면 된다. 

밑의 그림의 Read/Write 화살표에 표시된 것처럼 앱 익스텐션 및 컨테이닝 앱은 비공개로 정의된 공유 컨테이너의 공유 데이터에 접근할 수 있다. 익스텐션과 그의 호스트 앱 그리고 컨테이닝 앱 간의 통신에 대한 용어는 밑의 그림에 간단한 형태로 보여지고 있다. 

<img src="https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/Art/detailed_communication_2x.png"> 

> 시스템은 프로세스 간 통신을 사용하여 호스트 앱과 앱 익스텐션이 함께 작동하여 응집력 있는 사용자 경험을 제공할 수 있도록 한다. 코드에서 확장 지점과 시스템에서 제공하는 상위 수준의 API를 사용하기 때문에 기본 통신 메커니즘에 대해선 고려할 필요가 없다. 



#### Some APIs Are Unavailable to App Extensions

시스템에 집중된 역할을 하기 때문에 앱 익스텐션은 몇몇 활동에 대해 참여할 수 없다. 다음은 앱 익스텐션이 할 수 없는 작업들이다. 

- `UIApplication`의 `shared` 객체에 접근할 수 없기 때문에 이 객체의 어떠한 메소드도 사용할 수 없다. 
- `NS_EXTENSION_UNAVAILABLE` 메크로 혹은 이와 비슷한 사용할 수 없다는 것을 의미하는 메크로가 포함된 헤더 파일의 API 혹은 사용 불가능한 프레임워크의 API는 사용할 수 없다. 예를 들어 iOS 8.0에서 HealthKit 프레임워크와 EventKit 프레임워크는 앱 익스텐션에서 사용할 수 없는 프레임워크다. 
- iOS 앱의 카메라나 마이크에 접근할 수 없다. (다른 익스텐션과 다르게 iMessage 앱은 `Info.plist`의 키에 `NSCameraUsageDescription`과 `NSMicrophoneUsageDescription` 가 올바르게 구성되어 있다면 해당 자원들에 접근이 가능하다.)
- 오랜 시간이 소요되는 백그라운드 작업. 이 부분의 제한은 이 문서의 확정 지점에 설명된 것처럼 플랫폼에 따라 다르다. (앱 익스텐션은 `URLSession` 객체를 사용하여 다운로드와 업로드를 시작할 수 있고 해당 작업의 결과를 컨테이닝 앱에 전달할 수 있다.)
- AirDrop을 통한 데이터 수신. 앱 익스텐션은 `UIActivityViewController` 클래스를 사용하여 앱에서와 같은 방식으로 AirDrop을 사용하여 데이터를 전송할 수 있다. 

---

### Creating an App Extension

앱 익스텐션을 개발할 준비가 되었다면 먼저 사용자 작업을 원활하게 지원할 확장 지점을 선택해야 한다. 그에 해당하는 Xcode 앱 익스텐션 템플릿을 사용하고 사용자 인터페이스(UI)와 사용자 정의 코드를 기본 파일에 추가한다. 앱 익스텐션의 검수와 최적화가 끝난 후 컨테이닝 앱 안에서 배포될 준비가 된 것이다. 



#### Begin Development By Choosing the Right Extension Point 

각 확장 지점은 미리 잘 정의된 사용자 시나리오를 따르고 있기 때문에 당신이 해야 할 첫 번째 작업은 당신이 제공하고자 하는 기능을 전달할 적당한 유형의 확장 지점을 고르는 것이다. 이 선택은 사용자가 사용 가능한 API를 결정하고 경우에 따라 API가 작동하는 방식을 결정하기 때문에 중요한 선택이 된다. 

iOS와 OS X에서 지원되는 확장 지점은 `Info.plist`의 확장 지점 식별자 키를 따르는데 이는 `NSExtensionPointIdentifier` 섹션에 설명되어 있다.

앱 익스텐션에 맞는 확장 지점을 선택한 후 컨테이닝 앱에 새로운 타겟을 추가한다. 가장 쉽게 추가하는 방법은 Xcode가 제공하는 해당 확장 지점을 위해 미리 구성된 템플릿을 사용하는 것이다. 

Xcode 앱 프로젝트에 새로운 타겟을 추가하기 위해선 `File > New > Target`을 통해서 파일을 선택해야 한다. 왼편의 사이드바에서 iOS 혹은 OS X의 `Application Extension`을 선택한다. 그리고 오른 편에서 원하는 템플릿을 선택하면 된다. 다음 그림은 iOS에서 사용하는 앱 익스텐션을 선택한 것을 보여준다. 

<img src="https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/Art/add_new_target_2x.png">

템플릿을 고르고 프로젝트에 성공적으로 타겟을 추가한 후 사용자 정의 코드를 추가하기 전 빌드하고 실행할 수 있어야 한다. 빌드를 하면 `.appex` 확장자를 갖는 번들을 확인할 수 있다. 

> 앱 익스텐션 타겟은 아키텍쳐 빌드 세팅에서 arm64 (iOS) 혹은 x86_64 아키텍쳐 (OS X)를 포함해야 한다. 그렇지 않으면 앱 스토어에 리젝 될 것이다. 새로운 앱 익스텐션을 추가할 때 Xcode는 `Standard architectures`와 함께 적절한 64 비트 아키텍쳐를 포함시킬 것이다. 
>
> 만일 컨테이닝 앱이 임베디드 프레임워크와 연결되어 있다면 앱은 반드시 64 비트 아키텍쳐를 포함해야 하며 그렇지 않으면 앱 스토어에 리젝 될 것이다. 
>
> 64 비트 개발에 대한 자세한 정보는 플랫폼에 따라 [64-Bit Transition Guide for Cocoa Touch](https://developer.apple.com/library/archive/documentation/General/Conceptual/CocoaTouch64BitGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40013501) 혹은 [64-Bit Transition Guide for Cocoa](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Cocoa64BitGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40004247) 문서를 참고하라. 

대부분의 경우 System Preferences 혹은 Settings에서 활성화함으로써 기본 앱 익스텐션을 테스트할 수 있고 다른 앱을 통해 이에 접근할 수 있다. 예를 들어 OS X의 Share 익스텐션을 사파리의 웹 페이지에서 이를 열수 있고 툴바의 공유 버튼을 클릭하고 메뉴에서 당신의 익스텐션을 선택할 수 있다.



#### Examine the Default App Extension Template

각각의 앱 익스텐션 템플릿은 `Info.plist` 파일, 뷰 컨트롤러 클래스 그리고 기본 사용자 인터페이스를 기본으로 포함하고 있다. 기본 뷰 컨트롤러 클래스(혹은 주요 클래스)는 당신이 구현해야 할 메소드들의 일부분을 포함할 수 있다. 

앱 익스텐션 타겟의 `Info.plist` 파일은 확장 지점과 당신의 익스텐션에 대한 조금 더 상세한 부분을 식별한다. 최소한 이 파일은 `NSExtension` 키와 확장 지점을 명시하는 딕셔너리의 키와 값을 포함해야 한다. 예를 들어 `NSExtensionPointIdentifier` 키가 요구하는 값은 `com.apple.widget-extension`와 같이 확장 지점 DNS 명의 역순이다. 다음은 당신의 익스텐션 `NSExtension` 딕셔너리에서 확인할 수 있는 키와 값 들이다. 

- `NSExtensionAttributes`
  - 확장 지점의 특성을 지칭하는 딕셔너리로 Photo Editing 익스텐션의  `PHSupportedMediaTypes`가 있을 수 있다. 
- `NSExtensionPricipalClass`
  - `SharingViewController`와 같이 템플릿에 의해 생성되는 주요 뷰 컨트롤러의 이름이다. 호스트 앱이 익스텐션을 깨울 때 확장 지점은 이 클래스를 초기화해야 한다..
- `NSExtensionMainStoryboard` (오로지 iOS 익스텐션만)
  - 익스텐션의 기본 스토리보드 파일로 주로 `MainInterface`의 이름을 갖는다.

프로퍼티 리스트 설정에 이어 템플릿은 기본적으로 몇몇 기능(Capabilites)들을 가질 수 있다. 각 확장 지점은 확장 지점이 지원하는 작업 유형에 따라 기능들을 알맞게 정의할 수 있다. 예를 들어 iOS의 Document Provider 익스텐션은 `com.apple.security.application-groups` 권한을 가질 수 있다. 

OS X 앱 익스텐션의 모든 템플릿은 앱 샌드박스를 포함하며 `com.apple.security.files.user-selected.read-only` 권한을 기본적으로 갖는다. 당신은 필요에 따라 네트워크 사용이나 사용자의 사진첩 혹은 연락처 정보에 접근과 같은 추가적인 기능들을 추가해줄 수 있다. 

> 일반적으로 컨테이닝 앱에 개인 정보에 대한 접근을 허용하면 컨테이닝 앱의 모든 익스텐션들도 접근 권한을 갖는다. 



#### Respond the Host App's Request

[Understand How an App Extension Works](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/ExtensionOverview.html#//apple_ref/doc/uid/TP40014214-CH2-SW2)에서 배웠듯이 앱 익스텐션은 호스트 앱 내에서 사용자가 선택하고 호스트 앱이 요청을 발행했을 때 열린다. 상위 수준에서 익스텐션은 요청을 받고 사용자의 작업을 도우며 사용자의 행위에 따라 요청을 완료하거나 취소한다. 예를 들어 Share 익스텐션은 호스트 앱으로부터 요청을 받고 응답으로 자신의 뷰를 띄운다. 사용자가 그 뷰의 컨텐츠를 정리한 후 컨텐츠의 포스팅이나 취소를 선택할 수 있으며 익스텐션은 요청에 따라 이를 완료하거나 취소할 수 있다.

호스트 앱이 앱 익스텐션에 요청을 보낼 때 이는 익스텐션의 컨텍스트를 정의한다. 많은 익스텐션에서 가장 중요한 컨텍스트는 사용자가 익스텐션에 머물 때 그들이 다루고 싶어 하는 아이템들을 구성하는 것이다. 예를 들어 OS X의 Share 익스텐션의 컨텍스트는 사용자가 포스팅하고자 하는 선택된 텍스트를 포함할 것이다. 

호스트 앱이 요청을 발행하자마자 (일반적으로 [`beginRequest(with:)`](https://developer.apple.com/documentation/foundation/nsextensionrequesthandling/1413395-beginrequestwithextensioncontext) 메소드를 호출) 당신의 앱 익스텐션은 컨텍스트에 접근하기 위해 주요 뷰 컨트롤러에서 [`extensionContext`](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621411-extensioncontext) 프로퍼티를 사용할 수 있다. 자식 뷰 컨트롤러들 역시 체이닝을 통해 이에 접근할 수 있다.

다음으로 앱 익스텐션은 [`NSExtensionContext`](https://developer.apple.com/documentation/foundation/nsextensioncontext) 클래스를 컨텍스트를 조사하고 그 안의 아이템들에 접근하는데 사용할 수 있다. 종종 이는 뷰 컨트롤러의 `loadView` 메소드 안에서 컨텍스트와 아이템들을 얻기 위해 사용되며 이를 통해 정보를 뷰에 보여줄 수 있다. 익스텐션 컨텍스트를 얻기 위해선 다음과 같이 코드를 작성하면 된다. 

```swift
let myExtensionContext = self.extensionContext
```

컨텍스트 객체의 [`inputItems`](https://developer.apple.com/documentation/foundation/nsextensioncontext/1414827-inputitems) 프로퍼티는 앱 익스텐션에서 사용되는 아이템들을 포함하고 있다. 이 프로퍼티는 [`NSExtensionItem`](https://developer.apple.com/documentation/foundation/nsextensionitem) 객체들의 배열로 각각은 익스텐션이 사용할 수 있는 아이템들을 갖고 있다. 컨텍스트 객체에서 이 아이템들에 접근하려면 다음과 같이 코드를 작성하면 된다.

```swift
let inputItems = myExtensionContext.inputItems
```

각각의 `NSExtensionItem` 객체에는 제목, 컨텐츠 텍스트, 첨부 파일 그리고 사용자 정보와 같은 항목의 측면을 설명하는 여러 속성이 들어 있다. 

[`attachments`](https://developer.apple.com/documentation/foundation/nsextensionitem/1416690-attachments) 프로퍼티는 아이템과 관련된 미디어 데이터의 배열을 포함한다. 예를 들어 공유 요청과 관련된 아이템의 `attachments` 프로퍼티는 사용자가 공유하고자 하는 웹페이지 포함할 수 있다. 

사용자가 이러한 인풋 아이템을 다룬 후 일반적으로 앱 익스텐션은 사용자에게 작업의 완료와 취소의 선택지를 제공한다. 사용자의 선택에 따라 선택적으로 `NSExtensionItem` 객체를 호스트 앱에 반환하는[`completeRequest(returningItems:completionHandler:)`](https://developer.apple.com/documentation/foundation/nsextensioncontext/1411301-completerequest) 메소드나 에러 코드를 반환하는 [`cancelRequest(withError:)`](https://developer.apple.com/documentation/foundation/nsextensioncontext/1412773-cancelrequestwitherror) 메소드를 호출한다. 

> 만일 당신의 앱 익스텐션이 [`completeRequest(returningItems:completionHandler:)`](https://developer.apple.com/documentation/foundation/nsextensioncontext/1411301-completerequest) 를 호출한다면 시스템이 요청할 경우 앱 익스텐션을 일시 중지할 수 있도록 최소한의 코드 블록을 제공해야 한다. 이에 대한 자세한 내용은 [NSExtensionContext Class Reference](https://developer.apple.com/documentation/foundation/nsextensioncontext)를 참고하라.

iOS에선 웹 사이트에 컨텐츠를 업로드하는 것과 같은 약간의 시간이 더 필요한 작업들의 수행이 필요한 경우가 있다. 이러한 경우 `URLSession` 클래스를 사용하여 백그라운드에서 백그라운에드에서 전송을 시작해줄 수 있다. 백그라운드 전송은 별개의 프로세스를 사용하기 때문에 앱 익스텐션이 호스트 앱의 요청을 수행한 후 종료되어도 전송은 낮은 우선순위로 지속될 수 있다. 앱 익스텐션에서 `URLSession`을 사용하는 법에 대한 자세한 내용은 [Performing Uploads and Downloads](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/ExtensionScenarios.html#//apple_ref/doc/uid/TP40014214-CH21-SW2) 문서를 참고하라.

> 비록 업로드 혹은 다운로드 작업을 위한 백그라운드 작업은 가능해도 VoIP를 지원하거나 백그라운드 오디오와 같은 백그라운드 작업은 익스텐션에선 불가능하다. 
>
> 만일 앱 익스텐션의 `Info.plist` 파일에 `UIBackgroundModes` 키값을 포함시킨다면 익스텐션은 앱 스토어에 의해 리젝될 것이다. (이 키에 대한 자세한 내용은 [UIBackgroundModes](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/iPhoneOSKeys.html#//apple_ref/doc/uid/TP40009252-SW22) 문서를 참고하라.)



#### Optimize Efficiency and Performance

앱 익스텐션은 사용자가 느끼기에 빠르고 가벼워야 한다. 앱 익스텐션이 1초 미만을 목표로 신속하게 실행되도록 설계하라. 너무 느리게 시작하는 앱 익스텐션은 시스템에 의해 종료될 것이다. 

앱 익스텐션이 실행의 메모리 한계치는 포그라운드 앱의 메모리 한계치보다 낮다. 두 플랫폼 모두 사용자가 호스트 앱에 주 목적을 두고 있기 때문에 시스템은 쉽게 익스텐션을 종료시킬 수 있다. 몇몇의 익스텐션은 다른 것들보다 낮은 메모리 한계치를 갖고 있다. 예를 들어 위젯은 특히 효율적이어야 하는데 사용자가 동시에 여러 위젯을 주로 사용하기 때문이다. 

당신의 앱 익스텐션은 메인 런 루프에 권한이 없으므로 메인 런 루프에서 올바른 동작을 위해 설정된 규칙을 따라야 한다. 예를 들어 만약 익스텐션이 메인 런 루프를 차단하면 다른 앱 혹은 익스텐션에 좋지 않은 사용자 경험이 생기게 된다. 

GPU는 시스템 내에서 공유 자원임을 명심하자. 앱 익스텐션은 공유 자원에 대한 상위 수준의 우선순위를 갖지 않는다. 예를 들어 위젯이 그래픽을 주로 사용하는 게임을 실행시킨다면 이는 좋지 않은 사용자 경험을 제공하게 될 것이다. 시스템은 메모리에 대한 압박으로 이러한 익스텐션을 강제로 종료하곤 한다. 시스템 자원을 많이 사용하는 기능은 앱에 적합하지만 앱 익스텐션에는 적합하지 않다. 



#### Design a Streamlined UI

대부분의 확장 지점은 실행되었을 때 사용자가 볼 수 있는 최소한의 사용자 정의 UI를 필요로 한다. 익스텐션의 UI는 간단해야 하며 단일 작업의 수행을 목적으로 설계되어야 한다. 성능과 사용자 경험을 향상시키기 위해선 익스텐션의 메인 테스크가 지원할 수 없는 불필요한 UI는 삼가해야 한다. 

대부분의 Xcode 앱 익스텐션 템플릿은 개발자가 시작할 수 있는 플레이스 홀더 UI를 제공한다. 

사용자는 아이콘과 이름으로 당신의 앱 익스텐션을 구분한다. 익스텐션의 아이콘은 반드시 컨테이닝 앱 아이콘과 같아야 한다. 컨테이닝 앱 아이콘을 사용하는 것은 사용자가 앱이 제공하는 익스텐션에 대한 존재를 인지할 수 있도록 도와준다. 

iOS Share 익스텐션은 자동으로 컨테이닝 앱 아이콘을 가져오며 Share 익스텐션 타겟을 위한 별개의 아이콘을 제공한다면 Xcode는 이를 무시할 것이다. 다른 모든 유형의 앱 익스텐션에서도 반드시 컨테이닝 앱과 일치하는 아이콘을 제공해야 한다. 

앱 익스텐션에 아이콘을 추가하는 방법은 Creating an Asset Catalog와 Adding an App Icon Set or Launch Image Set 문서를 참고하라. iOS 앱 익스텐션 아이콘의 요구 사항에 대한 문서는 iOS 휴먼 인터페이스 가이드라인의 App Extensions 챕터를 참고하라.

앱 익스텐션의 이름은 익스텐션 타겟의 `CFBundleDisplayName` 값에 의해 정해지고 이는 `Info.plist`에서 수정할 수 있다. 이 키에 대한 값을 지정하지 않는다면 `CFBundleName` 키의 값에 해당하는 컨테이닝 앱의 이름을 사용할 것이다. 

현지화된 앱 익스텐션을 사용한다면 익스텐션 이름 역시 현지화된 이름을 사용해야 한다.

또한 몇몇의 앱 익스텐션은 짧은 설명을 필요로 한다. 예를 들어 OS X 위젯은 사용자가 투데이 뷰에서 원하는 위젯을 선택하는 것을 도울 수 있는 설명을 보여준다. 이 텍스트를 제공하기 위해선 `InfoPlist.strings` 파일의 `widget.description` 키의 값을 수정해야 한다. 



#### Debug, Profile and Test Your App Extension

> 컨테이닝 앱과 이에 포함된 앱 익스텐션들은 서명되어 있어야 한다.
>
> Xcode 프로젝트 내의 모든 타겟들은 같은 방식으로 서명되어야 한다. 예를 들어 테스팅 하는 동안 애드 혹 서명을 하거나 개발자 증명서를 사용해야 한다. 하지만 반드시 프로젝트 내 모든 타겟들이 동일한 방법으로 서명되어야 한다. 배포하기 위해선 모든 타겟에 배포 증명서를 사용해야 한다. 

Xcode를 사용하여 앱 익스텐션을 디버그 하는 것은 Xcode에서 다른 것들을 디버그 하는 것과 매우 비슷한 과정을 갖지만 한 가지 중요한 차이점은 익스텐션의 Run 단계 스키마에선 호스트 앱을 실행 파일로 지정해야 한다는 것이다. 지정된 호스트의 UI를 통해 익스텐션에 접근하면 Xcode 디버거는 익스텐션과 연결될 것이다. 

Xcode 앱 익스텐션 템플릿의 스키마에선 실행 파일에 대해 Ask On Launch 옵션을 사용한다. 이 옵션을 사용하면 프로젝트를 빌드하고 실행할 때마다 호스트 앱을 선택하라는 메시지가 나타난다. 매번 특정 호스트를 지정하려면 스키마 편집기를 열고 Run 단계의 앱 익스텐션 스키마를 위해 Info 탭을 사용하면 된다. 

Xcode 디버거와 앱 익스텐션을 연결하기 위해선

1. 앱 익스텐션 스키마를 활성화려면 `Product > Scheme > MyExtensionName`을 선택하거나 Xcode 툴바에서 스키마 팝업 메뉴를 선탠하고 `MyExtensionName`을 클릭한다.
2. 빌드와 실행 버튼을 눌러 Xcode에게 지정된 호스트 앱을 실행하라고 알린다. 디버그 탐색기는 앱 익스텐션을 호출할 때까지 기다린다.
3. 호스트 앱의 UI에서 앱 익스텐션을 깨운다. Xcode 디버거는 익스텐션 프로세스에 연결되고 브레이크 포인트를 설정하며 익스텐션이 실행되도록 한다. 이 시점에서 당신은 다른 디버깅 프로세스와 동일한 기능들을 갖는다. 

> Xcode 디버그 콘솔의 로그에서 익스텐션의 바이너리는 `CFBundleDisplayName` 속성이 아닌 `CFBundleIdentifier` 속성 값과 연관될 수 있다. 

앱 익스텐션은 반응에 신속해야 하며 효율적이어야 하기 때문에 익스텐션이 실행되는 동안 디버그 탐색기의 디버그 수치를 확인하는 것은 좋은 생각이다. 디버그 수치는 앱 익스텐션이 실행 중일 때의 CPU, 메모리 그리고 다른 시스템 자원들의 사용량을 보여준다. 비정상적인 CPU 사용량과 같은 성능의 문제에 대한 단서를 발견하면 Instruments를 사용하여 익스텐션의 상태와 개선할 부분을 식별할 수 있다. 디버깅 세션 안에 있는 동안엔 어느 디버그 수치 보고서(*디버그 수치 보고서를 보기 위해선 디버그 영역 안의 수치 부분을 클릭하라.*)에서도 Instruments의 Profile을 클릭하여 Instruments를 열 수 있다. 이에 대한 자세한 정보는 Instruments를 어떻게 사용하는지가 나와있는 [Instruments User Guide](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/InstrumentsUserGuide/index.html#//apple_ref/doc/uid/TP40004652) 문서를 참고하라. 

> `Product > Profile`을 선택하면 Instruments에서 앱 익스텐션이 직접 빌드 되어 실행된다. Instruments는 스키마의 Profile 섹션 내의 실행 파일을 익스텐션의 호스트로 하기 위해 사용한다. 

Xcode 테스팅 프레임워크(XCTest API)를 사용하여 앱 익스텐션을 테스트하려면 컨테이닝 앱을 호스트 환경으로 사용하여 익스텐션 코드를 사용하는 테스트를 작성하라. 



#### Distributing the Containing App

앱 익스텐션이 컨테이닝 앱 내에 있지 않다면 앱 스토어에 제출할 수 없고 한 앱에서 다른 앱으로 익스텐션을 이전할 수 없다. 

iOS 앱 익스텐션을 제출하기 위해선 컨테이닝 앱을 앱 스토어에 제출해야 한다. 

OS X 앱 익스텐션을 제출하기 위해선 컨테이닝 앱을 제출하는 것을 추천하지만 필수는 아니다.

> Mac 앱 스토어 외부에 OS X 앱 익스텐션을 배포하면 게이트키퍼는 사용자가 컨테이닝 앱을 열고 승인하기 전까지 앱이 실행되지 않도록 한다. 또한 개발자 ID 이외의 인증서로 코드를 작성하는 경우 사용자는 익스텐션을 사용을 활성화할 수 있는 컨테이닝 앱을 열기 위해 게이트 키퍼를 명시적으로 덮어써야 한다. 

앱 리뷰에 통과하기 위해선 단순히 익스텐션을 포함하는 것이 아닌 기능을 갖고 있어야 한다. 