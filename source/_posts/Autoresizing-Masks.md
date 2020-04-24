---
title: Autoresizing Masks
date: 2019-02-07 11:41:16
tags: ["autoresizingMasks", "UIView"]
category: ["iOS"]
---

개발을 할 때 뷰를 코드로 직접 작성할 때가 있다. 이때 항상 뷰를 생성하고 매번 무의식중에 `translatesAutoresizingMaskIntoConstraints` 값을 `false`로 지정해주었다. 지금까지는 단순히 *기본적으로 주어진 레이아웃 설정을 초기화시켜주고 내가 설정할 오토레이아웃으로 레이아웃을 지정할 때* 사용하는 것으로 말 그대로 **대충** 이해하고 있었다. 

오늘은 그래서 **Autoresizing Masks**가 무엇인지 알아보았고 공식 문서와 몇몇 블로그를 참고하여 내가 이해한 내용을 정리해보려 한다. 

### autoresizingMask

---

먼저 공식 문서에서 설명하는 [autoresizingMask](https://developer.apple.com/documentation/uikit/uiview/1622559-autoresizingmask)의 내용부터 살펴보자. `UIView`의 인스턴스 프로퍼티로써 설명되어 있다. 

*An integer bit mask that determines how the receiver resizes itself when its superview’s bounds change.*

*슈퍼 뷰의 바운드가 변경되었을 때 자신의 크기를 어떻게 재조정하는지를 결정짓는 정수형 비트 마스크*

> **bounds**의 사전적 의미를 찾아보면 범위, 영역 그리고 경계선 안의 영토 등으로 소개되어 있다. 즉 크기라고 보면 될 것 같다.

<br>

뷰는 크기가 변경되었을 때 자동으로 자신의 서브뷰들의 크기를 그들의 autoresizing mask에 따라 재조정한다. [`UIView.AutoresizingMask`](https://developer.apple.com/documentation/uikit/uiview/autoresizingmask)에 나와있는 상수를 조합하여 이 값을 지정해줄 수 있다. 이렇게 상수들을 조합해주어 슈퍼 뷰의 크기 변화에 따라 뷰를 어떻게 줄일지 늘릴지를 결정할 수 있다. 디폴트 값은 none으로 기본적으로 뷰의 크기는 변하지 않는다.

동일한 축(axis)을 따라 두 개 이상의 옵션이 설정되었을 때, 그에 따른 기본 동작은 크기 차이를 flexible portion에 비례하여 크기 차이를 적용한다. 다른 flexible portion이 커질수록 뷰의 크기도 커진다. 예를들어 [`flexibleWidth`](https://developer.apple.com/documentation/uikit/uiview/autoresizingmask/1622468-flexiblewidth)와 [`flexibleRightMargin`](https://developer.apple.com/documentation/uikit/uiview/autoresizingmask/1622662-flexiblerightmargin)를 지정하였지만 `flexibleLeftMargin` 을 지정하지 않았다면 뷰의 너비는 왼쪽 마진은 고정된 상태에서 오른쪽 마진이 늘어나며 뷰의 크기가 늘어날 것이다.

만일 autoresizing이 당신의 뷰에 필요한 레이아웃을 정확하게 제공하지 않는다면 컨테이너 뷰의 [`layoutSubviews()`](https://developer.apple.com/documentation/uikit/uiview/1622482-layoutsubviews)를 오버라이드 하여 보다 정확하게 서브뷰들의 레이아웃을 지정할 수 있다.  



### translatesAutoresizingMaskIntoConstraints

---

이번엔 `translatesAutoresizingMaskIntoConstraints`의 정의를 살펴보자. 

*A Boolean value that determines whether the view’s autoresizing mask is translated into Auto Layout constraints.*

*뷰의 autoresizing mask가 Auto Layout 제약조건으로 변환되는지 여부를 결정하는 Bool 타입의 값이다.*

<br>

이 값이 `true`라면, 시스템은 뷰의 autoresizing mask에 정해진 행위들을 제약(*contsraints*)의 집합으로 생성한다. 이는 오토레이아웃 없이 `frame`, `bounds` 혹은 `center` 프로퍼티를 사용해 정적이고 프레임에 기반한 레이아웃을 생성할 수 있도록 도와준다. 

**autoresizing mask는 뷰의 크기와 위치를 완전히 지정한다. 그러므로 뷰의 크기나 위치를 조정하기 위해 추가적으로 제약을 추가해줄 필요가 없다. 오토레이아웃을 사용해 동적으로 크기와 위치를 계산해주고 싶다면 반드시 이 값을 `false`로 지정해야 한다.** 그리고 애매하지 않고 충돌이 없는 제약의 집합을 제공해야 한다. 

코드로 뷰를 생성하게 된다면 이 프로퍼티의 기본 값은 `true`이다. 만일 인터페이스 빌더를 통해 뷰를 추가한다면 시스템은 자동으로 이 프로퍼티의 값을 `false`로 지정할 것이다. 

<br>

이렇게 둘의 정의만 살펴보아도 어느 정도 둘의 관계와 autoresizing mask가 무엇인지 감을 잡을 수 있을 것이다. 기존에 코드로 뷰를 추가하고 오토레이아웃을 추가하였을 때 가끔 다음과 같은 에러 메시지를 콘솔에서 확인할 수 있었을 것이다. 

```
[LayoutConstraints] Unable to simultaneously satisfy constraints.
```

그 이유는 즉슨 `translatesAutoresizingMaskIntoConstraints` 값을 `false`로 지정해주지 않아 기본적으로 지정되어 있는 제약과 새로 추가한 제약과 충돌을 하기 때문이다. 

그렇다면 autoresizing mask는 왜 만들어졌고 언제 사용되었나? 오토레이아웃이 등장하기 이전에 iOS에선 *springs*와 *struts* 레이아웃 시스템을 사용해 레이아웃을 결정해주었다. *springs*는 필요에 따라 늘어나며 너비와 높이를 조정해주었고, *struts*는 컨테이너 뷰(슈퍼 뷰)와의 거리이다. *struts*가 위치를 결정짓는 것이다. 

이는 Xcode 상에서도 시각적으로 확인해볼 수 있는데 우측 사이즈 인스펙터 탭에서 이를 확인할 수 있다. 

![](https://thecodedself.github.io/assets/images/AutoResizing/Autoresizing-Example.gif)

<br>

정리하자면 코드로 뷰를 작성하고 오토레이아웃을 작성하고 싶다면 기본적으로 autoresizing mask가 복사되어 생성된 제약과 충돌을 방지하기 위해 `traslatesAutoresizingMaskIntoConstraints` 값을 `false`로 지정해야 한다. 하지만 인터페이스 빌더를 통해 뷰를 올리는 경우에는 시스템이 이를 자동으로 지정해주기 때문에 이에 대해서 신경을 쓰지 않아도 된다. 

 <br>

**참고**

1. [Autoresizing Masks and You](http://www.thecodedself.com/autoresizing-masks/)

