---
title: Custom SCNGeometry
date: 2019-03-04 21:02:10
tags: ["SCNGeometry", "SCNGeometrySource", "SCNGeometryElement"]
category: ["ARKit"]
---

오늘은 ARKit을 사용하면서 사용자 정의 `SCNGeometry`를 만드는 방법에 대해 알아보았고 이에 대해 기록을 해보려 한다. 

> ARKit의 기본적인 좌표계나 개념 혹은 SceneKit에 관한 내용은 이 포스팅에서 다루지 않는다. 

기본적으로 ARKit에선 `SCNBox`, `SCNCone`과 같이 기본적인 `SCNGeometry` 구현체를 제공한다. 이렇게 기본적으로 제공해주는 것을 사용해도 되고 다른 프로그램에서 모델링된 3D 구현체를 가져와 사용해도 된다. 하지만 Open-GL에 익숙하거나 원하는 구현체가 없다면(없을 수가 있을까…?🧐) 직접 만들어서 사용하는 방법도 있다. 

직접 `SCNGeometry`를 사용해서 구현체를 만들기 위해선 다음의 순서를 따른다. 

1. 최소 한 개 이상의 `SCNGeometrySource` 객체를 만든다. 이 객체는 **정점(vertex)**를 포함하고 있다. 또한 구현체 표면의 **법선(surface normal)** 데이터, **질감 좌표(texture coordinate)**도 제공해줄 수 있다.
2. 최소 한 개 이상의 `SCNGeometryElement` 객체를 만든다. 이 객체는 구현체 소스안의 정점들을 식별하는 인덱스 배열을 포함하고 SceneKit이 구현체를 그릴 때 정점을 연결하는데 사용하는 도면 기본 요소를 나타낸다. 
3. 위에서 만든 두 가지 객체를 사용해 `SCNGeometry` 객체를 생성한다. 

그럼 위의 순서를 따라 평면 구현체를 하나 생성해보자. 



### Implementation

---

먼저 `SCNGeometrySource` 객체를 생성해보자. 우리는 평면 구현체를 만들 것이다.

```swift
extension SCNGeometry { 
    static func plane(width: CGFloat, height: CGFloat) -> SCNGeometry { 
        let src = SCNGeometrySource(vertices: [
            SCNVector3(-width / 2, -height / 2, 0),
            SCNVector3(width / 2, -height / 2, 0),
            SCNVector3(-width / 2, height / 2, 0),
            SCNVector3(width / 2, height / 2, 0),
        ])
    }
}
```

우리를 향해 서(?)있는 평면 구현체를 만들 것이다. 너비와 높이를 인자로 받는다. 그리고 위에서 부터 좌측 아래, 우측 아래, 좌측 상단, 우측 상단에 해당하는 `SCNVector3` 객체이다. 

그리고 위의 세 단계 중 첫 번째 단계에서 언급한 법선 데이터와 질감 좌표도 추가해보자. 

```swift
extension SCNGeometry { 
    static func plane(width: CGFloat, height: CGFloat) -> SCNGeometry { 
        let src = SCNGeometrySource(vertices: [
            SCNVector3(-width / 2, -height / 2, 0),
            SCNVector3(width / 2, -height / 2, 0),
            SCNVector3(-width / 2, height / 2, 0),
            SCNVector3(width / 2, height / 2, 0),
        ])
        
        let normals = SCNGeometrySource(normals: [SCNVector3](repeating: SCNVector3(0, 0, 1), count: 4) )
        let textureMap = SCNGeometrySource(textureCoordinates: [
            CGPoint(x: 0, y: 1),
            CGPoint(x: 1, y: 1),
            CGPoint(x: 0, y: 0),
            CGPoint(x: 1, y: 0)
        ])
    }
}
```

각각의 생성자를 애플의 공식문서에서는 다음과 같이 설명하고 있다. 

