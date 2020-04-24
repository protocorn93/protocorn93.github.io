---
title: Heap
subtitle: Data Structure
date: 2019-03-01 12:13:31
tags: ["Swift", "Heap", "Data Structure"]
category: ["Data Structure"]
---

> [Raywenderlich's Swift Algorithm Club](https://github.com/raywenderlich/swift-algorithm-club/tree/master/Heap)

**힙(heap)** 배열 안의 **[이진 트리(Binary tree)](https://github.com/raywenderlich/swift-algorithm-club/blob/master/Binary%20Tree)**로 부모/자식 포인터를 사용하지 않는다. 힙은 **"힙 속성(Heap property)"**을 기반으로 정렬되어 있으며 이는 트리 안 노드의 순서를 결정한다. 

힙이 사용되는 예

- [우선순위 큐](https://github.com/raywenderlich/swift-algorithm-club/blob/master/Priority%20Queue)
- [힙 정렬](https://github.com/raywenderlich/swift-algorithm-club/blob/master/Heap%20Sort)
- 콜렉션의 최대/최소 요소를 계산



### The heap property 

---

힙에는 **최대 힙**(*max-heap*), **최소 힙**(*min-heap*) 이 두 가지 종류가 존재한다. 최소 힙에서 모든 부모 노드는 자식 노드보다 작은 값을 갖는다. 이를 힙 속성, Heap property라 부른다.



![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Heap/Images/Heap1.png)

위의 예제는 최대 힙으로 모든 부모 노드는 자식 노드보다 큰 값을 갖는다. `10`은 `7`과 `2`보다 크며 `7`은 `5`와 `1`보다 크다.

힙 속성의 결과로 최대 힙은 항상 가장 큰 값이 루트 노드에 위치하게 된다. 최소 힙은 항상 가장 작은 값이 루트 노드에 위치하게 된다. 힙은 우선순위 큐에서 가장 *중요한 값*에 빨리 접근하는데 사용되기 때문에 힙 속성은 매우 유용하다. 

> **Note** : 힙의 루트 노드 값은 항상 최대 값이거나 최소 값이지만 다른 요소들의 정렬은 예측할 수 없다. 예를 들어 최대 힙에서 최대 값의 0번째 인덱스에 위치하지만 최소 값은 마지막 인덱스에 위치한다는 것은 보장할 수 없다. 



### How does a heap compare to regular trees?

---

힙은 이진 탐색 트리의 대체재가 아니다. 그들 사이에는 비슷한 점과 차이점이 존재한다. 주요 차이점은 다음과 같다. 

**노드의 순서**. [이진 탐색 트리](https://github.com/raywenderlich/swift-algorithm-club/blob/master/Binary%20Search%20Tree)에서 왼쪽 노드 자식은 반드시 부모 노드보다 작아야 하고, 오른쪽 노드 자식은 반드시 부모 노드보다 커야 한다. 이는 힙에 해당하지 않는다. 최대 힙에서 양쪽 자식 모두 부모 노드보다 작아야 한고 최소 힙에선 모두 부모 노드보다 커야 한다. 

**메모리**. 전통적인 트리는 저장해야 할 데이터보다 많은 메모리를 차지한다. 노드 객체와 왼쪽, 오른쪽 자식을 향하는 포인터를 위한 추가적인 저장 공간을 할당받아야 한다. 반면 힙은 오로지 순수한 배열만을 사용하며 포인터를 사용하지 않는다. 

**균형**. 이진 탐색 트리는 반드시 **균형** 잡혀야 한다. 그래서 모든 오퍼레이션의 시간 복잡도는 **O(logN)** 성능을 보인다. 임의의 순서 혹은 [AVL tree](https://github.com/raywenderlich/swift-algorithm-club/blob/master/AVL%20Tree)나 [red-black tree](https://github.com/raywenderlich/swift-algorithm-club/blob/master/Red-Black%20Tree)를 사용하여 데이터의 삽입, 삭제를 할 수 있다. 하지만 힙에선 전체적으로 정렬된 트리를 가질 필요가 없다. 단순히 힙 속성만을 필요로 하며 균형은 이슈가 되지 않는다. 힙을 이루고 있는 구조 때문에 힙은 **O(logN)**의 성능을 보장한다. 

**탐색**. 이진 트리에서 탐색이 탐색이 빠른 반면 힙에서의 탐색은 느리다. 힙의 목적은 가장 크거나 작은 값을  노드의 가장 앞에 위치시키는 것과 빠른 삽입, 삭제이기 때문에 탐색은 힙에서 우선순위가 높지 않다. 



### The tree inside an array 

---

트리와 같은 구조를 구현하기에 배열을 사용하는 것이 이상해보일 수 있다. 하지만 시간과 공간면에서 이는 효율적이다. 위 예제의 트리를 저장하는 방법은 다음과 같다. 

```
[10, 7, 2, 5, 1]
```

이게 전부다! 이 간단한 배열 외의 추가적인 저장소는 필요하지 않다. 

포인터를 사용하지 않는다면 자식 노드와 부모 노드를 어떻게 구분할까? 좋은 질문이다. 트리 노드의 배열 인덱스와 자신의 부모 그리고 자식의 배열 인덱스 사이에는 잘 정의된 관계가 존재한다. 

만일 `i`가 노드의 인덱스라면 다음의 공식을 통해 우리는 이 노드의 자식 노드와 부모 노드를 알 수 있다. 

```
parent(i) = floor((i-1)/2)
left(i) = 2*i + 1
right(i) = 2*i + 2
```

`right(i)`는 단순하게 생각하면 `left(i) + 1`이다. 왼쪽과 오른쪽 노드는 항상 서로 이웃하여 저장된다. 이 공식을 우리의 예제에 사용해보자. 이를 통해 우리는 다음 표와 같은 결과를 얻을 수 있다. 

![](https://ehdrjsdlzzzz.github.io/2019/03/01/Heap/fomula.png)

> **Note** : 루트 노드 (10)는 부모 노드를 갖지 않는다. 배열에서 -1은 유효하지 않은 인덱스이기 때문이다. 마찬가지로 (2), (5) 그리고 (1)의 자식 노드 인덱스는 범위를 초과하기 때문에 자식 노드를 갖고 있지 않음을 나타낸다. 그렇기 때문에 인덱스를 사용하기 전에 우리는 반드시 그들이 유효한 범위 내에 존재하는 인덱스인지를 계산해야 한다. 

최대 힙을 다시 생각해보면, 부모 노드의 값은 항상 자식 노드의 값보다 크다(크거나 같다). 이 뜻은 배열의 모든 인덱스 `i`에 대해 다음을 만족해야 한다는 것이다. 

```swift
array[parent(i)] >= array[i]
```

봤다시피 우리는 이러한 공식들을 통해 포인터 없이 부모와 자식 노드들을 찾을 수 있다. 단순히 포인터를 사용하는 것보다 복잡하지만 메모리 공간을 절약할 수 있다. 다행히도 계산은 오직 **O(1)** 시간밖에 걸리지 않는다. 

트리에서의 위치와 배열 인덱스 사이의 관계를 이해하는 것은 중요하다. 다음은 네 개의 레벨로 나누어진 보다 큰 힙이다. 

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Heap/Images/LargeHeap.png)

그림에 나와있는 숫자는 노드의 값이 아닌 노드를 저장하고 있는 배열의 인덱스 값이다. 다음은 트리의 다른 레벨들에 해당하는 배열의 인덱스 값이다. 

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Heap/Images/Array.png)

공식에 따르면 부모 노드는 자식 노드 이전에 위치해야 한다. 위의 그림을 통해 이를 확인할 수 있다. 이 계획은 한계가 있다는 것을 명심하자. 이진 트리로 다음 그림의 트리를 표현할 수 있지만 힙으로는 불가능하다. 

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Heap/Images/RegularTree.png)

현재 레벨이 완벽하게 채워져있지 않으면 다음 레벨을 진행할 수 없다. 그러므로 힙은 항상 다음과 같은 모습을 유지해야 한다. 

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Heap/Images/HeapShape.png)

> 물론 가상의 노드를 통해 힙으로 완벽하게 채워지지 않은 레벨이 존재하는 트리를 만들 수 있지만 이는 추가적인 불필요한 공간을 필요로 하며 비어있다는 것을 마킹해야 한다. 

다음과 같은 배열이 있다고 가정하자.

```
[10, 14, 25, 33, 81, 82, 99]
```

유효한 힙일까? 그렇다 유효한 힙이다. 오름차순의 정렬된 배열은 유효한 최소 힙이다. 우리는 이를 다음과 같이 표현할 수 있다. 

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Heap/Images/SortedArray.png)

**오름차순**으로 **정렬**되어 있는 배열이기 때문에 각 노드는 힙의 속성을 만족하고 있다. 

> **Note** : 하지만 모든 최소 힙이 반드시 정렬된 배열일 필요는 없다. 힙을 다시 정렬된 배열로 돌리기 위해선 [힙 정렬](https://github.com/raywenderlich/swift-algorithm-club/blob/master/Heap%20Sort)을 사용해야 한다. 



### More math!

---

힙 속성에 관한 몇 가지 공식이 더 존재한다. 이들을 가슴 깊이 새길 필요는 없지만 종종 편리함을 제공한다. 이 섹션은 생략해도 무방하다.

트리의 **높이**는 루트 노드에서 말단 노드까지 가는데 걸리는 단계 수로 정의된다. 이를 좀 더 공식으로 표현하자면 높이는 노드들 사이의 최대 간선 수이다. 힙의 높이가 *h*라면 힙은 *h + 1* 레벨을 갖는다. 

3의 높이를 갖는 힙은 4개의 레벨을 갖는다. 

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Heap/Images/LargeHeap.png)

n 개의 노드를 갖는 힙의 높이 공식은 `h = floor(log2(n))`이다. 왜냐하면 가장 낮은 레벨의 노드는 항상 모두 채워진 후 다음 레벨을 채우기 시작하기 때문이다. 위의 예제는 15개의 노드를 갖고 있고 높이는 `floor(log2(15)) = floor(3.91) = 3`이다.

가장 낮은 레벨이 모두 채워져 있다면 그 레벨은 2^h 개의 노드를 갖는다. 그리고 그 밑의 레벨의 노드들은 모두 2^h - 1개의 노드를 갖는다. 위의 그림을 예로 들자면 레벨 3의 노드의 갯수는 2^3으로 8개이고 그 밑 레벨인 레벨 0에서 레벨 2까지의 노드들의 총합은 7개로 2^3 - 1공식을 만족한다. 

그러므로 힙 전체의 노드 갯수의 총합은 *2^(h+1) - 1* 공식을 갖는다. 위의 예제에선 `2^4 - 1 = 16 - 1 = 15`이다.

말단 노드는 항상 *floor(n/2)*에서 *n-1*의 배열 인덱스에 위치한다. 우리는 이러한 사실들을 이용해 빠르게 배열에서부터 힙을 만들어 낼 것이다. 



### What can you do with a heap?

---

요소를 제거하거나 추가한 후 최대 힙 혹은 최소 힙을 유지하기 위한 두 가지 중요한 연산이 존재한다. 

- `shiftUp()` : 만일 요소가 자신의 부모보다 크거나(최대 힙) 작다면(최소 힙), 이 요소는 부모와 위치를 바꿔야 한다. 이는 요소를 트리를 올라가게 한다. 
- `shiftDown()` : 만일 요소가 자신의 자식보다 작거나(최대 힙) 크다면(최소 힙), 이 요소는 트리를 내려갈 필요가 있다. 이 오퍼레이션을 "heapify"라 부르기도 한다. 

두 연산은 **O(logN)**의 시간이 소요되는  재귀적인 방법이다.

다음은 위의 두 주요한 연산 위에 만들어진 연산들이다. 

- `insert(value)` : 새 요소를 힙 끝에 추가고 `shiftUp()` 연산을 통해 힙의 형태를 유지한다. 
- `remove()` : 최댓값(최대 힙) 혹은 최솟값(최소 힙)을 제거하고 반환한다. 이로 인해 생긴 빈 공간을 가장 마지막 요소로 채운 후 `shiftDown()` 연산을 통해 제자리를 찾아가게 하여 힙의 형태를 유지한다. (이를 최솟값 추출 혹은 최댓값 추출이라 부르기도 한다.)
- `removeAtIndex(index)` : `remove()` 연산과 비슷하지만 루트 노드만이 아닌 다른 곳에 위치한 요소를 제거, 반환하는 연산이다. 이는 `shiftDown()`과 `shiftUp()` 모두를 호출하는데 새로운 요소가 자식들과의 순서가 맞지 않으면 `shiftDown()`을 부모와의 순서가 맞지 않으면 `shiftUp()`을 호출한다. 
- `replace(index, value)` : 노드에 더 작은 값(최소 힙) 혹은 더 큰 값(최대 힙)을 할당한다. 이는 힙 속성을 무효화할 수 있기 때문에 `shiftUp()` 연산을 통해 힙의 형태를 유지한다. 

위에 나열된 모든 연산 `shiftUp()`과 `shiftDown()`이 비용이 비싸기 때문에 **O(logN)**의 시간 복잡도를 갖는다. 좀 더 시간이 오래 걸리는 연산도 존재한다. 

- `search(value)` : 힙은 탐색에 효율적으로 만들어지지 않았지만 `replace()`와 `removeAtIndex()` 연산은 노드의 배열 인덱스 값을 알아야 하기 때문에 이를 찾는데 걸리는 시간은 **O(N)**이다. 
- `buildHeap(array)` : 정렬 상태가 아닌 배열을 힙으로 전환시키기 위해선 반복적으로 `insert()`를 호출해야 한다. 이는 **O(N)**의 시간 복잡도를 갖는다. 
- [힙 정렬](https://github.com/raywenderlich/swift-algorithm-club/blob/master/Heap%20Sort). 힙은 배열이기 때문에, 우리는 힙의 고유한 속성을 이용해 배열을 오름차순으로 정렬할 수 있다. 이는 **O(NlogN)**의 시간 복잡도를 갖는다. 

힙은 또한 가장 큰 값(최대 힙) 혹은 가장 작은 값(최소 힙)을 제거하지 않고 단순히 반환만 하는 `peek()`이라는 연산도 갖고 있는데 이는 **O(1)**의 시간 복잡도를 갖는다. 

> **Note** : 지금까지 가장 흔히 사용하는 연산은 새로운 요소를 추가하는 `insert()` 연산과 가장 큰 값 혹은 작은 값을 반환 후 제거하는 `remove()` 연산이다. 이 둘 모두 **O(logN)**의 시간 복잡도를 갖는다. 다른 연산들은 좀 더 수준 높은 사용을 지원하기 위해 존재한다. 새 요소의 중요성이 큐에 추가된 후 변경되는 우선순위 큐를 만드는 것이 그 예이다. 



### Inserting into the heap

---

먼저 힙에 값을 삽입하는 것부터 자세히 살펴보자. 우리는 `16`을 아래의 힙에 추가할 것이다.

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Heap/Images/Heap1.png)

이 힙의 배열은 `[10, 7, 2, 5, 1]`이다.

값의 삽입의 첫 번째 단계는 새 요소를 배열의 끝에 추가하는 것이고 배열은 다음과 같은 모습을 할 것이다. 

```
[10, 7, 2, 5, 1, 16]
```

그리고 이에 따른 트리의 모습은 다음과 같을 것이다. 

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Heap/Images/Insert1.png)

`16`은 가장 마지막 행의 첫 번째 유효한 공간에 위치하게 된다. (부모 노드 기준 좌측 자식 노드 위치)

불행하게도 위의 힙은 더 이상 힙의 형태가 아니다. 최대 힙임에도 불구하고 부모 노드인 `2`가 자식 노드인 `16`보다 작기 때문이다. 힙의 속성을 유지하기 위해 `2`와 `16`의 위치를 바꿔야 한다. 

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Heap/Images/Insert2.png)

