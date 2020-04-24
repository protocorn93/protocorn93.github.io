---
title: setUp()과 tearDown() 이해하기
date: 2019-10-16 14:13:41
tags: ["setUp", "tearDown", "Unit Test", "단위 테스트"]
category: ["Unit Test", "단위 테스트"]
---



> 각 테스트 케이스에서 공통으로 사용되는 프로퍼티들은 하나의 테스트 케이스가 끝나면 이를 초기화해주는 작업이 필요하다. 그렇지 않으면 예상한 테스트 결과에서 어긋나는 경우가 있으며, 의외로 그 원인을 찾기 힘들 때가 종종 있었다. 그래서 오늘은 이러한 초기화에 사용되는 두 메소드인 `setUp()` 메소드와 `tearDown()` 메소드에 대해서 알아보았다. 



이들은 테스트 전 초기 상태를 준비하고 테스트가 끝난 후 이를 정리하는 역할을 한다. 

`setUp`과 `tearDown`은 클래스 메소드와 인스턴스 메소드에 동일한 이름으로 정의되어 있다. **클래스 메소드로** 정의된 것은 **모든 테스트 메소드**들의 이전과 이후에 공통으로 필요한 작업을 정의하고, **인스턴스 메소**로 정의된 것은 **각각의 테스트 메소드**들의 이전과 이후에 필요한 작업을 정의한다. 

즉 클래스 메소드로 정의된 메소드는 하나의 테스트 클래스를 테스트할 때 한 번만 호출되고 인스턴스 메소드로 정의된 메소드는 테스트 클래스 내부의 각각의 테스트 메소드의 앞뒤로 호출된다. 즉 여러 번 호출될 수 있다.

> **Note**
>
> `setUp` 클래스 메소드와 `tearDown` 클래스 메소드는 `XCTestCase`에 정의되어 있지만 `setUp` 인스턴스 메소드와 `tearDown` 인스턴스 메소드는 자신의 베이스 클래스인 `XCTest`에 정의되어 있다. 

테스트 메소드 안에서 [`addTeardownBlock(_:)`](https://developer.apple.com/documentation/xctest/xctestcase/2887226-addteardownblock) 메소드를 통해 코드 블록을 선언해줄 수 있는데 이 블록안에서 정의된 코드는 해당 메소드가 끝난 직후, `tearDown` 인스턴스 메소드가 호출되기 전에 호출된다. 그리고 LIFO를 기반으로 순서가 정해진다. 즉 나중에 선언된 블록이 먼저 호출되는 순서를 갖는다. 이렇게 정의된 블록이나, `tearDown` 메소드는  [`continueAfterFailure`](https://developer.apple.com/documentation/xctest/xctestcase/1496260-continueafterfailure) 값의 여부와 상관없이 호출된다.

>  [`continueAfterFailure`](https://developer.apple.com/documentation/xctest/xctestcase/1496260-continueafterfailure) : `boolean` 타입의 값으로 하나의 테스트 메소드 안에서 실패가 발생하면 그 테스트 메소드는 그 즉시 종료할지 여부를 결정한다. 기본 값은 `true`고 테스트 메소드 내부에서 이를 `false`로 지정하면 테스트 실패가 발생했을 때 그곳에서 테스트가 종료된다. 이는 다른 테스트 메소드에 영향을 주지 않는다. 

아래의 코드를 시각화 한 것이 상단의 사진 자료이다. 

```swift
class SetUpAndTearDownExampleTestCase: XCTestCase {

  override class func setUp() { // 1.
    super.setUp()
    // This is the setUp() class method.
    // It is called before the first test method begins.
    // Set up any overall initial state here.
  }

  override func setUp() { // 2.
    super.setUp()
    // This is the setUp() instance method.
    // It is called before each test method begins.
    // Set up any per-test state here.
  }

  func testMethod1() { // 3.
    // This is the first test method.
    // Your testing code goes here.
    addTeardownBlock { // 4.
      // Called when testMethod1() ends.
    }
  }

  func testMethod2() { // 5.
    // This is the second test method.
    // Your testing code goes here.
    addTeardownBlock { // 6.
      // Called when testMethod2() ends.
    }
    addTeardownBlock { // 7.
      // Called when testMethod2() ends.
    }
  }

  override func tearDown() { // 8.
    // This is the tearDown() instance method.
    // It is called after each test method completes.
    // Perform any per-test cleanup here.
    super.tearDown()
  }

  override class func tearDown() { // 9.
    // This is the tearDown() class method.
    // It is called after all test methods complete.
    // Perform any overall cleanup here.
    super.tearDown()
  }
}
```

위의 코드를 시각화 한 것이 아래의 사진 자료다.

![](https://docs-assets.developer.apple.com/published/83218ec0cb/58fa740a-f06d-4b1d-b757-628aef7d105c.png)

인스턴스 `setUp`, `tearDown` 메소드에선 `assert` 문을 정의할 수 있지만, 클래스 메소드에선 해당 `assert` 문은 클래스 메소드 범위 안에 존재하지 않기 때문에 사용할 수 없다.