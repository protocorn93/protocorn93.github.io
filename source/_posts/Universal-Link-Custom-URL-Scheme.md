---
title: Universal Link & Custom URL Scheme
date: 2019-11-25 21:53:20
tags: ["Universal Links", "Custom URL Scheme"]
category: ["iOS"]
---

커스텀 스킴과 유니버셜 링크에 대해 알아보도록 하자. 

이 둘에 대해 얼핏 많이 들어보았을 것이다. 단순히 말해 모두 딥링크를 지원하는 방법이다. 그렇다면 이 둘의 차이점은 무엇일까? 그리고 딥링크의 역할은 무엇이며 딥링크를 통해 우리는 어떤 이점을 얻을 수 있을까. 이것도 함께 알아보도록 하자. 



### 딥 링크란?

위키피디아의 정의에 따르면 딥링크는 **특정 페이지에 도달할 수 있는 링크**를 말한다. 링크란 단어는 실생활 대화에서도 많이 사용하고 실제로 링크도 많이 사용한다. 

*"야~ 내가 링크 보냈어 확인해봐"*

우리는 실생활에서 이런 말을 자주 하곤 한다. 그렇다면 위의 말에 담긴 뜻은 무엇일까? 그것은 바로 정보 전달이다. 정보 전달로는 부족하다. 편리한 정보 전달이다. 그럼 여기서 편리하다는 것은 무엇을 의미할까?

상대방이 내가 알려준 정보, 컨텐츠를 스스로 키워드를 이용하여 검색을 통해 접하는 것이 아닌 링크로 하여금 상대방에게 내가 전달하고자 하는 정보, 컨텐츠를 바로 노출시키는 것이 나는 **편리한 정보 전달의 수단**으로써의 링크라고 생각한다.

위의 *"야~ 내가 링크 보냈어 확인해봐"*의 링크가 딥링크다. 이 딥링크는 단순히 웹 브라우저 상에서만 존재하는 수단이 아니다. 우린 이런 딥링크를 이용해 특정 앱의 특정 컨텐츠를 상대방에게 편리하게 전달할 수 있다. 

이렇게 우리는 딥링크를 통해 사용자에게 우리 앱의 컨텐츠를 **Seamless**하게 노출시킬 수 있다. 

------

### 커스텀 URL 스킴이란?

커스텀 스킴은 iOS에서 딥링크를 지원하는 방법 중 하나이다.

> 현재는 애플에서 유니버셜 링크를 사용하도록 강력하게 추천하고 있다. 
>
> Universal links are strongly recommended as a best practice.
>
> 이 이유에 대해선 밑에서 천천히 살펴보도록 하자. 

공식 문서에서 커스텀 URL 스킴을 <u>앱 내의 자원에 접근할 수 있는 방법을 제공할 수 있는 방법</u>이라고 설명하고 있다. 애플은 시스템 앱들에 대해 미리 정의된 앱 스킴을 제공한다. `mailto`, `tel` 그리고 `facetime`이 그것들이다.

커스텀 URL 스킴을 지원하는 방법은 어렵지 않다. 

1. 앱의 URL 포맷을 정의한다. 

