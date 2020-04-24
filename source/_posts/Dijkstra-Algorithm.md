---
title: Dijkstra Algorithm
subtitle: Algorithm
date: 2019-01-28 11:16:53
tags: ["Graph", "Algorithm", "Dijkstra Algorithm", "Data Structure"]
category: ["Algorithm"]
---

> [Raywenderlich's Swift Algorithm Club](https://github.com/raywenderlich/swift-algorithm-club/tree/master/Dijkstra%20Algorithm)

### Weighted Graph general concepts

---

모든 가중치 그래프는 다음을 포함해야 한다. 

1. 간선과 정점

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Dijkstra%20Algorithm/Images/Vertices.png)

<br>

2. 간선은 정점을 연결한다. 그래프에 간선을 연결해보자. 간단하게 방향 그래프를 만들어보자. "방향"이란 간선이 정점에서 시작해서 정점에서 끝나는 방향성을 갖는다는 의미이다. 하지만 기억해야할 중요한 것이 있다. 
   1. 모든 방향성이 없는 그래프는 방향 그래프처럼 보일 수 있다. 
   2. 모든 간선이 반대 방향으로 가는 간선과 쌍으로 구성된 경우에만 방향성이 없는 그래프라고 한다. 

> 방향성이 없다는 것은 간선을 이루는 모든 두 정점에석 간선이 양방향으로 뻗어 있는 그래프를 말한다. 

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Dijkstra%20Algorithm/Images/DirectedGraph.png)

<br>

3. 모든 간선에 가중치를 매긴다. 

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Dijkstra%20Algorithm/Images/WeightedDirectedGraph.png)

<br>

4. 마침내 방향 가중치 그래프가 완성되었다. 

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Dijkstra%20Algorithm/Images/WeightedDirectedGraphFinal.png)

​	방향성이 없는 가중치 그래프는 다음과 같다.

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Dijkstra%20Algorithm/Images/WeightedUndirectedGraph.png)

<br>

**다시 한번 언급하지만 방향성이 없는 그래프란 모든 간선이 반대 방향의 간선과 짝을 이루는 방향 그래프를 말한다.**

이제 우리는 그래프의 일반적인 개념에 친숙해졌다.

<br>

### The Dijkstra's Algorithm

이 [알고리즘](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm)은 Edsger W. Dijkstra에 의해 1956년 발명되었다. 

이는 시작 정점에서부터 그래프의 모든 정점들에 대한 가장 짧은 경로를 찾아내는데 사용된다. 

가장 적절한 예가 바로 도로망이다. 집에서부터 직장까지 가장 빠른 길을 찾고 싶다면 혹은 집에서부터 가장 가까운 상점을 찾고 싶다면 다익스트라 알고리즘을 사용하면 된다. 

이 알고리즘은 그래프의 모든 정점이 방문되었다고 표시될 때 까지 다음의 사이클을 반복한다.

1. 아직 방문하지 않은 정점 중 시작점에서 거리가 가장 짧은 정점을 선택한다. (가장 짧은 거리의 정점이 하나 이상이라면 임의로 하나를 선택한다.)
2. 선택된 정점을 방문했다고 표시한다.
3. 모든 이웃한 정점을 검사한다. 출발 정점부터 이웃까지의 현재 경로길이에 현재 정점에서 이웃까지의 길이를 더한 것이 시작 정점으로부터 이웃까지의 거리보다 작다면 시적점부터 이웃점까지에 새로운 경로 길이를 할당한다. 모든 정점들이 방문되었다고 표시되면 알고리즘의 일은 끝났다. 그럼 이제 시적점에서부터 모든 정점까지 가는 최소의 경로를 알 수 있다. 

<br>

### Example

---

집에서 상점을 가는 상황을 가정해보자. 당신의 집은 A 정점이고 이곳에서부터 갈 수 있는 주변 상점은 총 4 곳이다. 가장 가까운 곳을 어떻게 찾을 수 있을까? 우리는 이를 그래프로 표현할 수 있기 때문에 그 방법을 알 수 있다. 

**Initialisation**

알고리즘의 다음의 그래프 상태에서 시작한다. 

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Dijkstra%20Algorithm/Images/image1.png)

다음은 현재 그래프 상태를 표현한 표이다. 

|                            | A    | B    | C    | D    | E    |
| -------------------------- | ---- | ---- | ---- | ---- | ---- |
| 방문 여부                  | F    | F    | F    | F    | F    |
| 시작점으로부터의 경로 길이 | inf  | inf  | inf  | inf  | inf  |
| 시작점으로부터의 정점 경로 | []   | []   | []   | []   | []   |