이 둘을 바꿨지만 아직 유효한 힙의 형태는 아니다. 여전히 `10`이 `16`보다 작기 때문에 한 번 더 교환 작업을 진행해야 한다. 이렇게 자리를 바꾸는 쉬프트 행위를 부모 노드가 새 노드보다 크거나 트리의 꼭대기(루트)에 도달할 때까지 진행해야 한다. 이러한 작업을 **shift-up** 혹은 **shifting**이라 부른고 이들은 모든 삽입 이후에 행해진다. 

삽입 후 최종 힙의 모습은 다음과 같아야 한다. 

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Heap/Images/Insert3.png)

이제 모든 부모 노드는 자식 노드보다 큰 값을 갖게 된다. 

쉬프팅 업에 요구되는 시간은 트리의 높이에 비례하고 그렇기 때문에 이는 **O(logN)**의 시간 복잡도를 갖는다. (단순히 노드를 추가하는 것은 배열의 끝에 추가하는 것이기 때문에 **O(1)**의 시간 복잡도를 갖는다. 그러므로 이는 삽입을 느리게 만들지 않는다.)



### Removing the root 

---

다음의 트리에서 `10`을 제거해보자.

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Heap/Images/Heap1.png)

가장 위의 빈 공간에는 무슨 일이 벌어질까?

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Heap/Images/Remove1.png)