- `SCNGeometry.init(normals:)`, `SCNGeometry.init(textureCoordinates:)` - SceneKit은 랜더링 성능을 최적화하기 위해 이 두 생성자를 통해 넘어오는 데이터를 자신만의 포맷으로 전환한다. 

  > 단순히 정점 데이터만을 제공하는 것이 아닌 이렇게 법선, 질감 좌표를 함께 제공하면 하나의 구현체를 만드는데 더 많은 정보를 제공하는 것이기 때문에 그 만큼 랜더링 성능이 좋아지는 것이 아닐까 하는 개인적인 생각이 든다 (틀렸다면 피드백 부탁드립니다.)

- 질감 좌표는 기본적으로 0에서 1 사이의 범위 값으로 지정해준다. Open-GL의 텍스처에 관한 [**글**](https://learnopengl.com/Getting-started/Textures)을 참고해보면 이해가 될 것이다.  



그럼 이제 `SCNGeometryElement`와 함께 평면 구현체를 완성해보자. 

```swift
extension SCNGeometry {
	static func Plane(width: CGFloat, height: CGFloat) -> SCNGeometry {
        let src = SCNGeometrySource(vertices: [
            SCNVector3(-width / 2, -height / 2, 0),
            SCNVector3(width / 2, -height / 2, 0),
            SCNVector3(-width / 2, height / 2, 0),
            SCNVector3(width / 2, height / 2, 0),
        ])

        let normals = SCNGeometrySource(normals: [SCNVector3](repeating: SCNVector3(0, 0, 1), count: 4) )
        let textureMap = SCNGeometrySource(textureCoordinates: [
            CGPoint(x: 0, y: 1),
            CGPoint(x: 1, y: 1),
            CGPoint(x: 0, y: 0),
            CGPoint(x: 1, y: 0)
        ])

        let indices: [UInt32] = [0, 1, 2, 3]
        let inds = SCNGeometryElement(indices: indices, primitiveType: .triangleStrip)
        return SCNGeometry.init(sources: [src, normals, textureMap], elements: [inds])
	}
}

```

- 네 개의 정점(`SCNVector3`)를 제공해주었기 때문에 그에 대응하는 4개의 인덱스를 제공해준다. 
- `init(indices:primitiveType:)` 생성자를 통해 `SCNGeometryElement` 객체를 생성해준다. 
  - `indices`는 구현체의 정점을 식별한다. 
  - `primitiveType`은 [`SCNGeometryPrimitiveType`](https://developer.apple.com/documentation/scenekit/scngeometryprimitivetype) 타입으로 SceneKit이 구현체의 정점 데이터를 어떻게 해석해야하는지에 대한 정보를 담은 값이다. 
    - `triangles` - 구현체 요소 정보가 연속된 삼각형의 형태로 각각의 삼각형은 새로운 세 개의 정점으로 이루어져 있다.
    - `triangleStrip` - 구현체 요소 정보가 연속된 삼각형 형태로 각각의 삼각형은 새로운 한 정점과 이전 삼각형의 두 정점으로 이루어져 있다. 즉 사각형(rectangle)의 모양을 이룰 수 있다. 
    - `line` - 구현체 요소 정보가 선 형태의 연속으로 각각의 선은 두 개의 새로운 정점 데이터로 이루어져 있다.
    - `point` - 구현체 요소 정보가 서로 연결되지 않은 점들의 연속된 형태이다. 



즉 `SCNGeometryElement`가 제공하는 정보는 

***"네 개의 정점 데이터는 연속된 삼각형의 모습인데 각각의 삼각형은 새로운 점 하나와 이전 삼각형의 점 두 개로 이루어져 있다"*** 

를 의미한다. 즉 평면을 이룰 수 있다는 의미이다. 



이렇게 만든 구현체를 다음의 코드를 통해 화면 상에 띄어줄 수 있다. 

```swift
let newNode = SCNNode(geometry: SCNGeometry.Plane(width: 0.2, height: 0.2))
newNode.position.z = -1
sceneView.scene.rootNode.addChildNode(newNode)
```



---

**참고자료**

- [ARKit + SceneKit Geometries Tutorial](https://medium.com/@maxxfrazer/scenekit-geometry-part1-e5dca2156d8)