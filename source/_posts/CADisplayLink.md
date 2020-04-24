---
title: CADisplayLink
subtitle: 부드러운 애니메이션 구현
date: 2019-03-24 16:08:38
tags: ["CADisplayLink", "iOS", "Core Animation"]
category: ["iOS"]
---

`CADisplayLink`는 네이버 지도 v3 api 데모 앱에서 처음 보게 된 클래스이다. 그리고 이를 활용해 데모 앱에선 사용자 위치 마커를 기준으로 원을 그려주는 애니메이션 코드를 작성했다. `CADisplayLink`가 무엇이고 왜 `CADisplayLink`를 사용하는지 알아보자. 

### CADisplayLink

---

먼저 공식 문서에서 이에 대한 정의를 먼저 살펴보자.

*디스플레이의 refresh rate와 drawing 행위를 동기화시켜주는 타이머 객체라고 정의되어 있다.*

즉 앱 디스플레이의 refresh에 맞춰 애니메이션 효과를 넣어주고 싶을 때 사용할 수 있다. 여기서 언급하는 애니메이션은 단순히 뷰의 모양을 바꿔주는 애니메이션이 아닌 값에 의해 뷰의 컨텐츠가 갱신되는 애니메이션이라 생각하면 앞으로 나올 내용을 더욱 이해하기 쉬울 것이다. 

예를 들어 `UILabel` 위에서 일정한 정수값까지 값을 더하는 행위가 그 예이다. 

<img src="http://ehdrjsdlzzzz.github.io/2019/03/24/CADisplayLink/countinglabel.gif">

이를 어떻게 구현할 수 있을까? 굉장히 짧은 시간 동안 뷰를 계속해서 다시 그려주고 있다. 아마 `Timer` 클래스를 생각했을 수 있다. 하지만 이를 사용하기엔 몇 가지 이슈가 존재한다. `Timer`의 런 루프는 많은 인풋을 처리해야 하므로 최소 갱신 주기는 50ms에서 100ms 정도다. 즉 짧은 시간 동안 뷰를 여러 번 갱신시켜주기에 원활하지 않을 수 있다. 

보다 부드러운 뷰의 갱신을 위한 갱신 주기는 60fps 정도는 되어야 한다. 즉 매 0.016초마다 화면을 다시 그려줘야 한다는 의미다. `Timer`로는 부족하다. 이때 사용하는 것이 바로 `CADisplayLink`다. 

`CADisplayLink`는 그리는 행위를 원하는 런 루프와 동기화할 수 있다. 예를 들어 어플리케이션의 매인 런 루프에 동기화를 시키게 된다면 매인 런 루프에선 60fps 주기로 갱신되기 때문에 보다 부드러운 애니메이션을 트리거(trigger) 할 수 있다. 

```swift
let countingDisplayLink = CADisplayLink(target: self, selector: #selector(handleUpdate))
countingDisplayLink.add(to: .main, forMode: .default)
```

뷰 컨트롤러 내부에서 `CADisplayLink`를 만들었고 이를 `.main`과 동기화 시켰다.  

> 인자로 넘기는 타입인 `RunLoop`와 `RunLoop.Mode`에 대해선 추후에 더 살펴보기로 하고 지금은 단순히 메인 런 루프에 기본 모드로 동기화 시킨다고 생각하자

그러면 메인 런 루프가 갱신될 때마다 `selector`에 넣어준 `handleUpdate` 메소드가 호출된다. 즉 `handleUpdate` 메소드가 0.016초마다 호출되는 것이다.

그럼 `handleUpdate` 메소드를 마저 작성해보자. 

```swift
private let animationStartDate = Date()
private let countingAnimationDuration = 3.0

@objc func handleUpdate() {
    let now = Date()
    let elapsedTime = now.timeIntervalSince(animationStartDate) // 애니매이션이 시작한 후 부터 지난 시간
    if elapsedTime > countingAnimationDuration {
        countingDisplayLink.invalidate()
        self.countingLabel.text = "\(Int(endValue))"
        return
    }
    let percentage = elapsedTime / countingAnimationDuration
    let value = startValue + percentage * (endValue - startValue)
    self.countingLabel.text = "\(Int(value))"
}
```

즉 0.016초마다 레이블이 갱신되는 것이다. 

`CADisplayLink`에는 몇 가지 프로퍼티가 존재한다. 

- `timestamp` - 이전 프레임이 그려진 시간
- `timestampTarget` - 다음 프레임이 그려질 시간 

그렇기 때문에 `timestampTarget - timestamp`를 통해 한 루프에 걸리는 시간을 알 수 있다. 위의 레이블을 갱신시켜주는 메소드 안에서 이를 출력시켜보면 `0.016666667001118185` 값을 확인할 수 있다. 

물론 `duration` 프로퍼티를 통해서도 알 수 있지만 이 역시 출력해보면 `timestampTarget - timestamp`가 소수점 아래 더 정교한 값을 알려준다. 또한 `selector`가 최소 1번 호출된 이후에 `duration` 값을 알 수 있다. 

또한 [`preferredFramesPerSecond`](https://developer.apple.com/documentation/quartzcore/cadisplaylink/1648421-preferredframespersecond)에 원하는 값을 넣어주어 주기를 지정할 수도 있다. 

> 지정한 값으로 주기가 정확히 정해지는 것은 아니지만 하드웨어의 성능에 따라 최대한 해당 프레임 수에 가깝게 정해진다. 



그리고 이렇게 `CADisplayLink`를 통해 멋지고 부드러운 다양한 애니메이션을 구현할 수 있다. 

[관련 글](https://medium.com/@duwei199714/animated-uilabel-with-cadisplaylink-9a761d693ca5)



### 마무리

---

오늘은 이렇게 매우 간단하게 `CADisplayLink`의 사용법과 사용 이유를 살펴보았는데 이에 대해 더욱 심화된 내용을 추후에 포스팅할 예정이다. 가볍게 다루려면 가볍게 다룰 수 있지만 무겁게 다루려면 한없이 무거워질 수 있는 주제가 `CADisplayLink`인 것 같다. 



**참고자료**

- [Swift Animations: Custom Counting Label - CADisplayLink](https://youtu.be/b3kZH1vfG2U)
- [CADisplayLink and its applications](https://medium.com/@dmitryivanov_54099/cadisplaylink-and-its-applications-bfafb760d738)