> `inf`는 현재는 해당 점까지 얼마나 먼지 알 수 없다는 의미이다. 시작점 역시 아직 그래프엔 반영이 되지 않은 것이라 모든 점이 `inf`인 것을 확인할 수 있다. 

시작점을 정하면 다음과 시작점 자신까지의 거리는 0, 경로는 자신을 포함한 그래프의 모습으로 초기화할 수 있다. 이제 알고리즘을 시작할 수 있다. 

|                            | A    | B    | C    | D    | E    |
| -------------------------- | ---- | ---- | ---- | ---- | ---- |
| 방문 여부                  | F    | F    | F    | F    | F    |
| 시작점으로부터의 경로 길이 | 0    | inf  | inf  | inf  | inf  |
| 시작점으로부터의 정점 경로 | [A]  | []   | []   | []   | []   |

알고리즘 사이클을 따라가보면 모든 정점을 방문하지 않았고 시작점으로부터 가장 짧은 이웃의 거리는 0이다. (시작점 자신) 우리는 이 정점의 이웃들을 살펴볼 것이다. 이 점을 방문했다고 체크하자. 

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Dijkstra%20Algorithm/Images/image2.png)

|                            | A    | B    | C    | D    | E    |
| -------------------------- | ---- | ---- | ---- | ---- | ---- |
| 방문 여부                  | T    | F    | F    | F    | F    |
| 시작점으로부터의 경로 길이 | 0    | inf  | inf  | inf  | inf  |
| 시작점으로부터의 정점 경로 | [A]  | []   | []   | []   | []   |

<br>

**Step 1**

그리고 우리는 A의 모든 이웃을 검사한다. A까지의 경로와 이웃으로 이어지는 간선의 가중치를 더한 값이 시작점에서 현재 검사중인 이웃까지의 거리보다 짧다면 시작점에서 검사중인 이웃까지의 거리를 현재 방문중인 정점에서 해당 이웃으로 이어지는 간선을 더한 값을 할당해준다. 그리고 해당 이웃까지 향하는 경로에 이웃 정점을 추가해준다. 이를 코드로 표현한다면 다음과 같다.

> 코드로 보는 것이 더욱 직관적일 것이다. 

```swift
if (A.pathLengthFromStart + AB.weight) < B.pathLengthFromStart {
    B.pathLengthFromStart = A.pathLengthFromStart + AB.weight
    B.pathVerticesFromStart = A.pathVerticesFromStart
    B.pathVerticesFromStart.append(B)
}
```

|                            | A    | B      | C    | D      | E    |
| -------------------------- | ---- | ------ | ---- | ------ | ---- |
| 방문 여부                  | T    | F      | F    | F      | F    |
| 시작점으로부터의 경로 길이 | 0    | 3      | inf  | 1      | inf  |
| 시작점으로부터의 정점 경로 | [A]  | [A, B] | []   | [A, D] | []   |

<br>

**Step 2**

우리는 이제 위와 같은 과정을 반복하여 테이블에 값들을 채워주면 된다. 

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Dijkstra%20Algorithm/Images/image4.png)

|                            | A    | B      | C    | D      | E         |
| -------------------------- | ---- | ------ | ---- | ------ | --------- |
| 방문 여부                  | T    | F      | F    | T      | F         |
| 시작점으로부터의 경로 길이 | 0    | 3      | inf  | 1      | 2         |
| 시작점으로부터의 정점 경로 | [A]  | [A, B] | []   | [A, D] | [A, D, E] |

<br>

**Step 3**

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Dijkstra%20Algorithm/Images/image5.png)

|                            | A    | B      | C            | D      | E         |
| -------------------------- | ---- | ------ | ------------ | ------ | --------- |
| 방문 여부                  | T    | F      | F            | T      | T         |
| 시작점으로부터의 경로 길이 | 0    | 3      | 11           | 1      | 2         |
| 시작점으로부터의 정점 경로 | [A]  | [A, B] | [A, D, E, C] | [A, D] | [A, D, E] |

<br>

**Step 4**

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Dijkstra%20Algorithm/Images/image6.png)

|                            | A    | B      | C         | D      | E         |
| -------------------------- | ---- | ------ | --------- | ------ | --------- |
| 방문 여부                  | T    | T      | F         | T      | T         |
| 시작점으로부터의 경로 길이 | 0    | 3      | 8         | 1      | 2         |
| 시작점으로부터의 정점 경로 | [A]  | [A, B] | [A, B, C] | [A, D] | [A, D, E] |

