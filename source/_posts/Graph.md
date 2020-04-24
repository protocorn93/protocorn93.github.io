---
title: Graph
subtitle: Raywenderlich Swift Algorithm Club
date: 2019-01-21 11:50:55
tags: ["Graph", "Data Structure"]
category: ["Data Structure"]
---

>  본 글은 Raywenderlich의 [swift-alogrithm-club](https://github.com/raywenderlich/swift-algorithm-club) 중 **그래프(Graph)** 자료구조를 공부하며 번역한 내용을 기록한 것입니다. 

그래프는 다음과 같은 모습을 보인다. 

![A graph](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Graph/Images/Graph.png)

컴퓨터 과학에서 **그래프**는 **간선(edge)** 집합과 쌍을 이루는 **정점(vertices)** 집합으로 정의된다. 정점들은 원으로 표현되고 간선은 그들 사이의 선으로 표현된다. 간선은 서로 다른 정점을 연결한다. 

> Note: 종종 정점들은 "노드(node)", 간선은 "링크(link)"라 불리기도 한다.

그래프는 소셜 네트워크를 표현할 수 있다. 각각의 사람이 정점이 되고 서로 아는 사람은 간선으로 연결된다.  다음은 이를 표현한 모습이다.

 ![Social network](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Graph/Images/SocialNetwork.png)

그래프는 다양한 모양과 크기를 갖는다. 간선은 **가중치(weight)**를 가질 수 있는데 이는 양수 혹은 음수의 값으로 각각의 간선에 할당된다. 다음의 예제는 비행기의 도시간 비행을 나타내고 있다. 도시는 정점, 비행은 간선이 될 수 있다. 그렇다면 간선의 가중치는 한 도시에서 다른 도시로의 비행기 표 값이나 시간등을 표현할 수 있다.

![Airplane flights](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Graph/Images/Flights.png)

위의 가상의 비행기 그래프에서 샌프란시스코에서 모스코로 가기 위해선 뉴욕을 거쳤을 때 가장 저렴하다. 

또한 간선은 **방향성**을 가질 수 있다. 위에 언급한 예제들에는 방향성이 존재하지 않는다. 즉 위의 예제들에선  Ada가 Charles를 안다면 Charles도 Ada를 안다는 것을 의미한다. 그와 반대로 방향성이 있는 간선에선 이는 단방향 관계를 의미한다. 정점 X에서 정점 Y로의 방향성 간선은 X에서 Y로 연결을 하지만 Y에서 X를 의미하진 않는다. 

비행 예제를 계속해서 살펴보면 샌프란시스코에서 알레스카로의 방향성 간선은 샌프란시스코에서 알레스카로의 비행이 있다는 것을 의미하지만 알레스카에서 샌프란시스코로의 비행은 없다는 것을 의미한다. 

![One-way flights](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Graph/Images/FlightsDirected.png)

다음의 것들도 역시 그래프이다. 

![Tree and linked list](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Graph/Images/TreeAndList.png)

왼쪽은 [트리(Tree)](https://github.com/raywenderlich/swift-algorithm-club/blob/master/Tree)이고 오른쪽은 [링크드 리스트](https://github.com/raywenderlich/swift-algorithm-club/blob/master/Linked%20List)이다. 이들은 단순한 형태의 그래프로 분류된다. 이들 모두 정점(노드)과 간선(링크)을 갖는다.

첫 번째 그래프는 시작점에서 출발해서 다른 정점들을 거쳐 다시 시작점으로 돌아올 수 있는 순환(Cycle)들을 포함한다. 트리는 이러한 순환이 존재하지 않는 그래프이다. 

다른 유형의 그래프는 방향성이 있는 비순환 그래프(*directed acyclic graph, DAG*)이다. 

![DAG](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Graph/Images/DAG.png)

트리와 마찬가지로 이 그래프는 어떤 순환도 갖지 않는다. 하지만 이 그래프는 계층을 형성하지 않으면서 방향성이 있는 간선들을 갖고 있다. 



### Why use graphs?

------

만일 당신이 겪고 있는 문제의 데이터를 정점과 간선으로 표현할 수 있다면 문제를 그래프로 그려 잘 알려진 으[깊이 우선 탐색(DFS)](https://github.com/raywenderlich/swift-algorithm-club/blob/master/Depth-First%20Search), [너비 우선 탐색(BFS)](https://github.com/raywenderlich/swift-algorithm-club/blob/master/Breadth-First%20Search)과 같은 알고리즘으로 해답을 구할 수 있을 것이다. 

예를들어 몇몇의 작업들이 수행되기 위해선 다른 작업들을 기다려야 하는 작업의 목록이 있다고 가정해보자. 이러한 유형은 비순환적 방향성 그래프로 표현될 수 있다.

 ![Tasks as a graph](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Graph/Images/Tasks.png)

각 정점은 작업을 의미하고 두 사이의 간선은 시작 정점은 목적지 정점이 시작되기 전에 반드시 완료되어야 한다는 것을 의미한다. 예를들어 C는 B와 D가 끝나지 않으면 실행될 수 없다. B와 D는 A가 끝나지 않으면 시작될 수 없다. 

이 문제는 그래프를 이용해 표현되었고, [위상 정렬](https://github.com/raywenderlich/swift-algorithm-club/blob/master/Topological%20Sort)을 수행하기 위해 깊이 우선 탐색을 사용할 수 있다. 이 작업은 작업들을 최적의 순서로 나열할 것이고 작업들이 완료되기 위해 기다리는 시간을 최소화할 수 있다. (A, B, D, E, C, F, G, H, I, J, K)

어려운 문제에 직면했을 때 항상 스스로 생각해라 "어떻게 이 문제를 그래프를 사용해서 표현할 수 있을까?" 그래프는 모든 데이터 간의 관계를 나타낸다. 비법은 "관계"를 어떻게 정의하느냐 이다.

만일 당신이 뮤지션이라면 다음의 그래프를 좋아할 것이다.

![Chord map](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Graph/Images/ChordMap.png)

정점들은 C 메이저의 코드들이다. 화음 사이의 관계를 나타내는 간선은 한 화음이 다른 화음을 따라 갈 가능성을 나타낸다. 이는 방향 그래프고 그렇기 때문에 어떻게 한 화음에서 다음 화음으로 가야하는지를 보여준다. 이는 또한 가중치 그래프로 간선의 가중치는 선의 두꺼움으로 표현되고 이는 두 화음간의 강항 관계를 보여준다.

아마 당신은 그래프를 알기 전에 이미 그것을 사용했을 수 있다. 데이터 모델 역시 그래프이다.

![Core Data model](https://camo.githubusercontent.com/3a4f3221d1868e723cd895404aebc9074e1b9074/68747470733a2f2f646576656c6f7065722e6170706c652e636f6d2f6c6962726172792f696f732f646f63756d656e746174696f6e2f436f636f612f436f6e6365707475616c2f436f72654461746156657273696f6e696e672f4172742f7265636970655f76657273696f6e322e302e6a7067)

개발자들이 사용하는 또 하나의 흔한 그래프는 바로 상태 기계(state machine)로 간선은 상태 간의 전환에 필요한 조건을 표현한다. 다음은 그 예제이다.

 ![State machine](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Graph/Images/StateMachine.png)

그래프는 아주 유용하다. 페이스북은 소셜 그래프로 수익을 만들어 냈다. 자료 구조를 배우려면 그래프와 표준 그래프 알고리즘의 방대한 그래프 알고리즘들을 선택해야 한다. 



### Vertics and edges, oh my!

------

이론상으론 그래프는 단지 정점과 간선 객체의 집합이다. 하지만 이를 어떻게 코드로 표현할까?

여기에는 두 가지 접근 방법이 있다. 

- 인접 리스트 (adjacency list)
- 인접 행렬 (adjacency matrix)

**인접 리스트**

인접 리스트 구현에 있어서 각 정점은 자신으로부터 시작하는 간선들의 리스트를 갖는다. 다음의 그림을 예를들어 정점 A는 B와 D, B. 그리고 C로 향하는 간선을 갖고 있기 때문에 3개의 간선을 포함하는 리스트를 갖는다. 

![Adjacency list](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Graph/Images/AdjacencyList.png)

인접 리스트는 바깥으로 향하는 간선들을 표현한다. 그렇기 때문에 A에서 B로 향하는 간선이 A가 갖고 있는 리스트에는 포함되어 있지만 B에서 A로는 향하지 않기 때문에 B가 갖고 있는 리스트에는 A가 포함되어 있지 않다. 간선을 찾거나 두 정점 사이의 가중치를 찾기 위한 **검색 비용이 비싸다**. 왜냐하면 리스트의 형태이기 때문에 무작위 접근(*random access*)이 불가능하기 때문이다. 검색을 위해선 인접 리스트를 순회해야 한다. 

**인접 행렬**

인접 행렬의 구현에 있어서 행렬의 행과 열은 연결되어 있는 연결되어 있는 정점 사이의 가중치를 표현한다. 예를들어 A에서 B로 가중치가 5.6인 간선이 있다면 A행 B열에 해당하는 값은 5.6이 된다. 

![Adjacency matrix](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Graph/Images/AdjacencyMatrix.png)

**새로운 정점을 추가하는 비용이 비싸다.** 왜냐하면 새로운 행렬 구조는 새 행과 열을 포함할 수 있을 정도의 충분한 공간을 갖는 행렬로 만들어져야 하기 때문이다. 그리고 기존의 구조는 새로운 구조로 복사될 것이다. 

그렇다면 인접 리스트와 행렬 중 어떤 것을 사용해야할까? 대부분의 경우 인접 리스트가 좋은 방법이다. 둘을 좀 더 자세히 비교해보자며 다음과 같다. 

V를 그래프의 정점 개수, E를 간선의 개수라고 하자. 그러면 다음과 같은 결과를 확인할 수 있다. 

| Operation   | Adjacency List | Adjacency Matrix |
| :---------- | -------------- | ---------------- |
| 저장 공간   | O(V + E)       | O(V^2)           |
| 정점 추가   | O(1)           | O(V^2)           |
| 간선 추가   | O(1)           | O(1)             |
| 인접성 검사 | O(V)           | O(1)             |

"인접성 검사"는 주어진 정점과 다른 점들이 이웃하는지(연결되어 있는지)를 검사하는 것을 의미한다. 인접 리스트에서 인접성을 검사에는 **O(V)**의 시간복잡도를 갖는다. 왜냐하면 최악의 경우 한 정점이 모든 정점과 연결되어 있을 수 있기 때문이다. 

간선이 적게 존재하는 희소 그래프(*sparse graph*)에선 인접 리스트가 간선을 저장하기 위한 최고의 방법이다. 하지만 그래프의 간선이 많은 밀집 그래프(*dense graph*)에선 인접 행렬이 선호된다. 

다음은 인접 리스트와 인접 행렬을 구현한 예제이다. 



### The code: edges and vertices

------

인접 리스트에서 정점은 `Edge` 객체를 구성한다. 

```swift
public struct Edge<T>: Equatable where T: Equatable, T: Hashable {
    public let from: Vertex<T>
    public let to: Vertex<T>
    
    public let weight: Double?
}
```

이 구조체는 `from`과 `to`로 정점을 `weight`로 가중치를 표현한다. `Edge` 객체는 항상 단방향 연결이라는 것을 명심하자. 만일 방향성이 없는 연결을 원한다면 반다 방향을 향하는 `Edge` 객체도 추가해야 한다. 각 `Edge`는 선택적으로 가중치를 갖기 때문에 가중치의 유무에 따른 그래프를 모두 표현할 수 있다. 

`Vertex`는 다음과 같이 작성할 수 있다. 

```swift
public struct Vertex<T>: Equatable where T: Equatable, T: Hashable {
    public var data: T
    public let index: Int
}
```

이는 제네릭 타입 `T`의 임의의 값을 갖고 이는 유일함을 보장받는다. (`Hashable`)  정점들은 그들 스스로 `Equatable`하다.



### The code: graphs

------

> Note: 그래프를 구현하는데는 많은 방법이 존재한다. 여기서 구현하고 있는 코드는 그들 중 하나일 뿐이다. 이 코드를 당신의 문제 해결에 맞추어 코드를 수정할 수 있다. 간선은 가중치를 전혀 갖지 않을 수도 있고, 방향성과 방향성이 없는 그래프를 구분할 필요가 없을 수 있다. 

다음은 코드로 구현할 그래프의 예제이다.

![Demo](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Graph/Images/Demo1.png)

우리는 이를 인접 행렬과 인접 리스트로 표현할 수 있다. 이 둘의 컨셉을 구현하고 있는 클래스들은 모두 공통 API인 `AbstractGraph`를 상속받고 안에선 다른 자료구조를 사용하고 있지만 동일한 외형을 가질 수 있다. 

방향성이 있고 가중치가 있는 그래프를 작성해보자.

```swift
for graph in [AdjacencyMatrixGraph<Int>(), AdjacencyListGraph<Int>()] {
    let v1 = graph.createVertex(1)
    let v2 = graph.createVertex(2)
    let v3 = graph.createVertex(3)
    let v4 = graph.createVertex(4)
    let v5 = graph.createVertex(5)
    
    graph.addDirectedEdge(v1, to: v2, withWeight: 1.0)
    graph.addDirectedEdge(v2, to: v3, withWeight: 1.0)
    graph.addDirectedEdge(v3, to: v4, withWeight: 4.5)
    graph.addDirectedEdge(v4, to: v1, withWeight: 2.8)
    graph.addDirectedEdge(v2, to: v5, withWeight: 3.2)
}
```

위에서 언급했듯이 방향성이 없는 간선을 만들기 위해선 두 개의 방향성이 있는 간선이 필요하다.

```swift
  graph.addUndirectedEdge(v1, to: v2, withWeight: 1.0)
  graph.addUndirectedEdge(v2, to: v3, withWeight: 1.0)
  graph.addUndirectedEdge(v3, to: v4, withWeight: 4.5)
  graph.addUndirectedEdge(v4, to: v1, withWeight: 2.8)
  graph.addUndirectedEdge(v2, to: v5, withWeight: 3.2)
```

`withWeight`의 값으로 `nil`을 전달해주면 가중치가 없는 그래프를 구현할 수 있다. 



### The code: adjacency list 

------

인접 리스트를 유지하기 위한 간선과 정점을 매핑하는 클래스가 있다. 그래프는 이런 객체의 배열을 관리하고 필요에 따라 이를 수정한다. 

```swift
private class EdgeList<T> where T: Equatable, T: Hashable {
    var vertex: Vertex<T>
    var edges: [Edge<T>]? = nil
    
    init(vertex: Vertex<T>) {
        self.vertex = vertex
    }
    
    func addEdge(_ edge: Edge<T>) {
        edges?.append(edge)
    }
}
```

이들은 구조체와 다르게 클래스로 구현되어 있기 때문에 이미 간선 리스트를 갖고 있는 시작 정점에 새로운 간선을 정점에 추가하는 것과 같은 행위에 있어서 참조(*reference*)를 이용해 이를 구현할 수 있다.

```swift
open override func createVertex(_ data: T) -> Vertex<T> {
    // 이미 존재하는 정점인지 검사
    let matchingVertices = vertex.filter { return $0.data == data }
    
    if matchingVertices.count > 0 { return matchingVertices.last! }
    
    // 만일 존재하지 않는 정점이라면 새로운 정점을 만든다.
    let vertex = Vertex(data: data, index: adjacencyList.count)
    adjacencyList.append(EdgeList(vertex: vertex))
    return vertex
}
```

예제의 인접 리스트는 다음과 같이 표현될 수 있다. 

```
v1 -> [(v2: 1.0)]
v2 -> [(v3: 1.0), (v5: 3.2)]
v3 -> [(v4: 4.5)]
v4 -> [(v1: 2.8)]
```

`a -> [(b: w), ...]`의 형태는 `a`에서 `b`로 향하는 `w`의 가중치를 갖는 간선이 존재한다는 것이다. 



### The code: adjacency matrix

------

인접 행렬의 2차원 배열은 `[[Double?]]` 배열 지속해서 추적할 것이다. `nil`은 간선이 없다는 것을 의미하고 다른 값들은 주어진 가중치를 갖는 간선을 의미한다. 만일 `adjacencyMatrix[i][j]`가 `nil`이 아니라면 이는 `i`에서 `j`로 향하는 간선이 간선이 존재한다는 것이다. 

정점을 사용하여 행렬을 인덱싱하려면 그래프 객체를 통해 정점 객체를 만들 때 할당된 `Vertex`안의 `index` 프로터티를 사용한다. 새 정점을 만들 때 그래프는 만드시 행렬의 크기를 재조정해야 한다. 

```swift
open override func createVertex(_ data: T) -> Vertex<T> {
    // 이미 존재하는 정점인지 검사
    let matchingVertices = vertex.filter { return $0.data == data }
    
    if matchingVertices.count > 0 { return matchingVertices.last! }
    
    // 만일 존재하지 않는 정점이라면 새로운 정점을 만든다.
    let vertex = Vertex(data: data, index: adjacencyMatrx.count)
    
    // 각 열에 해당하는 행을 추가
    for i in 0..<adjacencyMatrix.count {
        adjacecnyMatrix[i].append(nil)
    }
    
    // 마지막 행을 추가
    let newRow = [Double?](repeating: nil, count: adjacecnyMatrix.count + 1)
    adjacencyMatrix.append(newRow)
    
    _vertices.append(vertex)
    return vertex
}
```

인접 행렬의 모습은 다음과 같을 것이다. 

```
[[nil, 1.0, nil, nil, nil]    v1
 [nil, nil, 1.0, nil, 3.2]    v2
 [nil, nil, nil, 4.5, nil]    v3
 [2.8, nil, nil, nil, nil]    v4
 [nil, nil, nil, nil, nil]]   v5

  v1   v2   v3   v4   v5
```

