---
title: SwiftUI에서 여러 타입의 뷰 반환하기
subtitle: (feat. Opaque Types)
date: 2020-01-08 20:16:48
tags: ["Opaque Type", "AnyView", "Group"]
category: ["SwiftUI"]
---

지난 포스팅에서 불투명 타입(Opaque Type)에 대해 공부했었다. 

> 글 : [Opaque Types in Swift](https://ehdrjsdlzzzz.github.io/2019/12/12/Opaque-Types-in-Swift/)

당시 우리는 불투명 타입을 반환할 땐 타입 정체성(Identity)를 잃지 않기 위해 한 가지 타입만 반환해야 한다고 공부했다. 하지만 개발을 하다 보면 여러 예외 상황을 마주하게 되는데 이는 불투명 타입도 마찬가지다. 

먼저 SwiftUI로 프로젝트를 생성했을 때 마주하게 되는 기본 `ContentView` 코드를 살펴보자. 

```swift
struct ContentView: View { 
  var body: some View { 
    return Text("Hello World")
  }
}
```

`body` 프로퍼티는 `some View` 타입이기 때문에 한 가지 타입만 반환할 수 있다. 그래서 뷰 계층을 작성할 때 우리는 아래와 같은 구조에 익숙하다. 

```swift
struct ContentView: View { 
  var body: some View { 
    ...View { 
    	...View { 
        ...View { 
          ...View { 
          
          }
        }
      }
    }
  }
}
```

`some View`를 반환해야 하기 때문에 위와 같은 구조의 코드에 익숙하다. 하지만 조건에 따라 다른 타입의 뷰를 보여주어야 한다면 어떻게 해야 할까?

```swift
struct ContentView: View { 
  var body: some View { 
  	if isLogIn { 
      Image("User-Avatar")
    }else { 
      Text("Please Login")
    }
  }
}
```

이렇게 코드를 작성하면 우리는 `Function declares an opaque return type, but has no return statements in its body from which to infer an underlying type`와 같은 에러 메시지를 볼 수 있다. 

하지만 언제나 방법은 있다! 게다가 두 가지씩이나! 그럼 그 두 가지를 살펴보자. 



### Group

 **`Group`은 어떠한 레이아웃 특성도 지니지 않는다.** `Group` 안에서는 위와 같이 조건 분기를 통한 두 가지 타입을 반환할 수 있다.

```swift
struct ContentView: View { 
  var body: some View { 
    Group { 
      if isLogIn { 
        Image("User-Avatar")
      }else { 
        Text("Please Login")
      }
    }
  }
}
```

오늘의 주제와 별개로 `Group`의 사용 예를 한 가지 더 들어보자면 기본적으로 뷰 빌더 안에는 최대 10개 뷰만 들어갈 수 있다. 10개가 넘어가면 에러 메시지를 출력하는데, 이때 우리는 `Group`을 사용해 레이아웃을 해치지 않으면서 10개 이상의 뷰를 추가할 수 있다.

```swift
struct ContentView: View { 
  var body: some View { 
    VStack { 
      Group { 
      	Text("Text")
      	Text("Text")
      	Text("Text")
      	Text("Text")
      	Text("Text")
      	Text("Text")
      	Text("Text")
      	Text("Text")
      	Text("Text")
      	Text("Text")
      }
      
      Group { 
      	Text("Text")
      	Text("Text")
      	Text("Text")
      	Text("Text")
      	Text("Text")
      	Text("Text")
      	Text("Text")
      	Text("Text")
      	Text("Text")
      	Text("Text")
      }
    }
  }
}
```

---

### AnyView 

`Group ` 말고도 우린 `AnyView`를 통해 두 가지 타입의 뷰를 반환할 수 있다. 정확하게는 타입을 지워(Type erase) 하나의 타입이 반환되는 것처럼 보이게 하는 것이다. 

```swift
struct ContentView: View { 
  var body: some View { 
    if isLogIn { 
      return AnyView(Image("User-Avatar"))
    }else { 
      return AnyView(Text("Please Login"))
    }
  }
}
```

하지만 `AnyView`는 성능 비용이 높기 때문에 사용하는 것을 지양한다. `AnyView`의 공식 문서에는 다음과 같은 내용이 있다. 

***Whenever the type of view used with an `AnyView` changes, the old hierarchy is destroyed and a new hierarchy is created for the new type***

> `AnyView`와 함께 사용된 뷰가 변할 때마다 이전 뷰 계층을 파괴하고 새로 만들기 때문에 이 부분에서 성능 비용이 높다고 설명하는 것 같다. 

하지만 또 이 부분에 대해 [`Group`과 성능 비교를 한 글](https://medium.com/swlh/swiftui-performance-battle-anyview-vs-group-55bf852158df)도 확인할 수 있었다. 이 비교 실험 글에 따르면 사실상 큰 차이는 없다고 나온다. 또한 이 글에선 `body`에서 뷰 상태의 바인딩을 최소한으로 하는 것이 성능 향상에 더 도움이 된다고 언급하고 있다. 

> 상태에 의존하는 뷰가 많을수록 rebuild 횟수가 증가할 가능성이 크기 때문에 이렇게 말하고 있는 것으로 생각된다.



---

오늘은 SwiftUI에서 여러 타입의 뷰를 반환하는 두 가지 방법에 대해 알아보았다. 