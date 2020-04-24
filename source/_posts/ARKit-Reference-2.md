---
title: ARKit (2)
subtitle: Detecting Plane and Placing Object
date: 2018-10-28 21:24:14
tags: ["iOS","ARKit","SceneKit"]
category: ["iOS"]
---

이번 포스팅에서 ARKit에서 중요한 기능 중 하나인 **면 인식 (Plane Detection)**에 대해 알아보고 이를 코드로 구현해보려 한다. 바로 코드에 들어가기 앞서 먼저 [`ARSessionDelegate`](https://developer.apple.com/documentation/arkit/arsessiondelegate)와 [`ARSCNViewDelegate`](https://developer.apple.com/documentation/arkit/arscnviewdelegate) 이 두 프로토콜에 대해 먼저 알아보고자 한다.

#### ARSessionDelegate

세션이 캡쳐한 `ARFrame` 객체를 직접 다룰 일이 필요하거나 세션이 트래킹하고 있는 `ARAnchor`들의 변화들을 직접 다뤄주기 위해선 이 프로토콜을 채택해야 한다. 일반적으로 사용자 정의 뷰를 통해 AR 컨텐츠를 보여주고자 할 때 이 프로토콜을 사용한다. 만일 SceneKit이나 SpriteKit을 사용해 컨텐츠를 보여준다면 [`ARSCNViewDelegate`](https://developer.apple.com/documentation/arkit/arscnviewdelegate)나 [`ARSKViewDelegate`](https://developer.apple.com/documentation/arkit/arskviewdelegate) 프로토콜도 유사한 기능을 제공하고 SceneKit과 SpriteKit과 통합된 기술을 제공하기 때문에 이 두 프로토콜을 사용한다면 더욱 편리하게 AR 컨텐츠를 표시할 수 있다. 

> 이 포스팅에선 `ARSCNView`를 사용하여 컨텐츠를 보여주기 때문에 `ARSCNViewDelegate` 프로토콜을 사용할 것이다. 

#### ARSCNViewDelegate 

이 프로토콜을 구현하여 뷰의 AR 세션에 의해 추적 된 [`ARAnchor`](https://developer.apple.com/documentation/arkit/aranchor) 객체에 SceneKit 컨텐츠를 제공하거나 해당 컨텐츠의 갱신을 관리할 수 있다. 

이 두 프로토콜 모두 [`ARSessionObserver`](https://developer.apple.com/documentation/arkit/arsessionobserver) 프로토콜의 확장 형태로 세션 상태 변화에 대응할 수 있는 메소드를 구현할 수 있다. 

- [`func session(ARSession, cameraDidChangeTrackingState: ARCamera)`](https://developer.apple.com/documentation/arkit/arsessionobserver/2887450-session)
- [`func sessionWasInterrupted(ARSession)`](https://developer.apple.com/documentation/arkit/arsessionobserver/2891620-sessionwasinterrupted)
- [`func session(ARSession, didFailWithError: Error)`](https://developer.apple.com/documentation/arkit/arsessionobserver/2887453-session)



그럼 이제 본격적으로 코드를 통해 오늘의 주제를 살펴보도록 하자. 

> 전체 코드는 [이곳](https://github.com/ehdrjsdlzzzz/ARKit-example/tree/master/Plane%20Detection%20In%20Depth)을 참고

### Plane Detection

가장 먼저 면 감지를 위한 세션을 구성해보자.

```swift
override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)

    let configuration = ARWorldTrackingConfiguration()
    configuration.planeDetection = [.horizontal]
    self.sceneView.session.run(configuration)
}
```

이번 예제에선 수평의 면만 인식하도록 `.planeDetection`에는 `.horizontal`만 할당하도록 하자. 이렇게 매우 적은 양의 코드로 면 감지를 수행할 수 있다. 

그럼 이렇게 감지된 면에 오브젝트를 띄어보자. 감지된 면 위를 터치하면 해당 위치에 오브젝트를 위치시키도록 할 것이다. 이를 위해선 터치가 일어난 화면 부분에 대한 실제 세상의 좌표를 얻어와야 하고 이를 위해선 **Hit Testing**을 진행해야 한다. 



### Hit Testing and Placing Object

Hit Testing의 결과로 반환되는 교차점 위에 오브젝트를 위치시켜보자.

```swift
// --- 1
self.sceneView.addGestureRecognizer(UITapGestureRecognizer(target: self, action: #selector(handleTapGesture)))

// --- 2
@objc func handleTapGesture(_ gesture: UITapGestureRecognizer) {
    guard let sceneView = gesture.view as? ARSCNView else { return }
    let location = gesture.location(in: sceneView)
    // --- 3
    let hitResult = sceneView.hitTest(location, types: [.existingPlane,.existingPlaneUsingExtent])
    guard let result = hitResult.first else { return }

    guard let treeScene = SCNScene(named: "art.scnassets/box.scn") else { return }
    guard let node = treeScene.rootNode.childNode(withName: "box", recursively: false) else { return }
    // --- 4 
    node.position = SCNVector3(result.worldTransform.columns.3.x, result.worldTransform.columns.3.y, result.worldTransform.columns.3.z)
    sceneView.scene.rootNode.addChildNode(node)
}
```

1. 뷰에 제스처를 등록하는 여타 다른 과정과 동일하게 `UITapGestureRecognizer`를 등록해준다. 
2. 액션을 정의해준다. 
3. 제스처가 발생한 `sceneView` 상의 위치를 활용해 `hitTest(_:, types:)`를 진행해본다.
   1. 예제에선 `.existingPlane`과 `.existingPlaneUsingExtent` 타입을 할당해주었는데 이러한 유형에 해당하는 Hit Testing의 결과를 반환해준다. 이 둘의 차이점은 밑에서 살펴볼 것이다. 
4. Hit Testing의 결과는 화면 상에서 발생한 터치에 해당하는 실제 세상의 좌표(`worldTransform`)를 포함하고 있고 오브젝트의 `position`에 이 좌표를 할당해주어 터치가 발생한 위치에 오브젝트를 위치시킬 수 있다. 

> `worldTransform`의 `colums`는 무엇이고  `.3.x`는 무엇을 의미할까? 이에 대해서도 추후에 다룰 예정이다. 일단은 사용해보자!

면을 인식하고 원하는 위치에 오브젝트를 위치시키는 행위 모두 그리 길지 않은 코드로 쉽게(?) 구현할 수 있다. 

#### Hit Testing Type

우리는 Hit Testing을 진행할 때 테스트 유형을 지정해줄 수 있고 이렇게 지정되는 유형에 따라 테스트 결과는 달라진다. 이 테스트 유형은 [`ARHitTestResult.ResultType`](https://developer.apple.com/documentation/arkit/arhittestresult/resulttype) 타입으로 우리는 이들 중 [`.existingPlane`](https://developer.apple.com/documentation/arkit/arhittestresult/resulttype/2875738-existingplane)과 [`.existingPlaneUsingExtent`](https://developer.apple.com/documentation/arkit/arhittestresult/resulttype/2887459-existingplaneusingextent)에 대해 알아볼 것이다. 

먼저 [`.existingPlaneUsingExtent`](https://developer.apple.com/documentation/arkit/arhittestresult/resulttype/2887459-existingplaneusingextent)은 디바이스에서 나오는 빔이 감지된 면의 면적(`extent`)안에서 교차할 때만 해당 결과를 반환한다. 즉 위에서 탭 제스처가 발생한 좌표가 감지된 면의 면적 범위 안에 위치한다면 해당 교차점을 결과로 반환해주는 것이다. 

반면 [`.existingPlane`](https://developer.apple.com/documentation/arkit/arhittestresult/resulttype/2875738-existingplane)은 면적 범위에 상관없이 감지된 면과 공면에 있다고 판단되는 영역의 교차점도 결과로 반환한다. 즉 면이 한번 감지되면 그것의 면적이 무한하다고 판단하여 결과를 반환한다. 

단순히 이렇게 생각하면 확실히 와닿지 않을 수 있다. 그래서 둘의 차이점을 비교해볼 수 있도록 다르게 코드를 작성하고 앱을 실행해보았고 결과는 다음과 같았다. 

*existingPlaneUsingExtent(좌), existingPlane(우)*

<img src="https://ehdrjsdlzzzz.github.io/img/PlaneUsingExtent.jpeg" style="zoom:50%"><img src="https://ehdrjsdlzzzz.github.io/img/Plane.jpeg" style="zoom:50%">

감지되는 면 이외에는 올릴 수 터치를 해도 오브젝트를 띄울 수 없는 `.existingPlaneUsingPlane`과는 다르게 `.existingPlane`은 감지된 면과 공면으로 무한한 영역에 오브젝트를 띄울 수 있었다. 

이렇게 ARKit를 활용하면 적은 양의 코드로도 재미있는 AR 앱의 기능을 만들 수 있다.