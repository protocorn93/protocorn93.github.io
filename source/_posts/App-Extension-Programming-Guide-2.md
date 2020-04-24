---
title: App Extension Programming Guide
subtitle: App Extension Essentials - Handling Common Scenarios
date: 2018-10-09 19:18:56
tags: ["iOS", "App Extension"]
category: ["iOS","Programming Guide"]
---

> 투데이 익스텐션(위젯)을 개발하면서 필요했고 궁금했던 부분을 공부하고 이를 번역해보았습니다.

### Handling Common Scenarios

앱 익스텐션의 작업을 수행하는 사용자 정의 코드를 작성하게 된다면 다양한 유형의 익스텐션에서 공통적으로 발생할 수 있는 시나리오를 다룰 필요가 있을 수 있다. 이 장의 코드와 권장 사항을 사용해 구현하라.



#### Using an Embedded Framework to Share Code

익스텐션과 그의 컨테이닝 앱에서 코드를 공유하려면 임베디드 프레임워크를 만들어 사용하라. 예를 들어 컨테이닝 앱과 Photo Editing 익스텐션에서 이미지 필터를 개발하기 위해선 필터 관련 코드를 프레임워크에 위치시키고 이 프레임워크를 두 타겟 모두에 임베드해야 한다. 

이렇게 임베드되는 프레임워크에는 [Some APIs Are Unavailable to App Extensions](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/ExtensionOverview.html#//apple_ref/doc/uid/TP40014214-CH2-SW6)에 설명된 것과 같이 앱 익스텐션에서 사용 불가능한 API가 포함되어선 안된다. 만일 사용자 정의 프레임워크가 이러한 API를 포함한다면 컨테이닝 앱과는 안전하게 연결될 수 있지만 이 앱이 포함하는 익스텐션과는 코드의 공유가 불가능하다. 이러한 경우 앱 스토어는 앱을 리젝할 것이다. 

임베디드 프레임워크를 사용하는 앱 익스텐션 타겟을 구성하기 위해선 빌드 세팅의 `Require Only App-Extension-Safe API`를 Yes로 설정하라. 그렇지 않으면 Xcode는 *"linking against dylib not safe for use in application extensions"* 경고 메시지를 지속해서 띄울 것이다. 

> 임베디드 프레임워크가 연결된 컨테이닝 앱은 반드시 arm64(iOS) 혹은 x86_64(OS X) 아키텍쳐 빌드 세팅을 포함해야 하며 그렇지 않으면 앱 스토어는 이를 리젝 할 것이다. 

Xcode 프로젝트를 구성할 때 Copy Files 빌드 단계에서 반드시 임베디드 프레임워크를 `Frameworks`로 선택해야 한다. 

> `Frameworks`가 아닌 `SharedFramework`를 선택한다면 앱 스토어는 이를 리젝 할 것이다. 

임베디드 프레임워크를 만들고 사용하는 방법에 대한 자세한 정보는 [https://developer.apple.com/videos/wwdc/2014](https://developer.apple.com/videos/wwdc/2014/#416) 비디오를 통해 확인해볼 수 있다.



#### Sharing Data with Your Containing App

앱 익스텐션 번들이 컨테이닝 앱 번들에 포함되어 있다 하더라도 실행 중인 앱 익스텐션과 컨테이닝 앱은 직접적으로 서로에게 접근할 수 없다. 

> 컨테이너에 관한 정보는 [File System Programming Guide](https://developer.apple.com/library/archive/documentation/FileManagement/Conceptual/FileSystemProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40010672) 문서의 [About the iOS File System](https://developer.apple.com/library/archive/documentation/FileManagement/Conceptual/FileSystemProgrammingGuide/FileSystemOverview/FileSystemOverview.html#//apple_ref/doc/uid/TP40010672-CH2-SW12) 부분을 참고하라.

하지만 직접적이지 않을 뿐 데이터 공유는 가능하다. 예를 들어 선처리된 에셋들과 같이 단일의 커다란 데이터 묶음을 컨테이닝 앱과 앱 익스텐션이 공유하길 원할 수 있다. 

데이터 공유를 가능하게 하기 위해 Xcode나 개발자 포털을 사용하여 컨테이닝 앱과 앱 익스텐션을 위한 앱 그룹을 활성화해야 한다. 그다음 앱 그룹을 등록하고 컨테이닝 앱에서 이를 사용할 수 있게 명시해야 한다. 앱 그룹에 관련한 더 자세한 정보는 [Adding an App to an App Group](https://developer.apple.com/library/archive/documentation/Miscellaneous/Reference/EntitlementKeyReference/Chapters/EnablingAppSandbox.html#//apple_ref/doc/uid/TP40011195-CH4-SW19) 문서를 참고하라.

앱 그룹을 활성화한 후 앱 익스텐션과 그의 컨테이닝 앱은 `UserDefaults` API를 사용해 데이터를 공유할 수 있다. 이러한 공유를 가능케 하기 위해 `init(suiteName:)` 생성자 메소드에 그룹의 식별자 값을 넣어 `UserDefaults` 객체를 생성해야 한다. 예를 들어 Share 익스텐션에서 다음과 같은 코드를 사용하여 사용자가 가장 최근에 사용한 공유 계정을 갱신할 수 있다. 

```swift
let myUserDefaults = UserDefaults(suiteName: "com.example.domain.MyShareExtension")
myUserDefaults.set(theAccountName, forKey: "lastAccountName")
```

다음 그림은 데이터를 공유하기 위해 어떻게 공유 컨테이너를 사용하는지를 설명하고 있다.

<img src="https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/Art/app_extensions_container_restrictions_2x.png">

> 앱 익스텐션이 `URLSession` 클래스를 사용해 백그라운드 업로드 혹은 다운로드 작업을 수행한다면  반드시 공유되는 컨테이너를 구성해야 한다. 그래야 익스텐션과 컨테이닝 앱 모두 전달되는 데이터에 접근할 수 있다. 백그라운드에서 업로드와 다운로드를 수행하는 방법에 대해서는 [Performing Uploads and Downloads](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/ExtensionScenarios.html#//apple_ref/doc/uid/TP40014214-CH21-SW2) 문서를 참고하라.

공유 컨테이너를 구성하면 컨테이닝 앱과 데이터를 공유하고자 하는 그에 포함되는 익스텐션들은 공유 컨테이너에 대해 읽기와 쓰기 접근 권한을 갖는다.  데이터 무결을 유지하기 위해선 데이터 접근을 동기화해야 한다. 

Core Data, SQLite 혹은 Posix locks를 사용하여 공유 컨테이너의 데이터 접근을 조정하라. 



#### Accessing a Webpage

Share 익스텐션(두 플랫폼 모두)와 Action 익스텐션(iOS만)에선 사파리(Safari)에게 자바스크립트 파일을 실행할 것을 요청하고 그 결과를 다시 익스텐션에게 반환해줌으로써 사용자가 웹 컨텐츠에 접근할 수 있도록 합니다. 또한 자바스크립트 파일을 사용하여 익스텐션이 실행되기 전에 웹 페이지에 접근하거나(두 플랫폼 모두) 익스텐션이 작업을 완료한 후 웹 페이지에 접근하거나 수정할 수 있다. (iOS만) 예를 들어 Share 익스텐션은 사용자가 웹 페이지의 컨텐츠를 공유하는데 도움을 주거나 iOS의 Action 익스텐션을 사용해 사용자의 현재 웹 페이지를 번역할 수 있다. 

웹 페이지 접근과 조정을 익스텐션에 추가하기 위해선 다음의 단계를 거쳐야 한다. 

- `ExtensionPreprocessingJS`라는 이름의 전역 객체가 포함된 자바스크립트 파일을 만들어야 한다. 이 객체에 당신의 사용자 정의 자바스크립트 클래스의 객체를 할당하라.
- 앱 익스텐션의 `Info.plist` 파일 내 `NSExtensionActivateionRule` 딕셔너리에 `NSExtensionActivationSupportsWebPageWithMaxCount` 키와 값을 할당하라. 활성화 규칙에 대한 자세한 정보는 [Declaring Supported Data Types for a Share or Action Extension](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/ExtensionScenarios.html#//apple_ref/doc/uid/TP40014214-CH21-SW8) 문서를 참고하라. 
- 앱 익스텐션이 시작되면 [`NSItemProvider`](https://developer.apple.com/documentation/foundation/nsitemprovider) 클래스를 사용하여 자바스크립트 파일의 실행 결과를 받아올 수 있다. 
- iOS 앱 익스텐션에선 만일 당신이 익스텐션이 작업이 끝난 후 사파리가 웹 페이지를 수정하도록 하고 싶다면 자바스크립트 파일에 값을 넘겨라. 이 단계에서도 `NSItemProvider` 클래스를 사용한다. 

사파리에게 앱 익스텐션이 자바스크립트 파일을 포함하고 있다는 것을 알리기 위해선 `NSExtensionAttributes` 딕셔너리에 `NSExtensionJavaScriptPreprecessingFile` 키를 추가하라. 이 키의 값에는 익스텐션이 시작하기 전 사파리가 로드했으면 하는 파일이어야 한다. 

```xml
<key>NSExtensionAttributes</key>
	<dict>
		<key>NSExtensionJavaScriptPreproccessingFile</key>
        <string>MyJavaScriptFile</string> <!-- Do not include ".js" -->
	</dict>
```

두 플랫폼 모두에서 당신의 사용자 정의 자바스크립트 클래스는 자바스크립트 파일이 로드된 즉시 사파리가 실행시킬 수 있는 `run()` 함수를 정의할 수 있다.  사파리는 앱 익스텐션에 결과를 반환할 수 있는 키-값 객체 형태의  `completionFunction`이라는 이름의 매개변수를 제공한다. 

또한 iOS에선 익스텐션이 작업의 끝에서 `completeRequest(returningItems:completionHandler:)` 호출했을 때 사파리가 깨울 수 있는 `finalize()` 함수를 정의할 수 있다. `finalize()` 함수는 익스텐션이 ``completeRequest(returningItems:completionHandler:)`에 넘기는 아이템들을 사용하여 웹 페이지를 변경할 수 있다. 

예를 들어 iOS 앱 익스텐션이 시작할 때 웹 페이지의 기본 URI가 필요하고 웹 페이지가 멈추면 웹 페이지의 배경색을 변경하고 싶은 경우 다음과 같은 자바스크립트 코드를 작성할 수 있다. 

```javascript
var MyExtensionJavaScriptClass = function() {};
 
MyExtensionJavaScriptClass.prototype = {
    run: function(arguments) {
    // Pass the baseURI of the webpage to the extension.
        arguments.completionFunction({"baseURI": document.baseURI});
    },
 
// Note that the finalize function is only available in iOS.
    finalize: function(arguments) {
        // arguments contains the value the extension provides in [NSExtensionContext completeRequestReturningItems:completion:].
    // In this example, the extension provides a color as a returning item.
    document.body.style.backgroundColor = arguments["bgColor"];
    }
};
 
// The JavaScript file must contain a global object named "ExtensionPreprocessingJS".
var ExtensionPreprocessingJS = new MyExtensionJavaScriptClass;
```

두 플랫폼에서 모두 `run()` 함수가 실행되고 반환되는 값들을 다루는 코드를 작성할 필요가 있다. 결과의 딕셔너리를 얻기 위해선 `NSItemProvider`의 메소드인 `loadItem(forTypeIdentifier:options:completionHandler:)`의  `kUTTypePropertyList` 타입 식별자를 명시해야 한다. 이 딕셔너리에서 `NSExtensionJavaScriptPreprocessingResultsKey` 키값을 사용하여 결과 아이템을 가져올 수 있다. 예를 들어 위의 `run()` 함수에 전달된 기본 URI를 얻으려면 다음과 같은 코드를 사용할 수 있다. 

```swift
imageProvider.loadItem(forTypeIdentifier: kUTTypePropertyList as String, options: nil, completionHandler: { item, error in
    var results = item as? [AnyHashable : Any]
    var baseURI = (results?[NSExtensionJavaScriptPreprocessingResultsKey] as? [AnyHashable : Any])?["baseURI"] as? String
})
```

iOS 앱 익스텐션의 작업이 끝났을 때 `finalize()` 함수에 값을 전달하기 위해선 `NSItemProvider`의  `init(item:typeIdentifier:)` 생성자 함수를 이용해 딕셔너리의 `NSExtensionJavaScriptFinalizeArgumentKey` 키에 값을 전달한다. 예를 들어 `finalize()` 함수에서 빨강색을 배경색으로 지정하기 위해선 앱 익스텐션에 다음과 같은 코드를 작성해야 한다. 

```swift
var extensionItem = NSExtensionItem()
extensionItem.attachments = [NSItemProvider(item: [NSExtensionJavaScriptFinalizeArgumentKey: ["bgColor": "red"]], typeIdentifier: kUTTypePropertyList as String)]
extensionContext.completeRequestReturningItems([extensionItem], completion: nil)
```



#### Performing Uploads and Downloads

사용자는 앱 익스텐션에서 그들의 작업이 끝난 후 즉시 호스트 앱으로 돌아오곤 한다. 만일 그러한 작업이 잠재적으로 시간이 걸리는 업로드와 다운로드를 포함하고 있다면 익스텐션이 종료된 후에도 작업을 끝낼 수 있음을 보장할 필요가 있다. 이러한 업로드와 다운로드 작업을 수행하기 위해선 `URLSession` 클래스를 사용하여 URL 세션을 생성하고 백그라운드 업로드 혹은 다운로드 작업을 시작해야 한다. 

> VoIP 혹은 백그라운드 오디오 재생과 같은 백그라운드 작업은 익스텐션에서 수행될 수 없다. 이에 대한 자세한 정보는 [Respond to the Host App’s Request](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/ExtensionCreation.html#//apple_ref/doc/uid/TP40014214-CH5-SW3) 문서를 참고하라.

앱 익스텐션이 업로드 혹은 다운로드 작업을 시작한 후 익스텐션은 호스트 앱의 요청을 완료할 수 있고 이 작업에 결과에 영향을 미치지 않고 종료될 수 있다. 호스트 앱의 요청을 다루는 방법에 대한 자세한 정보는 [Respond to the Host App’s Request](https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/ExtensionCreation.html#//apple_ref/doc/uid/TP40014214-CH5-SW3) 문서를 참고하라. 만약 백그라운드 작업이 끝났을 때 익스텐션이 실행 중이 아니라면 시스템은 컨테이닝 앱을 백그라운드에서 깨워 앱 델리게이트 메소드인 `application(_:handleEventsForBackgroundURLSession:completionHandler:)` 메소드를 호출할 것이다. 

>만약 앱 익스텐션이 백그라운드 `URLSession` 작업을 시작하면 컨테이닝 앱과 익스텐션이 공유할 수 있는 컨테이너를 반드시 구성해야 한다. `URLSessionConfiguration` 클래스의 `sharedContainerIdentifier` 프로퍼티를 사용하여 추후에 접근할 수 있는 공유되는 컨테이너의 식별자를 지정한다. 

다음은 URL 세션을 구성하고 다운로드를 시작하는 방법 중 하나이다. 

```swift
var configurationMySession: URLSession {
    let config = URLSessionConfiguration.background(withIdentifier: "com.mycompany.myapp.backgroundsession")
    config.sharedContainerIdentifier = "com.mycompany.myappgroupidentifier"
    return URLSession(configuration: config)
}

func downloadTask() {
    let mySession: URLSession = self.configurationMySession
    let url = URL(string: "http://www.example.com/LargeFile.zip")
    let myTask = mySession.downloadTask(with: url!)
    myTask.resume()
}
```

오로지 하나의 프로세스만이 백그라운드 세션을 사용할 수 있기 때문에 컨테이닝 앱과 앱 익스텐션 각각의 백그라운드 세션을 만들어 주어야 한다. (각 백그라운드 세션은 유일한 식별자를 가져야 한다.) 컨테이닝 앱은 백그라운드에서 앱을 실행하여 해당 익스텐션의 이벤트를 처리할 때 익스텐션 중 하나가 만든 백그라운드 세션만을 사용하는 것이 좋다. 만일 컨테이닝 앱에서 네트워크 관련 다른 작업을 필요로 한다면 다른 URL 세션을 만들어서 제공하라.

백그라운드 URL 세션을 시작하기 전 호스트 앱의 요청을 완료할 필요가 있다면 세션을 만들고 사용하는 코드를 효율적으로 작성해야 한다. 앱 익스텐션이 호스트 앱의 요청을 끝낸 후 이를 알리는 `completeRequest(returningItems:completionHandler:)` 메소드를 호출하면 시스템은 언제든지 익스텐션을 종료시킬 수 있다. 