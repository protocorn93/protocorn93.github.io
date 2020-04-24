---
title: Union-Find
subtitle: Algorithm
date: 2019-02-06 11:45:21
tags:: ["Union-Find"]
category: ["Algorithm"]
---

> [Raywenderlich's Swift Algorithm Club](https://github.com/raywenderlich/swift-algorithm-club/tree/master/Union-Find)

**Union-Find**는 겹치지 않는 숫자들로 나뉘어진 요소의 집합들을 지속해서 추적할 수 있는 자료구조이다. 이는 **Disjoint-set**이라 불리기도 한다.

예를들어 다음의 집합들을 추적할 수 있는 것이다.  

```
[a, b, f, k]
[e]
[g, d, c]
[i, j]
```

이들은 모두 **disjoint(겹치지 않는)** 상태이다.

Union-Find는 세 가지 기본 기능을 지원한다. 

1. **Find(A)** : 요소 **A**가 어느 부분 집합에 속하는지 알아낸다. `find(b)`는 부분 집합 `[g, d, c]`를 반환할 것이다. 
2. **Union(A,B)** : 요소 **A**와 요소 **B**가 속한 각각의 부분 집합을 하나의 부분 집합으로 합친다.  `union(d, j)`는 `[g, d, c, i, j]`의 결과를 갖는다. 
3. **AddSet(A)** : 요소 A만을 포함하는 새 부분 집합을 만든다. `addSet(h)`는 `[h]` 부분 집합을 생성한다. 

이 자료구조는 비방향성(undirected) [그래프](https://github.com/raywenderlich/swift-algorithm-club/blob/master/Graph)의 연결 요소를 지속해서 추적하는데 가장 흔하게 사용된다. 또한 그래프의 최소 신장 트리를 찾는데 사용되는 **Kruskal** 알고리즘의 구현에도 사용된다. 



### Implementation

---

이를 구현하는데는 여러 방법이 있지만 효과적이고 이해하는데 쉬운 방법을 살펴볼 것이다. **Weighted Quick Union**이 바로 그것이다.

> 다양한 구현 방법은 플레이그라운드에 포함되어 있다. [저장소](https://github.com/raywenderlich/swift-algorithm-club/tree/master/Union-Find)

```swift
public struct UnionFind<T: Hashable> {
    private var index: [T : Int] = [:]
    private var parent: [Int] = []
    private var size: [Int] = []
}
```

구현할 Union-Find 자료구조는 나무([트리 자료구조](https://github.com/raywenderlich/swift-algorithm-club/blob/master/Tree))로 표현되는 부분 집합을 포함하는 하나의 숲이라 볼 수 있다. 

우리가 구현할 방법에서 우리는 각 트리 노드의 부모만 추척하면 된다. 이를 위해 우리는 `parent`라는 배열을 사용할 것이고 `parent[i]`는 노드 `i`의 부모가 되는 것이다. 그리고 노드 `i`는 배열의 인덱스 값이다. 

`parent`가 다음과 같다면,

```
parent [ 1, 1, 1, 0, 2, 0, 6, 6, 6 ]
     i   0  1  2  3  4  5  6  7  8
```

트리의 구조는 다음과 같을 것이다. 

```
      1              6
    /   \           / \
  0       2        7   8
 / \     /
3   5   4
```

숲에 두 트리가 존재하는 것이다. 각각은 요소들의 집합을 의미한다. 

우리는 각 부분 집합에 식별할 수 있는 고유 값을 할당하였다. 그 값은 부분 집합의 트리의 루트 노드의 인덱스를 의미한다. 위의 예제에선 `1`과 `6`이 그 값들이다.

우리는 이 예제에서 두 부분 집합을 갖고 있고 첫 번째는 `1`, 두 번째는 `6`이라는 인식표를 달고 있다. **Find** 연산은 부분 집합 내용이 아닌 인식표를 반환한다.

루트 노드는 `parent[]`에서 자기 자신을 값으로 갖는다. 그러므로 `parent[1] = 1`, `parent[6] = 6`의 상태를 갖는다.



### Add Set

---

기본 세 가지 연산 중 새 집합을 추가하는 연산의 구현부터 살펴보자. 

```swift
public mutating func addSetWith(_ element: T) {
    index[element] = parent.count // 1
    parent.append(parent.count) // 2
    size.append(1) // 3
}
```

1. 새 요소의 인덱스를 `index` 딕셔너리에 저장한다. 딕셔너리에 저장함으로써 빠르게 해당 요소를 가져올 수 있다. 
2. 그리고 새 요소의 인덱스를 `parent` 배열에 추가하여 이 집합을 위한 새 트리를 생성한다. `parent[i]`는 이제 자기 자신을 가리키게 된다. 왜냐하면 추가된 요소는 자신의 트리를 갖고 그 트리 안에는 자신밖에 없기 때문에 자신이 루트 노드가 되기 때문이다. 
3. `size[i]`는 루트가 인덱스 `i`에 있는 트리의 노드 개수를 의미한다. 즉 새 집합은 요소가 하나이기 때문에 1을 추가하는 것이다.  



### Find

---

종종 우리는 주어진 요소를 포함하는 집합이 이미 있는지 없는지를 알아야할 때가 있다. **Find** 연산이 이런 작업을 수행한다. 

```swift
public mutating func setOf(_ element: T) -> Int? {
    if let indexOfElement = index[element] {
        return setByIndex(indexOfElement)
    }else { 
    	return nil
    }
}
```

`index` 딕셔너리에서 해당 요소를 키 값으로 벨류인 인덱스를 찾고 이 인덱스를 `setByIndex(_:)` 함수에 넣어 해당 인덱스의 요소가 속한 집합을 찾아낸다. 

```swift
private mutating func setByIndex(_ index: Int) -> Int {
    if parent[index] == index { // 1
        return index
    }else {
        parent[index] = setByIndex(parent[index]) // 2
        return parent[index] // 3
    }
}
```

트리 자료구조를 이용하고 있기 때문에 재귀 함수를 사용하고 있다. 

각 집합은 트리로 표현되고 루트 노드의 인덱스는 해당 집한을 식별하는 숫자의 역할을 한다는 것을 명심하자. 우리가 찾으려는 요소가 속한 집합의 루트 노드를 찾을 것이고 해당 인덱스를 반환할 것이다. 

1. 먼저 요소의 인덱스가 루트 노드인지를 검사한다 (루트 노드라면 자신의 부모는 자신이기 때문). 그렇다면 바로 함수를 벗어난다. 
2. 그렇지 않다면 현재 노드의 부모 노드를 찾으며 재귀적으로 거슬러 올라간다. (부모의 부모의 부모...) 그리고 최종적으로 해당 요소가 속한 집합의 루트 노드 인덱스를 알게 되면 해당 요소의 현재 부모 노드 인덱스를 집합의 루트 노드 인덱스로 덮어씌운다. 이렇게 하면 다음에 같은 요소를 검사할 때 보다 빠르게 결과 값을 반환할 수 있기 때문이다. 이러한 최적화가 없다면 이 메소드의 시간 복잡도는 **O(n)**이지만 Union 부분에서 다룰 크기 최적화와 함께라면 거의 **O(1)**의 시간 복잡도를 갖는다.
3. 루트 노드의 인덱스를 결과 값으로 반환한다. 

다음은 이러한 과정을 도식화한 것이다. 초기 트리가 다음과 같은 모습이라고 가정하자.

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Union-Find/Images/BeforeFind.png)

`setOf(4)`를 호출했다. 루트 노드를 찾기 위해 4의 부모인 2를 찾고 2의 부모인 7을 찾는다.  이 후에 트리의 모습은 다음과 같이 바뀐다. 

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Union-Find/Images/AfterFind.png)

이 상태에서 다시 한번 `setOf(4)`를 호출한다면 2를 거칠 필요 없이 바로 7을 얻을 수 있게 된다. 그러므로 Union-Find 자료구조를 사용하면 그 스스로 최적화를 진행한다. 

또한 두 요소가 같은 집합에 속하는지 검사하는 메소드도 존재한다. 

```swift
public mutating func isSameSet(_ firstElement: T, secondElement: T) -> Bool {
    if let firstSet = setOf(firstElement), let secondSet(secondElement {
        return firstSet == secondSet
    }else{ 
    	return false 
    }
}
```

이 메소드 역시 `setOf(_:)`를 호출하기 때문에 이 스스로도 트리를 최적화한다. 



### Union(Weighted)

---

마지막 연산은 **Union**이다. 두 집합을 하나의 큰 집합으로 합치는 연산이다.

```swift
public mutating func unionSetsContaining(_ firstElement: T, secondElement: T) {
    if let firstSet = setOf(firstElement), let secondSet = setOf(secondElement) { // 1
        if firstSet != secondSet { // 2
            if size[firstSet] < size[secondSet] { // 3
                parent[firstSet] = secondSet // 4
                size[secondSet] += size[firstSet] // 5
            }else {
                parent[secondSet] = firstSet
                size[firstSet] += size[secondSet]
            }
        }
    }
}
```

1. 두 요소가 속한 집합의 식별값(각 집합 트리의 루트 노드)을 가져온다.
2. 두 요소가 같은 집합에 속하는지 검사하고 맞다면 합치는 연산(union)을 생략한다. 
3. 여기서 위에서 언급한 크기 최적화(**Weighting**)가 등장한다. 크기가 작은 트리를 크기가 큰 트리에 속하게 한다. 크기 배열을 통해 이를 비교한다. 
4. 크기가 작은 트리의 루트 노드의 부모를 크기가 큰 트리의 루트 노드로 지정한다. 속하게 된 것이다. 
5. 두 트리의 크기를 더한다. 

밑의 그림들이 이해를 도울 것이다. 다음과 같이 두 집합이 있다고 가정하자. 

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Union-Find/Images/BeforeUnion.png)

`unionSetsContaining(4, 3)`를 호출했다. 작은 트리가 큰 트리에 붙게 된다. 

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Union-Find/Images/AfterUnion.png)

`setOf`를 메소드 시작에 호출하였기 때문에 크기가 큰 트리도 역시 최적화가 진행된 상태이다. 그러므로 `3`이 `1`에 바로 붙어있는 것을 확인할 수 있다. 

최적화와 함께 진행된 Union 연산은 거의 **O(1)**의 시간 복잡도를 갖는다. 



### Path Compression

---

```swift
private mutating func setByIndex(_ index: Int) -> Int {
    if index != parent[index] {
        // Updating parent index while looking up the index of parent
        parent[index] = setByIndex(parent[index])
    }
    return parent[index]
}
```

이 작업은 트리를 보다 납작하게 만들어주고 이는 시간 복잡도를 보다 **O(1)**에 가깝게 해준다. 



### Complexity Summary

---

##### To process N objects

| Data Structure                          | Union                    | Find                     |
| --------------------------------------- | ------------------------ | ------------------------ |
| Quick Find                              | N                        | 1                        |
| Quick Union                             | Tree height              | Tree height              |
| Weighted Quick Union                    | logN                     | logN                     |
| Weighted Quick Union + Path Compression | Very close, but not O(1) | Very close, but not O(1) |

##### To process M union commands on N objects

| Algorithm                               | Worst-case time |
| --------------------------------------- | --------------- |
| Quick Find                              | M N             |
| Quick Union                             | M N             |
| Weighted Quick Union                    | M + MlogN       |
| Weighted Quick Union + Path Compression | (M + N) logN    |

