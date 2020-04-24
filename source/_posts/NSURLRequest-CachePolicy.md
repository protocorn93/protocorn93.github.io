---
title: NSURLRequest.CachePolicy
date: 2019-03-31 19:43:44
tags: ["URLRequest", "NSURLRequest.CachePolicy"]
category: ["iOS"]
---

## NSURLRequest.CachePolicy

[프로토콜 지향적인 네트워크 계층을 작성하는 방법](https://medium.com/flawless-app-stories/writing-network-layer-in-swift-protocol-oriented-approach-4fa40ef1f908)에 대한 글을 보며 읽다가 `NSURLRequest.CachePolicy`가 무엇인지 궁금해서 알아보고 정리한 내용입니다. 

기본적으로 네트워크를 통해 요청을 하게 되면 그에 대한 응답은 여러 방법으로 캐시될 수 있다. 그리고 여기서 언급하는 **정책**은 이러한 캐시 데이터의 상태에 따라 혹은 그와 무관하게 캐시된 데이터를 다루는 정책을 말한다.

`NSURLRequest.CachePolicy`는 **요청**을 생성하는 `URLRequest`를 생성할 때 인자로 함께 넣어준다 .

### URLRequest

------

프로토콜 혹은 URL scheme에 독립적인 URL 로드 요청 (서버에 요청을 보내는 행위가 아닌 요청에 대한 정의)

[`URLRequest`](apple-reference-documentation://hs-y-qEskb)은 로드 요청의 기본적인 두 가지 데이터 요소를 캡슐화한다.

- 로드할 URL
- 캐시 정책



**init(url:cachePolicy:timeoutInterval)**

주어진 url, 캐시 정책 시간 타임 아웃 간격으로 URL 요청을 생성하고 초기화한다. 



**NSURLRequest.CachePolicy**

캐시된 응답과의 상호작용 방법을 지정하는 상수 (기본값 : `.useProtocolCachePolicy`)

캐시 정책 종류

- [`useProtocolCachePolicy`](https://developer.apple.com/documentation/foundation/nsurlrequest/cachepolicy/useprotocolcachepolicy)

  - URL에 대한 요청의 기본 캐시 정책

  - HTTP, HTTPS 프로토콜에서 다음과 같이 동작한다. 

    - 요청에 대한 캐시된 응답이 존재하지 않는다면 URL 로딩 시스템은 **원본(*originating source*)**으로부터 데이터를 받아온다. 
    - 만일 캐시된 응답이 존재하지만 매 시간마다 갱신되어야 한다고 명시되어있지 않거나, 캐시된 응답이 오래되지 않았다면 (*not stale*) URL 로딩 시스템은 캐시된 응답을 반환한다. 
    - 만일 캐시된 응답이 오래되었거나 갱신을 필요로 한다면 URL 로딩 시스템은 원본에 리소스가 변경되었는지 확인하고 변경되었다면 원본으로부터 데이터를 가져오고 그렇지 않다면 캐시된 응답을 반환한다. 

    ![](https://docs-assets.developer.apple.com/published/d4a7d0cbf4/46788d50-fb95-48f2-8360-b8c2a4bf1648.png)

- [`reloadIgnoringLocalCacheData`](https://developer.apple.com/documentation/foundation/nsurlrequest/cachepolicy/reloadignoringlocalcachedata)

  - URL은 반드시 원본에서 로드해야 한다. 

  > HTTP 혹은 HTTPS byte-range 요청을 생성한다면 반드시 이 정책을 사용해야 한다.

- [`reloadIgnoringLocalAndRemoteCacheData`](https://developer.apple.com/documentation/foundation/nsurlrequest/cachepolicy/reloadignoringlocalandremotecachedata)

  - 로컬 캐시 데이터를 무시하고 프록시 및 다른 매개체에게 프로토콜이 허용하는 한 캐시를 무시하도록 지시한다.

  > 이 상수는 미완성 상태이기 때문에 사용해서는 안 된다.

- [`returnCacheDataElseLoad`](https://developer.apple.com/documentation/foundation/nsurlrequest/cachepolicy/returncachedataelseload)

  - 캐시된 데이터가 없을 때만 원본에 요청을 보내고 그 이외에 경우(캐시 데이터가 만료된 상태 포함)에는 모두 캐시된 데이터를 반환한다.

- [`returnCacheDataDontLoad`](https://developer.apple.com/documentation/foundation/nsurlrequest/cachepolicy/returncachedatadontload)

  - 캐시 데이터가 만료된 상태에 상관없이 캐시된 데이터를 반환한다. 만일 캐시된 데이터가 존재하지 않는다면 요청 실패로 간주한다. "오프라인" 모드와 유사한 동작을 한다.

- [`reloadRevalidatingCacheData`](https://developer.apple.com/documentation/foundation/nsurlrequest/cachepolicy/reloadrevalidatingcachedata)

  - 원본에서 캐시 데이터 검증이 가능하다면 캐시 데이터를 사용하고 그렇지 않다면 원본에서 로드한다.

  > 이 상수는 미완성 상태이기 때문에 사용해서는 안 된다.