새로운 값을 삽입할 때는 새 요소를 배열의 가장 끝에 추가를 했다. 하지만 여기서 우리는 이와 반대로 행동한다. 우리가 갖고 있는 가장 마지막 값을 트리의 가장 위(루트)에 위치시킨다. 그리고 힙의 형태를 정렬해나간다. 

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Heap/Images/Remove2.png)

`1`을 **shift-down** 하는 과정을 살펴보자. 최대 힙의 형태를 유지하기 위해 우리는 가장 큰 값이 맨 위에 올라가길 원한다. 우리는 `1`과 스와핑 될 두 후보(`7`과 `2`)를 알고 있다. 우리는 이 세 노드 중 맨 위로 갈 노드를 선택해야 한다. 그것은 `7`이 될 것이고 `1`과 `7`을 스와핑하면 다음의 트리 모습을 확인할 수 있다.

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Heap/Images/Remove3.png)

더 이상 자식 노드가 없거나 모든 자식 노드보다 클 때 까지 이 연산을 반복한다. 우리의 예제 힙에선 한 번의 연산만 더 진행하면 최대 힙의 형태를 유지할 수 있다. 

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Heap/Images/Remove4.png)

이렇게 쉬프팅하여 내려가는 모든 연산 역시 트리의 높이에 비례하고 이는 **O(logN)**의 시간 복잡도를 갖는다.

