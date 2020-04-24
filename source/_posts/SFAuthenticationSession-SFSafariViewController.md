---
title: 'SFAuthenticationSession, SFSafariViewController'
date: 2019-02-13 20:43:38
tags: ["SFAuthenticationSession", "SFSafariViewController"]
category: ["iOS", "Swift"]
---

현재 인턴을 진행하고 있는 회사에서 첫 과제가 주어졌다.  iOS 10.x 버전에서 `SFSafariViewController`를 통해 진행되는 소셜 로그인 중 사소한 버그가 있었고 이를 해결하는 것이었다. 

 iOS 11.0 이상에선 문제가 발생하지 않았다. 이를 위해 문제점을 분석하면서 그 차이에는 `SFAuthenticationSession`과 `SFSafariViewController`가 있다는 것을 발견했다. 

> 이 포스팅은 해당 버그를 해결한 과정을 담은 것이 아닌 둘의 차이점을 살펴보면서 알게된 것들을 정리하는 목적에서 작성되었다. 

 `SFAuthenticationSession`에 대해 살펴보는 것을 위주로 글을 작성해 나갈 것이다. 



### SFAuthenticationSession

---

> 이는 현재는 **Deprecated** 되었다. 

공식 문서를 통해 정의를 살펴보면 다음과 같이 설명되고 있다. 

앱과 사파리 사이의 일회성 로그인 공유를 관리하는 클래스로서 관련 앱의 자동 로그인에도 사용할 수 있다. 

로그인과 함께 이 클래스는 쿠키와 웹사이트 데이터 공유를 관리한다. `SFAuthenticationSession`을 사용할 수 있는 두 가지 경우를 예로 들어보자. 

- 인증 프로토콜(*OAuth 등* )을 사용해 서드 파티 서비스에 로그인을 할 때. 소셜 네트워크 어플리케이션에 적합하다. 
- 어플리케이션에서 SSO(Single Sign-on)를 제공할 때. 이는 동일한 디바이스에 많은 어플리케이션이 설치된 엔터프라이즈 기업에 적합하다. 

> **Note**
>
> 세션이 진행되는 동안 `SFAuthenticationSession` 인스턴스에 대한 강한 참조가 보장되야한다. 

```swift
class ViewController: UINavigationController {
    private var session: SFAuthenticationSession!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        session = SFAuthenticaitonSession(url: URL, callBackURLScheme: nil) { (url, error) in
            // ...
        }
    }
    
    @IBAction func handleProcessSession() {
        session.start()
    }
    
}
```

함수 내에서 생성하고 이를 `start` 시키면 바로 세션은 종료되게 된다.  반드시 세션보다 생명 주기가 긴 객체가 세션에 대한 강한 참조를 유지하고 있어야 한다. (ex `UIViewController`)



iOS 11.0 이상에서 소셜 로그인 등을 진행하다보면 iOS 10과 다르게 어플리케이션이 사파리 상의 웹 사이트 데이터에 접근을 알리는 다이올로그가 나타나는 것을 확인할 수 있다. 

![](http://ehdrjsdlzzzz.github.io/2019/02/13/SFAuthenticationSession-SFSafariViewController/SFAuthenticationSession.png)



웹페이지가 올라오면 이는 다른 프로세스에서 실행되기 때문에 사용자와 웹 서비스는 앱이 사용자의 credential에 접근하지 못함을 보장받을 수 있다. 대신에 앱은 고유한 인증 토큰을 받을 수 있다. 

그리고 좌측 상단의 취소 버튼을 통해 세션이 종료되면 완료 핸들러가 호출되고 고유 토큰을 포함하고 있는 URL(파라미터로 넣어준)로 리다이렉션 된다. 

또한 dismiss 버튼은 항상 `Cancel`이어야 하고 어플리케이션은 공유 시트(Share Sheet)에 자신만의 고유 *UIActivities*를 추가할 수 없다. 그리고 로그인을 방해하는 아이템은 제외된다.  `SFAuthenticationSession`은 인증을 위해서만 설계되었기 때문이다. 그렇기 때문에 민감한 정보가 오가는 웹페이지는 이를 통해 띄우는 것이 바람직하다. 

그리고 `SFSafariViewController`와 다르게 `SFAuthenticationViewController` 위에 `SFSafariView`를 올린다.

iOS 12.0 + 버전부터는 [`ASWebAuthenticationSession`](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession)가 이를 대체하게 된다.