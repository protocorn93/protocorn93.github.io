---
title: All subsets of an array
subtitle: Algorithm
date: 2018-11-14 19:36:36
tags: ["Swift", "Algorithm"]
category: ["Algorithm"]
---

오늘은 배열에서 나올 수 있는 모든 부분 집합을 구하는 알고리즘에 대해 공부해보았다. 본인은 두 가지 다른 로직을 사용하였는데 이 두개를 하나씩 살펴보도록 하자. 

> 알고리즘 초심자라 코드가 굉장히 비효율적일 수 있습니다. 많은 피드백 부탁드리겠습니다.

---

먼저 가장 처음 사용한 방법은 집합(배열)을 하나씩 순회하면서 **해당 값을 추가할지 추가하지 않을지를 결정**해나가는 방법이다. 하나씩 직접 작성해보면서 이해해보자.

배열은 `[a, b, c]`라 가정하자. 

1. 처음 시작은 empty 부분집합으로 시작한다 -  `[]`
2. 0번째 인덱스인 `a`부터 시작해보자.
   1. `a`를 넣을 것 - `[a]`
   2. `a`를 넣지 않을 것 -  `[]`
   3. 결과는 `[[], [a]]`가 된다.
3. 1번째 인덱스인 `b`를 생각해보자. 
   1. `b`를 넣을 것  - `[b], [a, b]`
   2. `b`를 넣지 않을 것. - `[], [a]`
   3. 결과는 `[[], [a], [b], [a,b]]`
4. 2번째, 즉 마지막 인덱스인 `c`를 동일한 방법으로 생각해보자.
   1. `c`를 넣을 것 - `[c], [a, c], [b, c], [a, b, c]`
   2. `c`를 넣지 않을 것 - `[], [a], [b], [a,b]`
   3. 결과는 `[[], [a], [b], [a, b], [c], [a, c], [b, c], [a, b, c]]`

이렇게 모든 인덱스를 순회하면서 이전 부분집합에 해당 인덱스의 값을 추가할지 안할지를 결정해주면 최종적으로 해당 배열의 모든 부분집합을 구할 수 있다. 

이를 코드로 옮겨보면 다음과 같다. 

```swift
var list = ["a", "b", "c", "d"]

func solution(subsets: Set<String>, index: Int) -> Set<String> {
    guard index != list.count else { return subsets } // 탈출 조건
    var newSubsets: Set<String> = []
    subsets.forEach {
        newSubsets.insert($0) // 추가 X
        newSubsets.insert($0 + list[index]) // 추가 O
    }
    return solution(subsets: newSubsets, index: index + 1)
}
```

---

다음 방법은 **깊이 우선 탐색(DFS)**를 이용한 방법이다. 위의 예제와 동일하게 배열은 `[a, b, c]`라 가정하자. 

1. 역시 empty 부분집합부터 시작 - `[]`
2. 첫 번째 익덱스의 값인 `a`부터 DFS 탐색 - `[], [a], [a,b], [a, b, c]`
3. `c`까지 도달하고 `a`까지 백트래킹 - `[a, c]`
4. 백트래킹을 통해 두 번째 인덱스인 `b`에서 탐색 - `[b], [b, c]`
5. 백트래킹을 통해 세 번째 인덱스인 `c`에서 탐색 - `[c]`
6. 결과는 `[[], [a], [a, b], [a, b, c], [a, c], [b], [b, c], [c]]`

이를 코드로 옮겨보면 다음과 같다. 

```swift
var results: [String] = [""]
func solution(_ list: [String], start: Int, pre: String) {
    guard start != list.count else { return }

    for i in start..<list.count {
        let current = pre + list[i]
        results.append(current)
        solution(list, start: i + 1, pre: current)
    }
}
```

---

> 알고리즘 초심자라 코드가 굉장히 비효율적일 수 있습니다. 많은 피드백 부탁드리겠습니다.