2. 스킴을 등록해 시스템이 정의된 URL에 따라 알맞는 앱으로 사용자를 보낼 수 있도록 한다. 

   ![](https://docs-assets.developer.apple.com/published/3dcc25d414/8239bd2e-1dd6-414e-9754-63f9ac3b0633.png)

3. 앱이 열린 URL을 적절히 처리해야 한다. 

URL은 반드시 정의된 커스텀 스킴 이름으로 시작해야 하며 파라미터를 함께 넘겨 이에 따른 별도의 동작을 처리하게 할 수도 있다. 

```swift
let url = URL(string: "myphotoapp:Vacation?index=1")
       
UIApplication.shared.open(url!) { (result) in
    if result {
       // The URL was delivered successfully!
    }
}
```

위와 같이 정의된 커스텀 스킴은 문서에서도 중복되지 않게 작성하라고 권고하고 있다. 그럼에도 불구하고 커스텀 스킴은 중복이 발생할 수 있다. 그리고 이렇게 중복된 스킴을 갖고 있는 두 앱이 존재한다면 시스템은 사용자가 의도하지 않은 다른 앱을 실행시키게 될 수 있다. 

또한 앱이 설치되어 있지 않다면 스킴은 동작하지 않는다. [`canOpenURL(_:)`](https://developer.apple.com/documentation/uikit/uiapplication/1622952-canopenurl)를 통해 스킴에 해당하는 URL을 열 수 있는지를 검사할 수 있지만 앱이 설치되어 있지 않았을 때 우리가 원하는 궁극적인 행동은 커스텀 URL 스킴으로는 한계가 있다. 

그렇다면 여기서 "우리가 궁극적인 행동"이란 무엇일까? 

우리는 다른 앱에서 우리의 컨텐츠를 그대로 보여주는 것이 아닌 사용자가 우리 앱에서 우리의 컨텐츠를 소비할 수 있도록 이동시키고 싶을 것이다. (사파리를 통해 미디움의 컨텐츠를 보는 것보다, 미디움 앱을 통해 컨텐츠를 보는 것이 보다 쾌적한 사용자 경험을 제공할 수 있다.)

하지만 앱이 설치되어 있지 않았을 때 단순히 이동시키지 못하는 것에 그치지 않고 최소한 우리 앱을 설치할 수 있는 앱 스토어로 이동이라도 시켜준다면 우리 앱으로의 유입은 앞의 상황보다 더욱 나아질 수 있다. 

위의 내용을 통해 우리는 커스텀 URL 스킴의 두 가지 단점을 알 수 있었다.

1. 중복이 일어나 다른 앱을 실행시킬 수 있다.
2. 앱이 설치되어 있지 않을 때 충분한 조치를 취할 수 없다. 

그리고 1번의 이유는 치명적인 보안 이슈의 여부도 있기 때문에 애플은 이러한 커스텀 URL 스킴의 단점을 보완하고자 유니버셜 링크라는 것을 만들었다.

> **중요**
>
> iOS 9.0 이상의 앱에선 열고자 하는 앱의 커스텀 스킴을 `Info.plist`의 `LSApplicationQueriesSchemes`키의 값으로 반드시 등록을 해야 한다. 그렇지 않으면 설령 해당 스킴의 앱이 설치되어 있다 하더라도 `canOpenURL(_:)` 메소드는 항상 `false`를 반환할 것이다.

------

### 유니버셜 링크란?

유니버셜 링크도 커스텀 URL 스킴과 동일하게 모바일 환경에서 딥링크를 지원하기 위한 수단이다. 하지만 유니버셜 링크는 앱이 설치되어 있지 않으면 웹 페이지를 통해 컨텐츠를 보여준다는 것이다. (물론 앱이 설치되어 있다면 바로 해당 앱을 실행시켜 컨텐츠를 보여준다.) 그렇기 때문에 유니버셜 링크는 표준 HTTP, HTTPS 링크다.

> 앱이 설치되어 있지 않을 경우 유니버셜 링크를 통해 컨텐츠를 웹을 통해 노출시키는 방법과 앱 스토어로 사용자를 보내는 방법이 있다.

서비스에 유니버셜 링크를 지원하기 위해선 웹 서버에서의 별도의 작업을 필요로 한다. 앱의 번들 ID와 앱이 열어야 할 경로(path)를 포함하는 **AASA(Apple-App-Site-Association** 파일이 웹 서버에 등록되어 있어야 한다. 

AASA 파일을 서버에 등록해야 하기 때문에 해당 도메인 소유자가 아닌 이상 등록을 할 수가 없다. 그렇기 때문에 고유하고 안전하게 딥링크를 지원할 수 있다. 

AASA 파일은 JSON 형태로 다음과 같이 작성되어야 한다. 

```json
{
    "applinks": {
        "apps": [],
        "details": [{
            "appID": "D3KQX62K1A.com.example.photoapp",
            "paths": ["/albums"]
            },
            {
            "appID": "D3KQX62K1A.com.example.videoapp",
            "paths": ["/videos"]
        }]
    }
}
```

- appID: <team identifier>.<bundle identifier>의 포맷의 값으로 앱의 식별자를 의미하는 값. 

- paths: 앱에서 처리할 수 있는 링크 경로의 배열

  > 와일드카드 문자를 사용해 보다 다양한 경로를 간편하게 지원할 수 있다. 
  >
  > `/videos/samples/201?/*` : `videos/sample`에서 2010년대(`201?`)의 하위 모든 경로를 의미한다.  



> 이미 웹 브라우저를 통해 컨텐츠를 즐기고 있던 사용자가 웹 브라우저 컨텐츠 내부의 유니버셜 링크를 클릭하면 웹 브라우저를 통해 계속해서 컨텐츠를 즐기수 있도록 한다. 계속해서 웹 브라우저에서 컨텐츠를 즐기던 사용자를 앱으로 보내버리는 것은 오히려 사용자 경험을 해칠 수 있기 때문이다.

사용자가 유니버셜 링크를 클릭하여 링크가 활성화되면 iOS는 앱을 실행시키고 [`NSUserActivity`](https://developer.apple.com/documentation/foundation/nsuseractivity) 객체에 정보를 담아보낸다. 그리고 사용자는 `AppDelegate` 메소드인 [application(_:continue:restorationHandler:)](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623072-application)를 오버라이딩하여 처리한다.

유니버셜 링크 활성화를 통해 전달된 `NSUserActivity`의 `activityType`은 [`NSUserActivityTypeBrowsingWeb`](https://developer.apple.com/documentation/foundation/nsuseractivitytypebrowsingweb)이다. 또한 `NSUserActivity` 객체의 [`webpageURL`](https://developer.apple.com/documentation/foundation/nsuseractivity/1418086-webpageurl) 프로퍼티는 사용자가 클릭한 URL을 포함하고 있다. 그리고 [`NSURLComponents`](https://developer.apple.com/documentation/foundation/nsurlcomponents)를 통해 URL의 경로나 파라미터를 추출해낼 수 있다. 

```swift
func application(_ application: UIApplication,
                 continue userActivity: NSUserActivity,
                 restorationHandler: @escaping ([Any]?) -> Void) -> Bool
{
  guard userActivity.activityType == NSUserActivityTypeBrowsingWeb,
  let incomingURL = userActivity.webpageURL,
  let components = NSURLComponents(url: incomingURL, resolvingAgainstBaseURL: true),
  let path = components.path,
  let params = components.queryItems else {
    return false
  }

  print("path = \(path)")

  if let albumName = params.first(where: { $0.name == "albumname" } )?.value,
  let photoIndex = params.first(where: { $0.name == "index" })?.value {

    print("album = \(albumName)")
    print("photoIndex = \(photoIndex)")
    return true

  } else {
    print("Either album name or photo index missing")
    return false
  }
}
```

------

### 번외

일반적인 딥링크가 아닌 **디퍼드 딥링크(Deferred)**라는 개념이 존재한다. 그럼 디퍼드 딥링크는 기존의 딥링크와 다른 점은 무엇일까? 

기존의 딥링크를 통해 우리가 할 수 있는 최선의 행동은 웹을 통해 컨텐츠를 보여주던지, 혹은 앱스토어로 사용자를 보내는 것에 불과하다. 디퍼드 딥링크를 굳이 번역해보자면 **지연된 딥링크** 정도로 해석해볼 수 있을 것이다. 그렇다면 여기서 무엇이 지연되었다는 것일까? 

상황을 가정해보자. 앱이 설치되어 있지 않은 상태에서 기존의 딥링크를 통해 앱스토어까지 사용자를 보냈고 사용자가 앱을 설치했다. 딥링크의 궁극적인 목표는 앱의 설치가 아니라 특정 컨텐츠로 사용자를 보내는 것이다. 그렇기 때문에 현재 상황에서 딥링크 본연의 목적을 달성했다고 보기는 힘들다. 여기서 디퍼드 딥링크가 이런 문제를 해결해 줄 수 있다.

디퍼드 딥링크를 사용하면 링크를 통해 앱스토어로 보내진 사용자가 앱을 설치하고 최초로 앱을 실행시켰을 때 최초 링크를 통해 보여주었어야 할 컨텐츠로 사용자를 이동시킨다. 이렇게 앱스토어 > 앱 설치 > 앱 실행이라는 지연된 과정을 거쳐 컨텐츠를 사용자에게 보여줄 수 있는 기능의 딥링크를 디퍼드 딥링크라고 한다.

그렇다면 링크를 통해 앱을 설치하고 최초로 실행한 사용자와 일반적으로 앱 스토어에서 바로 앱을 설치하여 최초로 실행한 사용자를 어떻게 구분하여 컨텐츠를 보여줄까? 이 부분에 대해서 정확하게 설명된 내용을 찾기 힘들었지만 큰 그림에서 설명해준 글에선 이를 다음과 같이 설명하고 있다. 

 *Historically **this has been done through fingerprinting.** Fingerprinting works by generating a “fingerprint” of a web user **consisting of a device’s IP address and user-agent (operating system, operating system version, and other device specific parameters)** and generating another fingerprint when a user opens the app. Recent techniques like Branch’s People Based Attribution model have moved beyond fingerprinting to using an unique Identity Graph that matches a device’s unique browser identifier to its unique app identifier, tying it back to the actual user across web and app to achieve true cross platform attribution, giving better matching between link licks and app opens.*

------

**참고자료**

1. [Allowing Apps and Websites to Link to Your Content](https://developer.apple.com/documentation/uikit/inter-process_communication/allowing_apps_and_websites_to_link_to_your_content)
2. [Universal links in iOS](https://medium.com/@abhimuralidharan/universal-links-in-ios-79c4ee038272)
3. [Deferred Deep Linking](https://branch.io/glossary/deferred-deep-linking/)