---
title: 'indexOfObject(passingTest:)'
subtitle: 공식 문서가 정답.
date: 2019-03-07 21:57:50
tags: ["Objective-C", "NSNotFound", "indexOfObject(passingTest:)", "firstIndex(where:)"]
category: ["Swift"]
---

## indexOfObject(passingTest:)

오늘은 인턴을 하면서 받은 과제 중 하나인 Objective-C 코드를 Swift로 전환하면서 알게 된 사실을 기록해보고자 한다. 

현재 언어를 전환하면서 깨달은 것은 **"절대 내 생각대로 문법을 선택하지 말자"**이다. 분명히 차이가 있음에도 불구하고 대충 앞 뒤 문맥을 보고 Objective-C 코드를 전환을 하다가 나중에 원인을 못찾아 고생한 경험이 한 두번이 아니다...

오늘은 그 중 한 가지 케이스를 정리해보고자 한다. 그 중심에는 `NSArray` 메소드 중 하나인 [`indexOfObject(passingTest:)`](https://developer.apple.com/documentation/swift/array/2994720-firstindex)다.



### indexOfObject(passingTest:)

------

메소드명과 아규먼트 레이블로 추정해보았을 때 배열에서 해당 조건에 맞는 요소의 인덱스를 반환해주는 것으로 추측했다. 그럼 정의를 살펴보자. 

*Returns the index of the first object in the array that passes a test in a given block.*
*주어진 블록 안의 테스트를 통과하는 배열의 요소 중 첫 번째 요소의 인덱스를 반환한다.*

외형을 살펴보자.

```swift
func indexOfObject(passingTest predicate: (Any, Int, UnsafeMutablePointer<ObjCBool>) -> Bool) -> Int
```

`predicate`가 인자로 받는 세 가지 중 지금 살펴볼 인자는 첫 번째 인자와 두 번째 인자이다. 각각은 배열의 요소(`obj`)와 해당 요소의 인덱스(`idx`)를 나타낸다.

이 블록안으로 넘어가는 배열의 요소는 모두 `Any` 타입으로 넘어간다. 그럼 일단 직접 사용해보자. 

```swift
let nsList: NSArray = [1,2,3,4,5]

let firstEvenNumberOfIndexAtNSList = nsList.indexOfObject { (obj, idx, stop) -> Bool in
    guard let obj = obj as? Int else { return false }
    return obj % 2 == 0
}

// 1
```

배열안의 요소 중 짝수 조건을 만족하는 첫 번째 인덱스(1)을 반환한다. 자 그럼 만약에 리스트 안에 짝수가 존재하지 않는다면 결과는 어떻게 될까?

```swift
let nsList: NSArray = [1,3,5,7]

let firstEvenNumberOfIndexAtNSList = nsList.indexOfObject { (obj, idx, stop) -> Bool in
    guard let obj = obj as? Int else { return false }
    return obj % 2 == 0
}

// 9223372036854775807
```

….??

당황스럽기 그지없다. 대체 저 숫자는 뭐지..? 

다시 문서를 자세히 살펴보자 



**Return Value**

*The lowest index whose corresponding value in the array passes the test specified by `predicate`. If no objects in the array pass the test, returns `NSNotFound`.*
*`predicate`가 지정한 테스트를 통과한 배열의 요소 값 중 가장 낮은 인덱스를 반환한다. 만약 배열의 요소 중 어떤 요소도 테스트를 통과하지 못하면 `NSNotFound`를 반환한다.*



`NSNotFound`…?? 이 친구는 대체 누구지! 이 친구도 문서를 통해 살펴보자.

*A value indicating that a requested item couldn’t be found or doesn’t exist.*
*요청한 아이템을 찾을 수 없거나 존재하지 않음을 나타내는 값이다.*

*`NSNotFound` is typically used by various methods and functions that search for items in serial data and return indices, such as characters in a string object or `id` objects in an `NSArray`object.*
*`NSNotFound`는 일반적으로 문자열 속의 문자들이나 `NSArray` 객체 안에서 `id` 객체를 찾는 것과 같이 연속된 데이터에서 아이템을 찾고 인덱스를 반환하는 다양한 메소드와 함수에서 사용된다.*



즉 `indexOfObject(passingTest:)`에서 반환되는 `9223372036854775807` 값은 `NSNotFound`를 의미하고 `NSNotFound`는 `NSIntergerMax` 값이다. 이는 64비트 컴퓨팅 환경에서 정수 타입이 가질 수 있는 가장 큰 값이다. 

> NSNotFound에 대한 자세한 내용은 추후에 공부해서 기록해 볼 예정이다. 일단 여기선 정수 중 가장 큰 수! 라고 이해하고 넘어가자!



어쨌든! `indexOfObject(passingTest:`의 조건을 통과하는 요소가 없다면 `NSNotFound`를 반환한다!

이게 내가 이번에 알게된 사실 중 하나이고 이 부분을 간과하고 Swift 배열의 어떤 생각없이 메소드를 사용했다. 그럼 그 메소드는 무엇이냐! 



### firstIndex(where:)

------

문서를 통해 정의부터 살펴보자. 

**Returns the first index in which an element of the collection satisfies the given predicate.*
주어진 조건을 만족하는 콜렉션 요소 중 첫 번째 인덱스를 반환한다.* 

**Result Value**

*The index of the first element for which `predicate` returns `true`. If no elements in the collection satisfy the given predicate, returns `nil`.*
*콜렉션에서 만족하는 값이 없으면 `nil`을 반환한다.*



그렇다. 만족하는 요소가 없으면 `NSNotFound`를 반환하는 `indexOfObject(passingTest:)`와 다르게 `firstIndex(where:)`는 `nil`을 반환한다. 이러한 차이가 있음에도 불구하고 이 코드 아래부턴 Swift로 전환한 코드에서도   `NSNotFound`로 비교를 하였기 때문에 당연히 이전 코드와 동일한 조건 검사가 아니게 된 것이다. 이런 사실도 모르고 애꿎은 코드들만 지우고 다시 작성하길 반복했다.



### 정리하는 글

------

절대 내 얄팍한 경험과 지식으로 판단하지말자! 아직 그럴 실력은 아니다…! 애매한 것은 반드시 공식 문서를 참고하자!