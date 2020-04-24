---
title: App Programming Guide for iOS (3)
subtitle: Background Execution
date: 2018-08-23 18:44:47
tags: ["iOS", "Swift"]
category: ["Programming Guide"]
---

사용자가 앱을 사용하고 있지 않는다면 시스템은 앱을 background로 옮깁니다. 많은 앱들에게 background 상태는 앱이 중지되기 전에 잠시 머무는 상태에 불과합니다. 앱을 중지시키는 행위는 베터리의 효율을 높이며 시스템으로 하여금을 중지된 앱의 자원을 회수하여 foreground에 있는 앱에 할당할 수 있게끔 합니다. 

대부분의 앱이 쉽게 중지 상태로 들어가지만 또한 역시 background에서도 앱이 실행될 수 있는 합당한 사유가 존재한다면 앱은 background 상태에서 중지 상태로 들어가지 않고 지속해서 작업을 수행할 수 있습니다. 등산 앱 같은 경우 사용자의 위치를 추적하여 동선을 앱에서 보여줄 필요가 있을 것입니다. 오디오 앱은 잠금에서도 미디어를 재생시켜야 할 필요가 있을 수 있으며 다른 앱들은 background에서 앱에서 사용할 컨텐츠를 다운받는 행위를 필요로 할 수 있습니다. 이렇게 background에서 컨텐츠를 다운로드 받는 행위를 통해 컨텐츠를 사용자에게 보여주는 딜레이를 최소화할 수 있습니다. 위와 같이 앱이 background에서도 작업을 수행할 필요가 있다면 iOS가 시스템 자원이나 디바이스의 베터리를 낭비하지 않으면서 이를 효율적으로 수행할 수 있도록 도와줄 것입니다. iOS가 제공하는 기술은 세 가지 카테고리로 나뉩니다. 

- 짧은 작업을 foreground에서 시작했고 앱이 background로 이동할 때 앱은 시스템에게 작업을 끝낼 수 있는 시간을 요청합니다. 
- foreground 상태에서 다운로드를 시작한 앱은 이에 대한 관리를 시스템에게 넘길 수 있고, 이로써 다운로드가 진행되는 동안 앱은 중지 상태로 이동하거나 아예 종료될 수 있습니다. 
- 특정 유형의 작업을 수행하기 위해 background에서 실행되어야 하는 앱은 하나 이상의 background 실행 모드에 대한 지원을 프로젝트 내에서 선언할 수 있습니다.

사용자 경험을 향상시킬 수 있는 행위가 아니면 background에서 작업을 수행하는 행위는 피하도록 해야합니다. 앱이 background 상태로 가는 것은 사용자가 다른 앱을 실행시켰거나 화면을 잠근 상태로 더 이상 해당 앱을 당장 필요로 하지 않기 때문입니다. 두 상황 모두 사용자가 더 이상 앱에게 의미있는 작업을 기대하지 않는다는 것을 의미합니다. 이러한 상황에서 앱이 계속해서 동작하는 것은 베터리의 수명을 낭비하는 것이고 이는 사용자로 하여금 앱을 강제 종료하는 결과를 초래합니다. 그러므로 background에서 작업을 하는 것에 항상 유의하며 이러한 상황을 피할 수 있다면 피하는 것이 좋습니다. 



#### Executing Finite-Length Tasks

background로 이동한 앱은 가능한 빨리 시스템에 의해 중지 상태로 들어갈 수 있기를 기대합니다. 만약 앱이 작업을 수행하는 중이고 그 작업을 완료하기 위해 약간의 추가 시간을 필요로 한다면 `UIApplication` 객체의 `beginBackgroundTask(withName:expirationHandler:)`나 `beginBackgroundTask(expirationHandler:)` 메소드를 사용하여 작업 수행을 위한 추가 시간을 요청할 수 있습니다. 두 메소드를 호출하는 행위로 작업 수행을 위한 추가 시간을 받아 앱이 중지 상태로 들어가는 것을 지연시킬 수 있습니다. 이렇게 추가 시간을 할당받은 작업이 완료된 후에는 반드시 `endBackgroundTask(_:)` 메소드를 호출하여 시스템에게 작업이 완료되었다는 사실을 알려야 하고 비로소 중지 상태로 들어갈 수 있습니다. 



`beginBackgroundTask(withName:expirationHandler:)`나 `beginBackgroundTask(expirationHandler:)` 메소드 각각의 호출은 호출한 작업에 대응하는 고유한 토큰을 생성합니다. 앱이 작업을 완료하면 반드시 해당 토큰과 함께 `endBackgroundTask(_:)`를 호출하여 시스템에게 작업이 끝났다는 것을 알려야 합니다. background 작업에 대한  `endBackgroundTask(_:)`의 호출에 실패한다면 앱이 종료됩니다. 만일 개발자가 작업을 시작할 때 만료 핸들러를 제공한다면 시스템은 그 핸들러를 호출하고 작업을 종료시키고 앱의 종료를 피할 수 있는 마지막 기회를 제공합니다. 



앱이 지정된 background 작업을 수행하기 위해 background로 이동할 때까지 기다릴 필요는 없습니다. 더 유용한 설계는 작업을 시작하기 전 `beginBackgroundTask(withName:expirationHandler:)`나 `beginBackgroundTask(expirationHandler:)`를 호출하고 끝나는 즉시 `endBackgroundTask(_:)`를 호출하는 것입니다. 앱이 foreground에서 실행되고 있는 상태에서도 이 패턴을 사용하여 작업을 수행할 수 있습니다. 

아래 코드는 앱이 background로 전환되었을 때 많은 시간이 소요되는 작업을 시작하는 방법입니다. 이 예제에서 background 작업 요청은 너무 많은 시간이 소요되는 경우에 대비한 만기 핸들러를 포함하고 있습니다. 그리고 작업은 비동기적으로 실행되기 위해 Dispatch Queue에 제출되고 `applicationDidEnterBackground(_:)`는 정상적으로 반환됩니다. 이런 핸들러에 해당하는 코드 블록의 사용은 background 작업 식별자와 같은 중요한 변수에 대한 레퍼런스 유지를 필요로 하는 코드를 단순화할 수 있습니다.

