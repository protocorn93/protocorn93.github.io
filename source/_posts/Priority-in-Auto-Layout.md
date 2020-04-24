---
title: Priority in Auto Layout
date: 2019-04-14 20:21:16
tags: ["contentHuggingPriority", "contentCompressionResistancePriority", "intrinsicContentSize"]
category: ["iOS"]
---

오늘은 오토레이아웃을 사용하면서 헷갈렸던 주제인 **우선순위(Priority)**에 대해 알아보려 한다. 분명히 우선순위를 사용해서 원하는 레이아웃을 만들어낸 기억이 있다. 하지만 그 이유에 대해 명확히 이해하진 못하고 넘어간 기억이 있다. 

먼저 이 우선순위에 대해 이해하려면 `UIView`의 `intrinsicContentSize`에 대해 이해할 필요가 있다. 



### Intrinsic Content Size 

------

프로퍼티 이름이 직관적이란 걸 알 수 있다. 해석하자면 **본질적인 컨텐츠 사이즈** 정도라고 할 수 있다. 그렇다면 여기서 말하는 본질적인 컨텐츠 사이즈란 무엇일까?

공식 문서에는 다음과 같이 설명하고 있다. 

*The natural size for the receiving view, considering only properties of the view itself.*
*뷰 자체의 특성만 고려한 뷰의 자연적 크기*

뷰 자체의 특성만을 고려했다는 말은 시스템에 의해 결정되는 레이아웃 시스템에 의한 크기가 아닌 **뷰가 갖고 있는 컨텐츠 자체의 의한 크기를 말한다.** 

`UILabel`과 `UIButton`을 예를 들어보자. 

`UILabel`과 `UIButton`의 오토레이아웃을 지정해줄 때 다른 `UIView` 서브클래스들과는 다르게 너비와 높이를 지정해주지 않아도 에러 메시지가 나오지 않는다. 그 이유는 `UILabel`과 `UIButton` 자체가 담고 있는 텍스트로 이 둘의 너비와 높이가 결정되기 때문이다. 

그럼 이제 오늘 포스팅에서 살펴볼 우선순위에 대해 알아보자. 오토레이아웃에서 우선순위는 기본적으로 1에서 1000사이의 값을 갖는데 1은 저수준의 우선순위, 1000은 필수 우선순위이다. **제약 간의 우선순위는 상대적이다.**

### Content Hugging Priority

------

먼저 Content Hugging Priority부터 살펴보자. 역시 이름으로 이 우선순위의 역할을 추측해보자. 그대로 해석해보자면 컨텐츠를 안고 있는 우선순위이다. 예제는 예전에 다뤘던 **[Self-Sizing Table View Cells](https://baked-corn.tistory.com/124)**를 사용하려 한다. 

테이블 뷰 셀 안에는 두 레이블이 존재하고 하나는 제목, 하는 설명을 보여주는 레이블이다. 그리고 설명의 길이에 따라 테이블 뷰 셀의 높이가 동적으로 늘어나야 한다. 여기서 Content Hugging Priority를 사용하는데 우리는 설명 레이블이 늘어났으면 하지 제목 레이블이 늘어나기는 원하지 않는다. 

셀의 크기가 늘어나도 자식 뷰들 중 하나인 제목 레이블은 자신의 사이즈를 지키고 있어야 한다는 것이다. 그렇기 위해선 컨텐츠를 안고 있는 우선순위가 설명 레이블보다 높아야 한다. 

![](https://ehdrjsdlzzzz.github.io/2019/04/14/Priority-in-Auto-Layout/contentHuggingError.png)

만일 두 레이블의 Content Hugging Priority가 같다면 뷰가 늘어났을 때 어떤 레이블이 늘어나야 하는지 몰라 에러 메시지를 보여주는 것이다. 즉 두 레이블의 Vertical 제약이 충돌하고 있는 것이다. 그래서 우리는 Vertical Content Hugging Priority를 조정해주어야 하는 것이다.

한 마디로 정리하자면 **부모 뷰의 크기가 늘어나도 자신의 고유 사이즈를 지킬 수 있는 우선순위**이다.



### Content Compression Resistance Priority

---

그렇다면 Content Compression Resistance Priority란 무엇일까? 늘 그랬듯이 뜻부터 해석해보자. 

컨텐츠 압축 저항 우선순위(?) 정도로 해석할 수 있다. 그러면 여기서 말하는 컨텐츠 압축 저항이란 무엇일까? 외부의 압축을 저항한다면 그 물체는 어떤 압박이 와도 본연의 크기를 유지할 수 있다. 즉 **외부의 압축에도 고유 사이즈를 지킬 수 있는 우선순위**이다. 

Content Hugging Priority와 자신의 크기를 지킨다는 것에는 동일하나 외부 환경 변화가 반대인 것이다. 부모 뷰가 줄어들어도 Vertial 제약을 갖고 있는 다른 뷰들보다 Content Compression Resistance Priority가 높다면 다른 뷰는 부모 뷰가 줄어듦에 따라 줄어들지만 이 우선순위가 높다면 이는 최대한 줄어들지 않을 것이다.



![](https://i.stack.imgur.com/6GelD.png)



**Programmatically**

이 두 우선순위를 인터페이스 빌더를 사용하면 값을 사용해 지정해줄 수 있지만 코드를 사용해 지정해주어야 한다면 다음과 같이 지정해줄 수 있다.

```swift
titleLabel.setContentHuggingPriority(UILayoutPriorityRequired, forAxis: .Vertical)
descriptionLabel.setContentHuggingPriority(UILayoutPriorityDefaultLow, forAxis: .Vertical)
```

우선순위에선 그 값이 중요하지 않다. 단순히 충돌하는 제약과 어떤 제약이 우선순위가 더 높은지만 알면 되기 때문이다.