> **Note** : `shiftUp()`과 `shiftDown()`은 한 번에 한 요소만 순서를 수정할 수 있다. 만일 올바르지 않은 위치에 존재하는 요소가 다수 존재한다면 각 요소들에 대해 이 연산들을 호출해주어야 한다. 



### Removing any node 

---

대부분의 경우 힙의 루트를 제거한다. 그것이 힙이 생겨나게 된 이유이기 때문이다. 

하지만 임의의 요소를 제거하는 것도 유용할 수 있다. 이는 일반적인 `remove()`이고 `shiftDown()`과 `shiftUp()` 연산을 포함할 수 있다. 

예제 트리에서 `7`을 제거해보자. 

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Heap/Images/Heap1.png)

이 트리의 배열의 형태는 다음과 같다. 

```
[10, 7, 2, 5, 1]
```

요소를 제거한다는 것은 잠재적으로 최대 힙 혹은 최소 힙의 형태를 무효화할 수 있다. 이를 수정하기 위해 제거하려는 노드를 가장 마지막 요소와 스와핑한다. 

```
[10, 1, 2, 5, 7]
```

이제 마지막 요소가 우리가 반환할 요소가 된다. 우리는 `removeLast()` 연산을 통해 힙에서 마지막 값을 제거, 반환할 수 있다. 이제 `1`이 순서를 벗어난 요소가 된 것이다. 이제 우리는 이 요소를 `shiftDown()` 연산을 통해 제자리로 찾아가게 할 것이다. 