```swift
func applicationDidEnterBackground(_ application: UIApplication) {
    bgTask = application.beginBackgroundTask(withName: "fetch") {
        // 미완성된 작업을 정리
        application.endBackgroundTask(self.bgTask)
        self.bgTask = UIBackgroundTaskInvalid
    }

    // Start the long-running task and return immediately.
    DispatchQueue.global().async {
        // 작업과 관련된 코드를 작성
        application.endBackgroundTask(self.bgTask)
        self.bgTask = UIBackgroundTaskInvalid
    }
}
```

> 작업을 시작할 때 항상 만기 핸들러를 제공하지만 만일 앱이 백그라운드에서 실행되어야 하는 시간 중 남은 시간이 얼마나 남았는지 알고 싶다면 `UIApplication` 객체의 [`backgroundTimeRemaining`](https://developer.apple.com/documentation/uikit/uiapplication/1623029-backgroundtimeremaining) 프로퍼티를 사용하면 됩니다.

만기 핸들러에서 작업을 마무리하는데 필요한 코드를 포함시킬 수 있습니다. 그러나 어떤 코드던지 작업을 수행하는데 너무 많은 시간이 소요되면 안됩니다. 왜나하면 만기 핸들러가 호출된 그 시점에 앱은 이미 종료까지 얼마 남지 않았기 때문입니다. 이러한 이유로 최소한의 작업만이 이곳에서 수행되어야 합니다.



#### Downloading Content in the Background

파일을 다운로드 받을 때, `URLSession` 객체를 사용하여 다운로드를 시작해야 앱이 일시 중지되거나 종료될 경우 시스템이 다운로드 프로세스를 제어 할 수 있습니다. background 데이터 전송을 위해 `URLSession` 객체를 구성하면 시스템에서 이러한 전송을 별도로 프로세스로 관리하고 그 상태에 대한 정보를 앱에 전달합니다. 만일 데이터 전송이 진행되는 동안 앱이 종료된다면 시스템은 background에서 데이터 전송을 지속하고 전송이 완료되거나 하나 이상의 작업이 앱의 특정한 동작을 필요로 한다면 앱을 다시 실행시킬 것입니다. 

background 데이터 전송을 지원하기 위해선  `URLSession`을 적절하게 구성해야 합니다. 세션을 구성하기 위해선 먼저 `URLSessionConfiguration`객체를 생성하고 여러 프로퍼티에 적절한 값들을 할당해주어야 합니다. 이렇게 생성된 구성 객체를 `URLSession`의 적절한 생성자 메소드에 전달하여 세션을 생성합니다. 

background 다운로드를 지원하기 위한 구성 객체 생성 과정은 다음과 같습니다.

1. `URLSessionConfiguration`의 `background(withIdentifier:)` 메소드를 사용하여 구성 객체를 생성합니다. 
2. 이렇게 생성된 구성 객체의 `sessionSendsLaunchEvents` 프로퍼티에 `true` 값을 할당합니다. 
3. foreground에서 데이터 전송이 시작된다면 `isDiscretionary` 프로퍼티에 값도 `true`로 할당하는 것을 추천드립니다. 
4. 구성 객체의 다른 프로퍼티 값들도 구성하려는 환경에 맞추어 적절하게 할당합니다.
5. 이렇게 생성된 구성 개게를 `URLSession` 객체를 생성하는데 사용합니다. 

이렇게 구성되면 `URLSession` 객체는 적절한 시점에 데이터를 업로드하고 다운로드 받는 작업을 시스템에 위임합니다. 앱이 아직 동작하고 있는 중(foreground, background 상관없이)에 데이터 전송 작업이 완료되면 세션 객체는 자신의 delegate 객체에 이 사실을 알립니다. 만일 작업이 완료되지 않은 상태에서 시스템이 앱을 종료시킨다면 시스템은 자동으로 이 작업을 background에서 지속해서 수행합니다. 하지만 사용자가 앱을 강제로 종료시킨 경우라면 시스템은 남아있는 작업들을 모두 취소합니다. 

background 세션과 관련된 모든 작업이 완료되면 시스템은 종료되었던 앱(`sessionSendsLaunchEvents` 프로퍼티가 `true`이며 사용자가 강제로 종료한 앱이 아니라는 가정)을 재실행시키고 앱 delegate 객체의 `application(_:handleEventsForBackgroundURLSession:completionHandler:)` 메소드를 호출합니다. delegate 메소드를 구현할 때 제공된 식별자를 사용하여 이전과 동일한 구성으로 새로운 `URLSessionConfiguration`객체와 `URLSession` 객체를 생성합니다. 시스템은 새 세션 객체를 이전 작업에 다시 연결하고 해당 상태를 세션 객체의 delegate에 알립니다. 



#### Implementing Long-Running Tasks

작업을 수행하는데 보다 많은 시간이 소요되는 작업을 background에서 중지되는 것 없이 수행하기 위해선 특별한 허가를 요청하야 합니다. iOS에선 특정 유형의 앱들만이 background에서 동작될 권한을 갖고 있습니다. 

- 뮤직 플레이어와 같이 background에서 음원 컨텐츠를 재생하는 앱들
- background에서 오디오 컨텐츠를 녹음하는 앱들
- 네비게이션 앱과 같이 지속해서 사용자들에게 사용자의 위치를 알려주어야 하는 앱들
- VoIP 프로토콜을 지원하는 앱들
- 정기적으로 새로운 컨텐츠들을 다운로드하고 처리해야 하는 앱들
- 외부 악세서리로부터 정기적인 업데이트를 수신받는 앱들

이러한 서비스들을 구현한 앱들은 반드시 그들이 지원하는 서비스들을 명시하고 이를 구현하기 위해 시스템 프레임워크를 사용해야 합니다. 이렇게 사용할 서비스를 선언함으로써 시스템이 해당 서비스를 사용할 것이라는 것을 알게 되지만 몇몇의 경우에는 앱이 중지 상태로 가는 것을 막는 것이 시스템 프레임워크일 수도 있습니다. 



#### Declaring Your App's Supported Background Tasks

특정 유형의 background 실행 작업을 지원하기 위해서는 이를 반드시 명시해주어야 합니다. Xcode 5 이후 버전에서는 이를 프로젝트 세팅의 `Capabilites` 탭에서 명시해줄 수 있습니다. 이외에도 `Info.plist` 파일에서도 `UIBackgroundModes` 키에 원하는 모드를 값으로 할당하여 명시해줄 수 있습니다. 다음은 이렇게 지정할 수 있는 background 모드와 `UIBackgroundModes` 키에 할당해줄 수 있는 값들을 나타내는 표입니다. 

| Xcode background mode             | UIBackgroundModes value | Description                                                  |
| --------------------------------- | ----------------------- | ------------------------------------------------------------ |
| Audio and AirPlay                 | `audio`                 | background에서 오디오 컨텐츠를 제공하거나 오디오를 녹음하는 앱 (음원 스트리밍이나 비디오 AirPlay도 포함). 이러한 기능을 사용하기 위해선 사용자로부터 마이크에 대한 사용 권한을 허가 받아야 합니다. 좀 더 자세한 정보는 [Supporting User Privacy](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/ExpectedAppBehaviors/ExpectedAppBehaviors.html#//apple_ref/doc/uid/TP40007072-CH3-SW6) 문서를 참고하세요. |
| Location updates                  | `location`              | background에서 실행 중일 때도 사용자에게 지속해서 사용자의 위치를 알려주는 앱 |
| Voice over IP                     | `voip`                  | 인터넷 연결을 통해 전화 통하를 가능케 하는 앱                |
| Newsstand downloads               | `newsstand-content`     | background에서 잡지나 신문 컨텐츠를 다운받고 처리하는 앱     |
| External accessoryr communication | `external-accessory`    | 외부 악세서리 프레임워크를 통해 지속해서 하드웨어 악세서리로부터 데이터를 전달받는 앱 |
| Uses Bluetooth LE accessories     | `bluetooth-central`     | Core Bluetooth 프레임워크를 통해 지속적으로 블루투스 악세서리로부터 데이터를 전달받는 앱 |
| Acts as a Bluetooth LE accessory  | `bluetooth-peripheral`  | Core Bluetooth 프레임워크를 통해 주변 장치 모드에서 Bluetooth 통신을 지원하는 앱. 이 모드를 사용하기 위해선 추가적인 인증을 필요로 합니다. 좀 더 자세한 정보는 [Supporting User Privacy](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/ExpectedAppBehaviors/ExpectedAppBehaviors.html#//apple_ref/doc/uid/TP40007072-CH3-SW6) 문서를 참고하세요. |
| Background fetch                  | `fetch`                 | 정기적으로 네트워크를 통해 작은 양의 컨텐츠를 다운로드하고 처리하는 앱 |
| Remote notifications              | `remote-notification`   | 푸시 알람이 도착했을 때 컨텐츠 다운로드를 시작하고 싶은 앱. 이 알람을 통해 푸시 알람과 연관된 컨텐츠를 보여주는데 생기는 딜레이를 최소화할 수 있습니다. |

위의 각 모드는 관련 이벤트에 응답하기 위해 앱이 깨어나거나 시작되어야 한다는 사실을 시스템에 알립니다. 예를 들어 음악을 재생하고 background로 이동하는 앱은 오디오 출력 버퍼를 채우기 위한 추가 작업 시간을 필요로 합니다. 오디오 모드를 활성화하면 적당한 시간 간격으로 앱에 필요한 콜백 메소드를 계속해서 호출해야 한다는 것을 시스템 프레임워크에 알립니다. 앱이 이러한 오디오 모드를 활성화시키지 않고 background로 이동한다면 재생되고 있는 오디오나 녹음되고 있는 오디오가 중지됩니다. 



#### Tracking the User's Location

background에서 사용자의 위치를 추적하는 방법엔 몇 가지가 존재합니다. 대부분의 방법이 background에서 지속적으로 앱이 실행되는 것을 요구하지 않습니다. 

- The significant-change location service (Recommended)
- Foreground-only location services
- Background location services

The significant-change location service 방법은 위치 정보의 높은 정확도를 필요로 하지 않는 앱에서 강력하게 추천되는 방법입니다. 이 방법은 사용자의 위치의 상당한 변화가 있을 때만 위치가 갱신이 됩니다. 그러므로 높은 정확도의 위치 정보를 요구하지 않는 소셜 앱등에 적합합니다. 위치의 갱신이 발생하였을 때 앱이 중지 상태라면 시스템은 위치 갱신을 처리할 수 있도록background에서 앱을 깨웁니다. 앱이 이 서비스를 시작하고 종료되었다면 새로운 위치의 갱신이 가능하다면 시스템은 앱을 자동으로 재실행킵니다. 이 서비스는 iOS 4 버전 이후부터 가능하며 셀룰러 라디오를 포함하는 기기에서만 가능합니다.

Foreground-only service와 background-only 서비스 둘 모두 위치 정보를 가져오기 위해 Core Location 프레임워크를 사용합니다. 둘의 차이점은 foreground-only service는 앱이 중지 상태에 들어가면 위치 갱신 전달을 멈춥니다. Foreground-only service는 foreground에서만 위치 정보를 필요로 하는 앱을 대상으로 합니다.

Xcode 프로젝트 설정의 `Capabilites` 탭에 있는 Background modes 섹션에서 위치 지원을 활성화 할 수 있습니다. 이 모드를 활성화하더라도 시스템이 앱을 중지하는 것을 막을 수는 없습니다. 하지만 갱신할 새 위치 데이터가 있을 때마다 앱을 깨워야 한다고 시스템이 알립니다. 그러므로 background에서 실행되어 위치가 갱신될 때마다 효과적으로 이를 처리할 수 있도록 합니다. 

> [**중요**] 위치 서비스는 iOS 기기의 온보드 라디오 하드웨어의 사용을 요구합니다. 이러한 하드웨어를 지속적으로 사용하는 것은 베터리의 상당한 소모를 불러옵니다. 만일 앱이 정확하고 지속적인 위치 정보를 필요로 하지 않는다면 위치 서비스의 사용을 최소화하는 것이 바람직합니다.

이렇게 각기 다른 위치 서비스를 사용하는 방법에 대해 궁금하시다면 [Location and Maps Programming Guide](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/LocationAwarenessPG/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009497)를 참고하세요.



#### Playing and Recording Background Audio

오디오를 재생하거나 녹음하는 앱(background에서도 동작하는)은 이런 작업들을 background에서 진행되도록 등록할 수 있습니다. Xcode 프로젝트 세팅 `Capabilities` 탭의 Background modes 섹션에서 오디오 지원을 활성화하면 됩니다. background에서 오디오 컨텐츠를 재생하는 앱은 반드시 소리가 있는 오디오 컨텐츠를 재생해야 합니다.

전형적인 background 오디오 앱에는 다음과 같은 것들이 있습니다.

- 뮤직 플레이어 앱
- 오디오 녹음 앱
- AirPlay를 통한 오디오나 비디오 재생을 지원하는 앱
- VoIP 앱

`UIBackgroundModes` 키가 `audio` 값을 포함하고 있다면, 시스템의 미디어 프레임워크는 자동으로 앱이 background로 들어갔을 때 중지되는 것을 막습니다. 오디오나 비디오가 재생되거나 오디오가 녹음되는 동안 앱은 계속해서 background에서 동작하게 됩니다. 하지만 녹음이나 재생이 멈춘다면, 시스템은 앱을 중지시킬 것입니다. 

어떤 시스템 오디오 프레임워크를 사용해도 background에서 오디오 컨텐츠를 재생되게끔 할 수 있으며 이러한 프레임워크를 사용하는 프로세스는 변하지 않습낟. (AirPlay를 통한 비디오 재생의 경우 Media Player나 AVFoundation 프레임워크를 사용해 비디오를 띄울 수 있습니다.) 미디어 파일이 재생되는 동안 앱은 정지되지 않기 때문에 콜백 메소드는 앱이 background에서 동작하는 동안에도 정상적으로 작동합니다. 이러한 콜백에서는 재생을 위한 데이터를 제공하는데 필요한 작업만을 수행하면 됩니다. 예를 들어, 오디오 스트리밍 앱은 서버로부터 음악 스트림 데이터를 받아와 이를 재생하는 작업을 필요로 할 것입니다. 이곳에서는 재생과 관련 없는 작업은 하지 말아야 합니다.

하나 이상의 오디오 기능을 지원하기 때문에 시스템은 주어진 시간에 어떤 앱의 재생과 녹음을 허가해야 할지 결정해야 합니다. foreground에 있는 앱이 항상 오디오 작업에 최우선권을 갖습니다. 하나 이상의 background 앱에서 오디오를 재생하도록 허용할 수 있으며 이러한 결정은 앱의 오디오 세션 객체가 어떻게 구성되어 있는가를 기반으로 결정됩니다. 항상 앱의 오디오 세션 객체를 적절히 구성해야 하며 오디오 관련 알람과 방해(전화 등)를 시스템 프레임워크를 사용해 적절히 다뤄주어야 합니다. background에서 동작하기 위한 오디오 세션 객체 구성하는 법이 궁금하시다면 [Audio Session Programming Guide](https://developer.apple.com/library/archive/documentation/Audio/Conceptual/AudioSessionProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40007875)를 참고해주세요.



#### Implementing a VoIP App

VoIP를 지원하는 앱은 사용자로 하여금 디바이스의 셀룰러 서비스가 아닌 인터넷 연결만으로 통화를 할 수 있게 해줍니다. 이러한 앱들은 네트워크의 지속적으로 연결되어 있어야 앱이 제공하는 서비스와 관련된 정보나 수신오는 전화를 받을 수 있습니다. 시스템은 VoIP 앱을 항상 깨워놓는 것 보단 앱을 정지 상태로 가는 것을 허용하면서 그들의 소켓을 모니터링할 수 있는 기능을 제공합니다. 데이터 수신이 감지되면 시스템은 VoIP 앱을 깨워 소켓에 대한 권한을 앱에 위임합니다.

VoIP 앱을 구성하기 위해선 다음의 항목들을 따라야 합니다.

1. 프로젝트 세팅 `Capabilites` 탭의 Background modes 섹션에서 VoIP 지원을 활성화합니다.
2. VoIP 사용을 위한 앱의 소켓을 구성합니다.
3. Background로 이동하기 전에 [`PushKit`](https://developer.apple.com/documentation/pushkit)을 사용하여 주기적으로 호출될 핸드러를 구성합니다. 여기서 구성된 핸들러를 사용하여 서비스 연결을 유지할 수 있습니다.
4. 데이터의 수신과 발신을 위해 오디오 세션을 구성합니다.

`UIBackgroundModes` 키에 VoIP 값을 포함시키면 시스템이 필요에 따라 앱을 background에서 실행시켜 네트워크 소켓을 관리 할 수 있어야 한다는 사실을 시스템에게 알립니다. 이 키가 존재하는 앱은 시스템 부팅 직후 background에서 재실행되어 항상 VoIP 서비스를 사용할 수 있게 됩니다. 

또한 대부분의 VoIP 앱은 background에서도 오디오를 전달하기 위해 background 오디오 앱처럼 구성될 필요가 있습니다. 그러므로 `UIBackgroundModes`의 키 값으로 `audio`와 `voip`를 포함시켜야 합니다. 이 두 키를 모두 포함하지 않는다면 앱이 background로 들어갔을 때 오디오와 관련된 작업을 할 수 없게 됩니다. `UIBackgroundModes`에 대해 좀 더 알아보고 싶으시다면 [Information Property List Key Reference](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009247) 문서를 참고하세요.

또한 VoIP 앱을 구성하기 위해 거쳐야 하는 단계들에 대해 궁금하시다면 [Tips for Developing a VoIP App](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/StrategiesforImplementingYourApp/StrategiesforImplementingYourApp.html#//apple_ref/doc/uid/TP40007072-CH5-SW13) 문서를 참고하세요.



#### Fetching Small Amounts of Content Opportunistically

주기적으로 새로운 컨텐츠를 필요로 하는 앱은 시스템에게 요청하여 시스템이 앱을 깨워 그들로 하여금 새 컨텐츠를 받아올 수 있는 준비를 할 수 있습니다. 이 모드를 지원하기 위해선 `Capabilities` 탭의 Background modes에서 Background Fetch를 활성화하면 됩니다. 이 모드를 활성화한다고해서 시스템이 앱에 background fetch를 수행할 수 있는 시간을 제공한다는 것을 보장할 수는 없습니다. 앱은 이러한 서비스 제공을 다른 앱들과 그리고 시스템 자체와 균형을 잡아야 하기 때문입니다. 시스템은 적절한 시기에 이러한 시간을 각 앱들에게 제공합니다. 

이런 적절한 시기가 오면 시스템은 background로 부터 앱을 깨우거나 실행시키고 앱 delegate 메소드인 `application(_:performFetchWithCompletionHandler:)` 메소드를 호출합니다. 이 메소드를 사용하여 새 컨텐트츠를 체크하고 컨텐츠가 사용 가능하다면 다운로드 작업을 위한 초기화를 진행합니다. 컨텐츠 다운로드가 완료되자마자 반드시 완료 핸들러 블록을 실행해야 하며 컨텐츠가 사용 가능한지 여부가 담긴 결과를 전달해야 합니다. 이 블록을 실행시킴으로써 시스템은 앱을 중지 상태로 이동시킬 수 있고 이로써 전원 사용량을 향상시킬 수 있습니다. 작은 양의 컨텐츠를 빠르게 다운로드 받고 이를 정확하게 반영할 수 있는 앱은 컨텐츠를 다운로드 받는데 오래 걸리거나 이를 반영하는데 오래걸리는 앱보다 작업을 수행할 수 있는 시간을 시스템으로부터 할당 받을 수 있는 가능성이 더 높습니다. 

컨텐츠를 다운로드 받을 땐 `URLSession` 클래스를 사용하여 이를 시작하고 관리하는 것이 좋습니다. 이 클래스를 사용하여 업로드 및 다운로드 작업을 관리하는 방법에 대한 자세한 내용은 [URL Loading System](https://developer.apple.com/documentation/foundation/url_loading_system)을 참고하세요.



#### Using Push Notifications to Initiate a Download

새 컨텐츠가 앱에서 사용할 수 있을 때 서버가 사용자의 다비이스로 푸시 알람을 보낸다면 앱을 background에서 실행하도록 시스템에게 요청할 수 있고 그러므로 푸시 알람을 받는 그 즉시 바로 다운로드를 시작할 수 있게 됩니다. 이 Background mode의 목적은 알람을 받고 컨텐츠가 사용자에게 보여지게 되는데 필요한 시간차를 최소화하는 것입니다. 일반적으로 앱은 사용자가 푸시 알람을 확인하는 시간과 거의 동시에 일어납니다. 하지만 그래도 푸시 알람을 보고도 그 즉시 확인하지 않을 수도 있기 때문에 작업을 위한 시간은 제공받을 수 있습니다. 

이 기능을 지원하기 위해선 역시 `Capabilites` 탭에서 푸시 알람 기능을 활성화하면 됩니다. 

푸시 알람으로 다운로드 작업을 시작하기 위해선 알람의 `content-available` 키의 값이 `1`로 설정되어야 합니다. 이 키가 존재한다면 시스템은 앱을 background에서 깨워 앱 delegate 메소드인 `application(_:didReceiveRemoteNotification:fetchCompletionHandler:)` 메소드를 호출합니다. 이 메소드 안에서 다운로드에 필요한 코드를 작성해주면 됩니다.

컨텐츠를 다운로드 받을 때 이러한 작업을 `URLSession` 클래스를 이용해 초기화하고 관리하는 것을 추천합니다. 



#### Downloading Newsstand Content in the Background 

새로운 잡지나 뉴스를 다운로드 받는 뉴스 스탠드 앱은 background 다운로드 기능을 `Capabilites`에서 활성화함으로서 이 작업을 background에서 수행할 수 있습니다. `UIBackgroundModes`에 뉴스 스탠드 기능 활성화 키가 존재한다면  동작할 준비가 되지 않은 앱이라도 시스템은 앱을 실행시키고 새로운 컨텐츠를 다운로드 받을 수 있습니다. 

다운로드 초기 설정을 하기 위해 뉴스 스탠드 키트 프레임워크를 사용한다면 시스템이 다운로드의 전반적인 프로세스를 맡게 됩니다. 시스템은 앱이 중지되거나 종료되었어도 다운로드를 계속 진행할 것입니다. 다운로드 작업이 완료되면 시스템은 다운로드 받은 파일을 앱의 샌드박스로 보내고 앱에 알람을 보낼 것입니다. 앱이 동작 중이 아니라면 이 알람은 앱을 깨우고 새로 받은 파일을 처리할 수 있는 기회를 제공합니다. 다운로드 과정에서 에러가 발생한다면 위와 같이 앱은 이 에러를 처리하기 위해서 깨어날 것입니다.

뉴스 스탠드 키트 프레임워크를 사용해 컨텐츠를 다운로드 받는 방법이 궁금하시다면 [Newsstand Kit Framework Reference](https://developer.apple.com/documentation/newsstandkit) 문서를 참고하세요.



#### Communicating with External Accessory

외부 악세서리 장치와 함께 작업을 하는 앱은 앱이 중지 상태일 때 외부 악세서리가 업데이트를 전달하면 깨어날 수 있습니다. 이러한 기능 지원은 심작 박동수를 모니터링하는 장치와 같이 주기적으로 정보를 전달하는 장치를 사용하는데 중요한 역할을 합니다. 이 역시 `Capabilites`에서 기능을 활성화하여 앱 내에서 서비스를 제공할 수 있습니다. 이 모드를 활성화한다면 외부 악세서리 프레임워크는 앱과의 활성화된 세션을 닫지 않고 유지할 것입니다. (iOS 4 포함 이전 버전은 앱이 중지 상태에 들어가면 세션 연결이 끊어졌습니다.) 악세서리로부터 새로운 정보가 전달되면 프레임워크가 앱을 깨워 이 정보를 처리할 수 있도록 합니다. 또한 시스템은 액세서리 연결 및 연결 해제 알림을 처리하도록 앱을 작동시킬 수 있습니다.

background에서 악세서리 업데이트를 처리해야하는 앱은 반드시 다음의 기본적인 가이드라인을 따라야 합니다.

- 앱은 반드시 사용자가 악세서리의 업데이트 이벤트를 받을 것인지 말 것인지를 결정할 수 있는 인터페이스를 제공해야 합니다. 이렇게 만들어진 인터페이스를 통해 악세서리 세션을 적절히 열고 닫을 수 있어야 합니다.


- 앱앱이 깨어나면 앱은 전달받은 정보를 처리할 수 있는 10초의 시간을 갖습니다. 이상적으론 가급적 빠르게 정보를 처리해야 하고 스스로 다시 정지 상태로 들어갈 수 있어야 합니다. 그러나 만일 이보다 시간이 좀 더 필요하다면 `beginBackgroundTask(expirationHandler:)` 메소드를 사용하여 추가 작업 시간을 요청해야 합니다. 이는 반드시 필요한 경우에만 행해져야 합니다. 



#### Communicating with a Bluetooth Accessory

블루투스 주변기기와 작동하는 앱은 앱이 중지 상태일 때 그 주변기기가 업데이트를 전달하면 깨어날 수 있습니다. 이러한 기능 지원은 심작 박동수 측정 벨트와 같이 주기적으로 정보를 전달하는 블루투스-LE 악세서리에겐 매우 중요합니다. 이 역시 `Capablilties`에서 활성화할 수 있습니다. 이 기능을 활성화한다면 Core Bluetooth 프레임워크는 이 주변기기와 연결 가능한 활성화 세션을 지속해서 열어 놓을 것입니다. 게다가 이 주변기기로부터 새로운 정보가 도착하면 시스템은 앱을 깨워 이 정보를 적절히 처리할 수 있도록 합니다. 또한 시스템은 악세서리 연결과 연결 해제 알림을 처리하도록 앱을 작동시킬 수 있습니다. 

iOS 6부터 앱은 스스로 블루투스 악세서리가 되어 주변기기 모드에서 작동할 수 있습니다. 블루투스 악세서리로 행동하기 위해선 이 역시 `Capabilites`에서 기능 지원을 활성화해야 합니다. 이 기능을 활성화함으로서 Core Bluetooth 프레임워크가 background에서 앱을 잠시 깨워 악세서리와 관련된 요청을 처리할 수 있습니다. 이러한 요청을 처리하기 위해 깨어난 앱은 최대한 빠르게 작업을 처리해야 다시 중지 상태로 들어갈 수 있습니다. 

background에서 블루투스 데이터를 처리해야 하는 앱은 반드시 세션 기반이어야 하며 다음의 몇 가지 가이드라인을 따라야 합니다.

- 사용자가 블루투스 이벤트를 받는 행위를 시작하고 멈출 수 있는 인터페이스를 제공해야 합니다. 그리고 이러한 인터페이스는 적절하게 세션을 열고 닫을 수 있어야 합니다. 


- 앱이 깨어나면 앱은 전달받은 정보를 처리할 수 있는 10초의 시간을 갖습니다. 이상적으론 가급적 빠르게 정보를 처리해야 하고 스스로 다시 정지 상태로 들어갈 수 있어야 합니다. 그러나 만일 이보다 시간이 좀 더 필요하다면 `beginBackgroundTask(expirationHandler:)` 메소드를 사용하여 추가 작업 시간을 요청해야 합니다. 이는 반드시 필요한 경우에만 행해져야 합니다. 



#### Getting the User's Attention While in the Background

알람은 중지되어 있거나, background에 존재하거나 혹은 동작하고 있지 않은 앱이 사용자의 주의를 끌 수 있는 방법입니다. 앱은 로컬 알람을 통해 알람을 띄우고 소리를 재생하며 앱의 아이콘에 뱃지를 추가하거나 혹은 이 셋을 함께 혼합할 수 있습니다. 예를들어 알람 시계 앱은 로컬 알람을 통해 알람 소리와 이 알람을 비활성화할 수 있는 알람 창을 띄울 수 있습니다. 알람이 사용자에게 전달되었을 때 사용자는 해당 알람을 보낸 앱을 foregound로 가져올 수 있는지 여부를 결정해야 합니다. (이미 앱이 foreground에 존재한다면 앱은 사용자가 아닌 앱에만 조용히 알람을 전달할 것입니다.)

알람 전송의 스케쥴을 정하고 싶다면 `UNNotificationRequest` 클래스 객체를 만들고 알람에 필요한 프로퍼티를 적절히 구성해야 합니다. 그리고 이 알람의 스케쥴을 정하고 싶다면 `UNUserNotificationCenter` 클래스 메소드를 사용해야 합니다. 로컬 알람 객체는 전달할 알람 타입에 대한 정보(소리, 알람 혹은 뱃지)와 언제 전달할 지를 결정하는 시간 정보를 담고 있어야 합니다. `UNUserNotificationCenter` 메소드는 지금 당장 전달할 것인지 예약된 시간에 전달할 것인지에 대한 옵션을 제공합니다. 

다음의 코드는 사용자에 의해 설정된 시간과 날짜에 단일 알람을 예약하는 예제입니다. 이 예제는 설정된 시간에 하나의 알람을 구성하고 이전에 예약된 알람 한개를 취소하는 예제입니다. (앱은 128개 이상의 로컬 알람을 활성화 할 수 있으며, 이들은 정해진 시간 간격으로 반복되도록 구성될 수 있습니다.) 알람은 앱이 동작하고 있지 않거나 background에 존재할 때 알람이 시작되면 재생되는 음원 파일과 알람 창을 포함하고 있습니다. 앱이 활성화 상태이거나 foreground 상태라면 `userNotificationCenter(_:willPresent:withCompletionHandler:)` 메소드를 대신 호출합니다. 

```swift
let center = UNUserNotificationCenter.current()

center.getPendingNotificationRequests { (request) in
    if request.count > 0 {
        self.center.removeAllPendingNotificationRequests()
    }
}

guard let times = pasreTime(from: time) else { return }

let content = UNMutableNotificationContent()
content.title = title
content.body = body
content.sound = UNNotificationSound.default()
content.badge = 1

let userCalendar = Calendar.current
var components = userCalendar.dateComponents([.hour, .minute], from: Date())

components.hour = times.hour
components.minute = times.minute

let trigger = UNCalendarNotificationTrigger(dateMatching: components, repeats: true)
let request = UNNotificationRequest(identifier: "LocalNotification", content: content, trigger: trigger)

center.add(request) { (error) in
    if ((error) != nil){
        self.vc?.showRegisterFailedAlert()
    }

    UserDefaults.standard.setValue(self.time, forKey: KeyIdentifier.notification.value)
}
```

로컬 알람에 사용되는 음원 파일은 푸시 알람에 사용되는 것과 동일한 요구 사항을 갖습니다. 사용자 정의 음원 파일은 반드시 앱의 매인 번들안에 위치해야 하며 다음 중 하나의 포멧을 지원해야 합니다.

- Linear PCM, MA4, µ-Law 혹은 a-Law

또한 `UNNotificationSound.default()`를 통해 디바이스의 기본 알람 음원을 사용할 수 있습니다. 알람이 전달되고 음원이 재생되면 시스템은 진동을 발생시킬 것입니다. 

`UNUserNotificationCenter` 메소드를 통해 예약된 알람을 취소하거나 알람들의 목록을 가져올 수 있습니다. 로컬 알람을 구성하는 방법에 대해 좀 더 알아보고 싶다면 [Local and Remote Notification Programming Guide](https://developer.apple.com/library/archive/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/index.html#//apple_ref/doc/uid/TP40008194) 문서를 참고하세요. 



#### Understanding When Your App Gets Launched into the Background 

background 실행을 지원하는 앱은 들어오는 이벤트를 처리하기 위해 시스템에 의해 재실행될 수 있습니다. 만일 앱이 사용자에 의한 강제 종료가 아닌 다른 이유에 의해 종료되었다면 다음의 이벤트들에 대해선 시스템이 앱을 다시 실행시킬 것입니다. 

- 위치 앱
  - 시스템에 의해 구성된 기준에 맞는 위치 변화가 일어났을 때
  - 등록된 지역에 들어가거나 그 지역을 벗어났을 때 (**지역**은 지리학적 지역일 수도 있고 iBeacon의 지역일 수도 있습니다.)

- 오디오 앱의 경우 오디오 프레임워크가 데이터를 처리해야 하는 경우 (오디오 앱은 오디오를 재생하거나 마이크를 사용하는 경우 모두를 포함합니다.)


- 블루투스 앱
  - 연결된 주변기기로부터 데이터를 전달받는 중심의 역할을 하는 경우
  - 연결된 중심으로부터 명령을 받는 주변기기의 역할을 하는 경우

- background 다운로드 앱
  - 푸시 알람을 받고 알람이 `1`의 값을 갖는 `content-available` 키를 포함하고 있을 때
  - 시스템이 새로운 컨테츠를 다운로드 받기 위해 앱을 깨우는 경우
  - `URLSession` 클래스를 이용해 background에서 컨텐츠를 다운로드 받는 경우 세션 객체와 관련된 모든 작업이 성공적으로 완수되거나 에러를 받을 때
  - 뉴스 스탠드 앱이 시작한 다운로드가 완료되었을 

대부분의 경우 사용자에 의해 강제 종료된 앱은 시스템이 재실행하지 않습니다. 하나의 예외가 바로 위치 앱으로 iOS 8 이후 사용자에 의해 강제 종료된 앱도 위치 앱은 시스템에 의해 재실행될 수 있습니다. 그 밖의 다른 경우는 사용자가 직접적으로 앱을 실행하거나 디바이스를 재부팅해야 앱이 background에서 시스템에 의해 재실행 될 수 있습니다. 디바이스 암호가 활성화되어 있는 상태라면 시스템은 사용자가 디바이스를 잠금 해제 하기 전까진 앱을 background에서 실행시킬 수 없습니다.



#### Being a Responsible Background App

foreground 앱은 언제나 시스템 자원과 하드웨어 사용에 있어서 background 앱보다 우선권을 갖고 있습니다. Background에서 동작하고 있는 앱은 이러한 자원 불균등에 대비해야 할 필요가 있고 이러한 상황에 background에서 동작하는 행동들을 맞춰야 합니다. background로 이동하는 앱은 반드시 다음의 가이드라인을 준수해야 합니다. 

- **OpenGL ES를 호출해선 안됩니다.** background에서 동작할 때는 `EAGLContext` 객체나 OpenGL ES의 드로잉 관련 명령어를 호추해선 안됩니다. 이러한 종류의 명령어가 호출되면 앱은 그 즉시 종료됩니다. 앱은 또한 이전에 제출된 이와 관련된 명령어는 반드시 background로 가기 전에 완료되어야 함을 보장해야 합니다. background로 진입할 때 혹은 background로부터 나올 때 OpenGL ES를 다루는 방법에 대해 좀 더 알고 싶다면 [OpenGL ES Programming Guide](https://developer.apple.com/library/archive/documentation/3DDrawing/Conceptual/OpenGLES_ProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008793)의 [Implemting a Multitasking-aware OpenGL ES Application](https://developer.apple.com/library/archive/documentation/3DDrawing/Conceptual/OpenGLES_ProgrammingGuide/ImplementingaMultitasking-awareOpenGLESApplication/ImplementingaMultitasking-awareOpenGLESApplication.html#//apple_ref/doc/uid/TP40008793-CH5) 문서를 참고하세요.


- **앱이 중지되기 전 Bonjour와 관련된 서비스를 취소해야 합니다.** 앱이 background로 들아가고 중지되기 전에 Bonjour로부터 등록 해제되어야 하며 네트워크 서비스와 관련된 소켓을 닫아야 합니다. 중지된 앱은 어떠한 방법으로도 들어오는 서비스 요청에 응답할 수 없습니다. 이러한 서비스를 닫게 되면 해당 서비스는 사용할 수 없게 됩니다. Bonjour 서비스를 직접 종료하지 않는다면 앱이 중지되었을 때 자동으로 서비스가 종료됩니다.


- **네트워크 기반 소켓에서의 연결 실패를 처리할 준비를 해야 합니다.** 앱이 중시 상태로 들어간다면 시스템은 모든 소켓 연결을 끊을 것입니다. 소켓 기반 코드가 신호 손실이나 네트워크 전환과 같은 다른 유형의 문제를 대비하고 있다면 다른 문제는 발생하지 않을 것입니다. 앱을 다시 사작할 때 소켓을 사용하여 오류가 발생하면 연결만 다시 설정하면 됩니다.


- **background로 들어가기 전 앱의 상태를 저장하세요.** 메모리가 부족한 상황에서 앱은 메모리 공간을 확보하기 위해 지워질 수 있습니다. 중지된 앱들이 먼저 지워지고 이렇게 지워지기 전에 어떠한 알람도 앱에 전달되지 않습니다. 앱은 iOS 6나 그 이후에 사용할 수 있는 상태 보존 메커니즘을 활용해 앱의 인터페이스 상태를 디스크에 저장해야 합니다. 이러한 기능을 지원하는 방법에 대해 궁금하시다면 [Preserving Your App's Visual Appearance Across Launches](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/StrategiesforImplementingYourApp/StrategiesforImplementingYourApp.html#//apple_ref/doc/uid/TP40007072-CH5-SW2) 문서를 참고하세요.


- **background로 들어갈 때 불필요한 강한 참조들을 제거하세요.** 앱이 메모리 안에 큰 용량의 캐시 데이터를 유지하고 있다면 (특히 사진들) 이에 대한 모든 강한 참조들을 삭제해야 합니다. 더 자세한 사항은 [Reduce Your Memory Footprint](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/StrategiesforHandlingAppStateTransitions/StrategiesforHandlingAppStateTransitions.html#//apple_ref/doc/uid/TP40007072-CH8-SW28) 문서를 참고하세요.


- **중지되기 전 시스템 공유 자원의 사용을 중단하세요.** 앱이 연락처나 캘린더 데이터베이스와 같은 시스템 공유 자원과 상호작용하고 있다면 앱이 중지되기 전 해당 자원들의 사용을 중단하세요. 이러한 자원들의 우선권은 foreground 앱이 갖습니다. 앱이 중지되고 이러한 공유 자원을 사용하고 있다면 앱은 강제 종료될 것입니다.


- **윈도우와 뷰의 갱신은 피하세요.** 앱이 background로 들어갔을 때 윈도우와 뷰는 보이지 않기 때문에 그들을 갱신하는 행위는 피해야 합니다. 앱의 스냅샷을 만들기 전에 윈도우의 컨텐츠를 갱신하는 경우는 예외에 해당합니다. 


- **외부 악세서리 연결과 연결 헤제에 대한 알람에 응답하세요.** 외부 악세서리와 연동되는 앱이 background로 이동한다면 시스템이 자동으로 연결 해제 알람을 앱에 보냅니다. 앱은 반드시 이 알람에 대한 동작을 등록해야 하며 현재 악세서리 세션과의 연결을 끊는데 사용해야 합니다. 앱이 foreground로 복귀한다면 연결 알람을 받을 것이고 앱은 다시 연결될 기회를 얻습니다. 악세서리 연결과 연결 헤제 알람을 다루는 방법은 [External Accessory Programming Topics](https://developer.apple.com/library/archive/featuredarticles/ExternalAccessoryPT/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009502) 문서를 참고하세요.


- **bakground로 가기 전 활성화되어 있는 알람에 대한 자원을 지워야 합니다.** 앱간의 전환에서 컨텍스트를 보존하기 위해 앱이 background로 들어갈 때 시스템은 액션 시트나 알람 뷰를 내리지 않습니다. 앱이 background로 들어갈 때 이러한 행위들을 지워주는 것은 온전히 개발자에 달려있습니다. 예를 들어 액션 시트 혹은 알람 뷰를 취소하거나 해당 뷰들을 다시 띄어주기 위해 당시 컨텍스트를 저장하는 행위를 해줄 수 있습니다.


- **background로 가기 전 뷰 위의 민감한 정보들은 삭제하세요.** 앱이 background로 전환될 때 시스템은 앱 메인 윈도우의 스냅샷을 저정하고 앱이 다시 foreground로 전환될 때 이를 잠깐동안 보여주는데 사용합니다. `applicationDidEnterBackground(_:)`가 반환되기 전 비밀번호나 다른 민감한 개인정보들이 스냅샷의 일부로 캡쳐되기 전에 지워주어야 합니다.


- **background에선 최소한의 작업만을 진행하세요.** background 앱에게 주어지는 작업 가능 시간은 foreground 앱에 비해 극히 제한적입니다. background에서 많은 시간을 소모하는 앱은 시스템에 의해 종료될 수 있습니다.


#### Opting Out of Background Execution

앱이 background에서 전혀 동작하지 않기를 원한다면 `Info.plist`에 `true` 값을 갖는 `UIApplicationExitsOnSuspend` 키를 추가하면 됩니다. 이렇게 되면 앱은 `not-running`, `inactive` 그리고 `active` 상태에서만 순환하며 backgroud나 중지 상태로 들어가진 않습니다. 만일 사용자가 앱에서 나가기 위해 홈 버튼을 누른다면 앱 delegate 메소드인 `applicationWillTerminate(_:)`가 호출될 것이고 앱이 종료되기 전 앱을 정리하고 종료하는데 약 5초의 시간이 소요됩니다. 

이러한 background 상태를 완전 배제하는 행위는 장려되진 않지만 몇몇의 특정 상황에서는 선호되기도 합니다. 특히 background 작업 코드가 앱의 복잡도를 상당히 높인다면 앱을 종료하는게 더 간단한 해결법이기도 합니다. 또한 앱이 상당한 양의 메모리를 사용하며 이들을 쉽게 해지하지 않는다면 시스템은 다른 앱들을 위한 공간을 확보하기 위해 이 앱을 종료시킬 것입니다. 그러므로 background로 전환되는 것보단 종료되는 것이 같은 결과를 보여주면서도 앱 개발의 시간과 노력을 줄일 수 있습니다. 