<br>

**Step 5**

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Dijkstra%20Algorithm/Images/image7.png)

|                            | A    | B      | C         | D      | E         |
| -------------------------- | ---- | ------ | --------- | ------ | --------- |
| 방문 여부                  | T    | T      | T         | T      | T         |
| 시작점으로부터의 경로 길이 | 0    | 3      | 8         | 1      | 2         |
| 시작점으로부터의 정점 경로 | [A]  | [A, B] | [A, B, C] | [A, D] | [A, D, E] |

<br>



### Code Implementation

---

첫 번째로, 그래프에서 정점을 나타내는 클래스를 만들어보자. 

```swift
open class Vertex {
    // 정점은 모두 고유해야 한다. 
    open var identifier: String
    
    // 그래프에서 정점은 최소 한개 이상의 이웃 정점을 갖는다. 하지만 처음에는 모든 정점을 이웃 정점 없이 초기화를 하는 경우도 있다. 그리고 다음 반복 구문에서 각 정점의 이웃들을 지정해준다. Double은 해당 Vertex로의 가중치이다. 
    open var neighbours: [(Vertex, Double)] = []
    
    // 그래프의 초기화 시기에는 시작점으로부터의 모든 정점까지의 거리는 무한의 값이다.
    open var pathLengthFromStart = Double.infinity
    
    // 이 배열은 시작점으로부터 이 정점까지의 최소거리 겅로를 지나는 정점을 담는다. 
    open var pathVerticesFromStart: [Vertex] = []
    
    public init(identifier: String) {
        self.identifier = identifier
    }
    
    // 이 메소드를 사용하여 시작점이 변경될 때마다 쉽게 이전 시작점으로부터의 값들을 모두 지워줄 수 있다. 
    open func clearCache() {
        pathLengthFromStart = Double.infinity
        pathVerticesFromStart = []
    }
}
```

모든 정점들은 고유해야하기 때문에 이들을 `Hashable`하게 만들어주고 `Equatable`을 체택하도록 한다. 우리는 이러한 목적으로 `indentifier`을 사용한다. 

```swift
extension Vertex: Hashable {
    open var hashValue: Int {
        return identifier.hashValue
    }
}

extension Vertex: Equatable {
    public static func ==(lhs: Vertex, rhs: Vertex) -> Bool {
        return lhs.hashValue == rhs.hashValue
    }
}
```

[깃헙 저장소](https://github.com/raywenderlich/swift-algorithm-club/tree/master/Dijkstra%20Algorithm)에 알고리즘의 기본 베이스 코드들은 올라와 있다. 

```swift
public class Dijkstra {
    private var totalVertices: Set<Vertex>
    
    public init(vertices: Set<Vertex>) {
        totalVertices = vertices
    }
    
    private func clearCache() {
        totalVertices.forEach { $0.clearCache() }
    }
    
    public func findShortestPahts(from startVertex: Vertex) {
        // 초기화가 선행.
        clearCache() 
        
        startVertex.pathLengthFromStart = 0
        startVertex.pathVerticesFromStart.append(startVertex)
        
        var currentVertex: Vertex? = startVertex
        
        while let vertex = currentVertex {
            // 방문한 정점은 제거.
            totalVertices.remove(vertex)
            
            // 현재 방문한 점의 이웃 중 방문하지 않은 정점을 필터링
            let filteterdNeighbours = vertex.neighbours.filter { totalVertices.contains {$0.0}}
            for neighbour in filteterdNeighbours {
                let neighbourVertex = neighbour.0
                let weight = neighbour.1
                
                let theoreticNewWeight = vertex.pathLengthFromStart + weight
                if theoreticNewWeight < neighbourVertex.pathLengthFromStart {
                    // 새 값들을 할당.
                    neighbourVertex.pathLengthFromStart = theoreticNewWeight
                    neighbourVetex.pathVerticesFromStart = vertex.pathVerticesFromStart
                    neighbourVertex.pathVerticesFromStart.append(neighbourVertex)
                }
            }
            
            // 비어있다는 것은 모든 정점을 방문했다는 의미
            if totalVertices.isEmpty {
                currentVertex = nil 
                break
            }
            
            // 아직 방문해야할 정점이 존재한다면 현재 남은 정점들 중 시작점에서부터 거리가 짧은 정점을 고른다. 
            currentVertex = totalVertices.min { 
                $0.pathLengthFromStart < $1.pathLengthFromStart
            }
        }
    }
}
```