하지만 여기서 이 연산만을 필요로 하는 것은 아니다. 새 요소가 위로 올라갈 수도 있기 때문에 `shiftUp()` 연산이 사용될 수도 있다. 만일 다음 힙에서 `5`를 제거한다면 어떤 일이 벌어질까?

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Heap/Images/Remove5.png)

`5`는 `8`과 스와핑될 것이고 `8`은 부모 노드보다 크기 때문에 `shiftUp()` 연산을 통해 제자리를 찾아가야 한다. 



### Creating a heap from an array

---

배열을 힙으로 전환하는 것은 수월할 수 있다. 단순히 배열 요소를 힙의 형태를 만족할 때 까지 섞는 것인데 코드는 다음과 같다. 

```swift
private mutating func buildHeap(from array: [T]) { 
    for value in array { 
    	insert(value)
    }
}
```

단순히 배열의 각 요소들에 대해 `insert()` 연산을 호출한 것이다. 단순하지만 효율적이지 못하다. **n** 개의 요소에 대해 **logN**이 소요되는 삽입 연산을 진행하기 때문에 총 **NlogN**의 시간이 소요된다. 

위의 수학에 관한 세션을 생략하지 않았다면 배열 인덱스 중 `n/2` 에서 `n-1`까지의 힙 요소들은 말단 노드에 위치한다는 것을 알 수 있다. 우리는 해당 노드들은 생략할 수 있다. 우리는 그 외의 노드들에 대해서만 작업을 진행해주면 된다. 왜냐하면 말단 노드를 제외한 노드들은 하나 이상의 자식을 가진 부모 노드들이기 때문에 잘못된 순서를 가질 수 있기 때문이다.

```swift
private mutating func buildHeap(fromArray array: [T]) {
    elements = array
    for i in stride(from: (nodes.count/2-1), through: 0, by: -1) {
        shiftDown(index: i, heapSize: elements.count)
    }
}
```

`elements`는 힙의 고유한 배열이다. 우리는 말단이 아닌 첫 번째 노드에서 시작한다. 그리고 `shiftDown()`을 호출하여 노드들과 우리가 생략한 말단 노드들을 올바른 순서로 배치한다. 이는 Floyd 알고리즘으로 알려져 있고 **O(n)** 시간 복잡도를 갖는다.



### Searching the heap

---

힙은 빠른 탐색을 위해 고안된 것이 아니다. 하지만 만일 `removeAtIndex()` 혹은 `replace()`를 통해 임의의 요소를 제거하거나 변경하고 싶다면 해당 요소의 인덱스를 알아야 한다. 탐색은 이를 가능하게 하지만 느릴 수 있다.

이진 탐색 트리에서 노드의 정렬 순서에 따라 빠른 탐색은 보장될 수 있다. 반면 힙에서 노드의 정렬은 이와 다르기 때문에 이진 탐색 트리는 맞지 않고 트리의 모든 노드를 탐색해야 한다. 

우리의 예제 힙을 다시 사용해보자. 

![](https://github.com/raywenderlich/swift-algorithm-club/raw/master/Heap/Images/Heap1.png)

`1`의 값을 갖는 노드의 인덱스를 알고 싶다면 `[10, 7, 2, 5, 1]`  배열을 선형 탐색을 통해 이를 알아낼 수 있다. 

힙의 속성이 탐색을 의도하진 않았지만 우리는 이 힙 속성을 탐색의 이점으로 활용할 수 있다. 우리는 최대 힙에서 부모 노드의 값이 항상 자식 노드의 값 보다 크다는 사실을 알고 있고 만일 우리가 찾으려는 값보다 부모 노드가 작다면 그 자식 노드들은 살펴볼 필요가 없다. 

우리가 존재하지 않는 `8`을 탐색한다고 가정하자. 우리는 루트 노드(`10`)부터 시작할 것이다. 이는 우리가 원하는 값이 아니므로 양쪽 자식들을 살펴볼 것이다. 왼쪽 자식 `7` 역시 우리가 원하는 값이 아니지만 이는 최대 힙이므로 `7` 밑으로는 볼 필요가 없다는 사실을 알고 있다. `2`도 마찬가지이고 그러므로 `8`은 존재하지 않는 값이 된다. 

이 사소한 최적화에도 불구하고 탐색은 여전히 **O(N)**의 시간 복잡도를 갖는다.

> **Note** : 노드 값을 인덱스에 매핑하는 딕셔너리를 사용함으로써 **O(1)**의 시간 복잡도를 갖는 방법도 존재한다. 만약 힙으로 만들어진 우선순위 큐에서 `replace()` 연산을 통해 객체들의 우선순위를 변경해주는 작업이 빈번하다면 이를 사용하는 것도 효과적일 수 있다. 



### The code 

[Heap.swift](https://github.com/raywenderlich/swift-algorithm-club/blob/master/Heap/Heap.swift) 코드를 통해 이러한 컨셉을 스위프트로 구현한 코드를 확인할 수 있다. 대부분의 코드는 직관적이다. `shiftUp()`과 `shiftDown()`이 약간 헷갈릴 수 있다. 

최대 힙과 최소 힙이 존재한다는 것은 이미 알고 있다. 이 둘의 차이점은 오직 그들이 노드를 어떻게 정렬하냐이다. 가장 큰 값이 먼저 올지, 가장 작은 값이 먼저 올지

`MaxHeap`과 `MinHeap`, 두 버전 모두 만드는 것보다 하나의 힙을 만들고 이 힙은 `isOrderedBefore` 클로저를 받도록 만들었다. 이 클로저는 두 값의 순서를 결정하는 로직을 포함하고 있다. 이는 스위프트에서 `sort()`를 사용할 때 본 적이 있을 것이다. 

최대 힙을 만들기 위해선 다음과 같이 작성하면 되고

```swift
var maxHeap = Heap<Int>(sort: >)
```

최소 힙을 만들기 위해선 다음과 같이 작성하면 된다.

```swift
var minHeap = Heap<Int>(sort: <)
```

