---
title: ARKit (1)
subtitle: Class References and Implementation using SceneKit
date: 2018-10-13 11:57:47
tags: ["iOS","ARKit","SceneKit"]
category: ["iOS"]
---

> 다은 ARKit를 활용해 프로젝트를 진행하기 전 공식 문서를 통해 iOS가 제공할 수 있는 AR 경험과 이를 구축하는데 필요한 클래스 그리고 iOS가 증강 현실을 제공하는 방법에 대해 공부하고 간략하게 정리한 내용입니다. 

증강 현실이란 디바이스 카메라 라이브 뷰에 3D 혹은 2D 요소를 추가하여 실제 세상에 존재하는 것 처럼 보이게 만드는 사용자 경험을 제공한다. ARKit는 디바이스 모션 트래킹, 카메라 장면 캡쳐, 고급 장면 처리 및 편의성 표시를 결합하여 통해 AR 경험 구축 작업을 단순화한다. iOS 디바이스의 후면과 전면 카메라와 이러한 기술들을 통해 다양한 AR 경험을 만들 수 있다.

 

#### Augmented Reality with the Back Camera

가장 일반적인 종류의 AR 경험은 후면 카메라를 통해 보는 화면에 다른 시각적 컨텐츠가 추가된 것으로 사용자는 이를 통해 자신의 주변 환경과 상호작용할 수 있다. 

[`ARWorldTrackingConfiguration`](https://developer.apple.com/documentation/arkit/arworldtrackingconfiguration)는 이러한 경험들을 제공한다. ARKit는 사용자가 위치하는 실제 세상을 추적하고 측정하여 이를 당신이 시각적 컨텐츠를 위치시키는 좌표계와 일치시킨다. 월드 트래킹은 사용자 환경에서 물체와 이미지를 인식하고 실제 조명 상태에 반응하는 등 보다 몰입할 수 있는 AR 경험을 만드는 기능을 제공한다. 

> 당신은 사용자 정의 AR 경험을 구출할 필요 없이 사용자의 실제 세상 환경에 3D 객체를 표시할 수 있다. iOS 12에선 당신이 앱 안의 USDZ 파일과 함께 [`QLPreviewController`](https://developer.apple.com/documentation/quicklook/qlpreviewcontroller)를 사용하거나 USDZ 파일이 포함된 웹 컨텐츠를 사파리나 웹킷을 통해 보여준다면 시스템은 3D객체를 위한 AR 화면을 제공한다. 

#### Augmented Reality with the Front Camera 

아이폰 X에선 [`ARFaceTrackingConfiguration`](https://developer.apple.com/documentation/arkit/arfacetrackingconfiguration)는 전면의 트루댑스 카메라를 사용하여 당신이 시각적 컨텐츠를 랜더링하는데 사용할 수 있는 포즈와 사용자 표정 감정에 대한 정보를 제공한다. 예를 들어 화면에 사용자의 얼굴을 보여주고 현실적인 시각적 마스크를 랜더링할 수 있다. 또한 iMessage의 애니모지와 같이 카메라 화면과 ARKit가 제공하는 표정 감정 데이터를 통해 시각적인 캐릭터에 애니메이션 효과를 줄 수 있다. 

------

### Basis Classes

#### [ARSession](https://developer.apple.com/documentation/arkit/arsession)

공유 객체로 증강 현실 경혐 구축에 필요한 디바이스 카메라와 모션 처리를 관리.

`ARSession` 객체는 증강 현실 경험을 만들기 위해 ARKit가 수행하는 주요 프로세스를 조정한다. 이 프로세스에는 디바이스 모션 감지 하드웨어로부터 데이터 읽기, 디바이스에 내장된 카메라 조정 그리고 캡쳐된 카메라 이미지들로부터의 이미지 분석 수행이 포함된다. 세션은 이러한 작업들의 결과를 통합하여 장치가 실제 존재하는 공간과 AR 컨텐츠를 모델링하는 가상 공간 사이의 연결을 구축한다. 

ARKit로 만들어진 모든 AR 경험은 단일 `ARSession` 객체를 필요로 한다. [`ARSCNView`](https://developer.apple.com/documentation/arkit/arscnview) 혹은 [`ARSKView`](https://developer.apple.com/documentation/arkit/arskview)를 사용한다면 이들은 세션 객체를 포함하고 있기 때문에 AR 경험의 시각적인 일부분을 쉽게 만들 수 있다. 만일 자신만의 랜더링 주체를 구축한다면 스스로 `ARSession` 객체를 만들고 유지해야 한다. 

세션을 실행시키기 위해선 구성(Configuration)이 필요하다.

> URLSession과 URLSessionConfiguration 관계와 비슷한건가?

[`ARConfiguration`](https://developer.apple.com/documentation/arkit/arconfiguration) 추상 클래스의 서브클래스들은 ARKit가 실제 세상과 연관된 디바이스 위치와 모션을 추적하는 방법을 결정하고 이는 당신이 만드는 AR 경험의 종류에 영향을 준다. 예를 들어 사용하여 후면 카메라를 통해 사용자 주변 환경을 증강시키는 경험을 제공하기 위해선 [`ARWorldTrackingConfiguration`](https://developer.apple.com/documentation/arkit/arworldtrackingconfiguration)을 사용하라.



#### [ARConfiguration](https://developer.apple.com/documentation/arkit/arconfiguration)

AR 세션 구성을 위한 추상 클래스

이는 추상 클래스로 이 클래스의 인스턴스를 직접 만들거나 작업해선 안된다. 

AR 세션을 실행시키기 위해선 당신이 제작하는 앱이나 게임에서 필요한 AR 경험 종류에 따라 알맞는 [`ARConfiguration`](https://developer.apple.com/documentation/arkit/arconfiguration)의 서브클래스들의 인스턴스를 생성해서 사용해야 한다. 그리고 구성 객체의 프로퍼티를 지정하고 이를 세션의 [`run(_:options:)`](https://developer.apple.com/documentation/arkit/arsession/2875735-run) 메소드에 전달한다. ARKit는 다음과 같은 구성 클래스들을 포함하고 있다.

- [`ARWorldTrackingConfiguration`](https://developer.apple.com/documentation/arkit/arworldtrackingconfiguration) 
  - 후면 카메라를 활용해 디바이스의 위치와 방향을 정교하게 추격하고 지면 감지, 히트 테스팅, 환경에 기반한 조명, 이미지와 사물 인식이 가능한 상위 수준의 AR 경험을 제공
- [`AROrientationTrackingConfiguration`](https://developer.apple.com/documentation/arkit/arorientationtrackingconfiguration)
  - 후면 카메라를 활용해 디바이스의 방향만을 추적하는 기본적인 AR 경험을 제공
- [`ARImageTrackingConfiguration`](https://developer.apple.com/documentation/arkit/arimagetrackingconfiguration)
  - 후면 카메를 활용해 사용자 환경에 상관없이 가시적인 이미지들을 추적하는 기본적인 AR 경험을 제공
- [`ARFaceTrackingConfiguration`](https://developer.apple.com/documentation/arkit/arfacetrackingconfiguration)
  - 전면 카메라를 활용해 사용자 얼굴의 움직임과 감정을 추적하는 AR 경험을 제공
- [`ARObjectScanningConfiguration`](https://developer.apple.com/documentation/arkit/arobjectscanningconfiguration)
  - 후면 카메라를 활용해 고성능 공간 데이터를 수집하고 다른 AR 경험에서의 감지를 위한 참조 객체를 생성



### Display Classes

#### [ARSCNView](https://developer.apple.com/documentation/arkit/arscnview)

카메라 뷰에 3D SceneKit 컨텐츠를 증강시킨 AR 경험을 출력하는 뷰 

`ARSCNView` 클래스는 실제 세상의 디바이스 카메라 뷰에 3D 컨텐츠를 조합한 증강 현실을 만들 수 있는 가장 쉬운 방법을 제공한다. 이 뷰가 제공하는 [`ARSession`](https://developer.apple.com/documentation/arkit/arsession) 객체를 실행시키면 뷰는 다음의 작업들을 수행한다. 

- 뷰는 자동으로 디바이스 카메라가 제공하는 실시간 비디오 피드를 장면의 배경으로 랜더링한다.
- 뷰의 SceneKit 장면의 실제 세상 좌표계는 세션 구성에 의해 설정된 AR 세상 좌표계에 직접 반응한다. 
- 뷰는 자동으로 SceneKit 카메라를 움직여 디바이스의 실제 세상 움직임과 일치시킨다. 

ARKit는 SceneKit 공간과 실제 세상을 자동으로 일치시키기 때문에 실제 세상에서의 위치를 유지하는 것 처럼 보이는 가상 객체를 배치하기 위해선 객체의 SceneKit 위치만 적절히 조절하면 된다. ([Providing 3D Virtual Content with SceneKit](https://developer.apple.com/documentation/arkit/arscnview/providing_3d_virtual_content_with_scenekit) 문서 참고.)

화면에 추가된 객체의 위치를 추적하기 위해서 [`ARAnchor`](https://developer.apple.com/documentation/arkit/aranchor)를 필수로 사용할 필요는 없지만 [`ARSCNViewDelegate`](https://developer.apple.com/documentation/arkit/arscnviewdelegate) 메소드를 구현하면 ARKit에 의해 자동으로 감지된 모든 앵커에 SceneKit 컨텐츠를 추가할 수 있다.



#### [ARSKView](https://developer.apple.com/documentation/arkit/arskview)

카메라 뷰에 2D SpriteKit 컨텐츠를 증강시킨 AR 경험을 출력하는 뷰 

실제 세상의 디바이스 카메라 뷰 내의 3D 공간에 2D 요소를 배치한 증강 현실을 만들기 위해선 `ARSKView`를 사용하라. 이 뷰가 제공하는 [`ARSession`](https://developer.apple.com/documentation/arkit/arsession)을 실행시키면 뷰는 다음의 작업들을 수행한다. 

- 뷰는 자동으로 디바이스 카메라가 제공하는 실시간 비디오 피드를 장면의 배경으로 랜더링한다.
- [`ARSKViewDelegate`](https://developer.apple.com/documentation/arkit/arskviewdelegate) 메소드를 구현하여 SpriteKit 컨텐츠를 실제 위치와 연관시키면 뷰가 자동으로 해당 SpriteKit 노드의 크기를 조절하고 회전시켜 카메라가 보는 실제 세상을 추적하는 것처럼 보인다. 

------

### Implementation

위에서 살펴본 기본적인 클래스들 이외에도 증강 현실을 구현하기 위해서는 많은 클래스들을 사용해야 한다. 이번 포스팅에서는 간단한 직육면체를 띄워볼텐데 이에 필요한 나머지 클래스들은 코드를 통해 살펴보며 그들의 용도와 정의를 살펴보도록 하자. 

우선 코드는 다음과 같다. Xcode 프로젝트를 생성할 때 AR 템플릿을 사용하여 생성하고 다음의 코드를 작성해보자.

```swift
import UIKit
import SceneKit
import ARKit

class ViewController: UIViewController, ARSCNViewDelegate {

    @IBOutlet var sceneView: ARSCNView!

    override func viewDidLoad() {
        super.viewDidLoad()

        // Show statistics such as fps and timing information
        sceneView.showsStatistics = true

        // --- 1
        // Create a new scene
        let scene = SCNScene()

        // --- 2
        let box = SCNBox(width: 0.2, height: 0.2, length: 0.2, chamferRadius: 0)

        // ---- 3
        let material = SCNMaterial() 
        material.diffuse.contents = UIColor.red 

        // --- 4
        let node = SCNNode()
        node.geometry = box
        node.geometry?.materials = [material]
        node.position = SCNVector3(0, 0.1, -0.5)

        // --- 5
        scene.rootNode.addChildNode(node)

        // Set the scene to the view
        sceneView.scene = scene
    }

    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        
        // --- 6
        // Create a session configuration
        let configuration = ARWorldTrackingConfiguration()

        // Run the view's session
        sceneView.session.run(configuration)
    }

    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)

        // Pause the view's session
        sceneView.session.pause()
    }
}
```

여기서 살펴볼 클래스는 총 네 가지이다. 

- [SCNScene](https://developer.apple.com/documentation/scenekit/scnscene)
- [SCNGeometry](https://developer.apple.com/documentation/scenekit/scngeometry)
- [SCNMaterial](https://developer.apple.com/documentation/scenekit/scnmaterial)
- [SCNNode](https://developer.apple.com/documentation/scenekit/scnnode)

> 사실 이들은 ARKit에 속하는 클래스들이 아니다. 3D 컨텐츠를 표현해주는 AR 경험을 제공하기 위해 ARKit가 사용하는 SceneKit에 속하는 클래스들이다. 

그럼 이들을 하나씩 살펴보고 다시 코드로 돌아와보자.

#### [`SCNScene`](https://developer.apple.com/documentation/scenekit/scnscene)

노드 계층과 3D **씬**(scene)을 만드는 전역 프로퍼티들의 컨테이너

SceneKit를 사용해 3D 컨텐츠를 표시하기 위해선 시각적 요소를 표현하는 속성들과 노드들의 계층을 포함하는 씬을 생성해야 한다. 일반적으로 3D 비주얼 에디터로 에셋을 생성하고 Xcode의 SceneKit Scene 에디터를 사용해 이들을 하나의 씬으로 조립하면 SceneKit에서 랜더링할 준비가 끝난 것이다. 

<img src="https://docs-assets.developer.apple.com/published/4f035789b7/592d7da9-814c-4bbd-b1a7-0e07b3003d94.png">

당신의 씬을 출력하기 위해선 런타임에서 불러와야 하며 이를 [`SCNView`](https://developer.apple.com/documentation/scenekit/scnview)의 프로퍼티로 지정해야 한다. 그리고 증강 현실을 제공하기 위한 ARKit의 `ARSCNView`는 `SCNView` 타입의 `scene` 프로퍼티를 갖고 있어 위의 코드에서 우리가 생성한 `SCNView` 타입의 `scene` 객체를 할당해줄 수 있는 것이다. 

> `SCNView` 클래스는 3D SceneKit 컨텐츠를 출력해주는 뷰로 `ARSCNView`의 부모 클래스이다. `ARSCNView`는 `SCNView`와 달리 출력하기 위한 non-nil한 객체를 필요로 한다. 



#### [`SCNGeometry`](https://developer.apple.com/documentation/scenekit/scngeometry)

3D 형태의 모형(Model 혹은 Mesh라 불리기도 한다.)으로 이의 외형을 결정하는 물질(materials)들과 함께 씬에서 출력될 수 있다. 

SceneKit에서 [`SCNNode`](https://developer.apple.com/documentation/scenekit/scnnode) 객체에 붙은 `SCNGeometry` 객체들은 씬의 시각적인 요소들을 구성하고 [`SCNMaterial`](https://developer.apple.com/documentation/scenekit/scnmaterial) 객체들은 `SCNGeometry`들의 외형을 결정하기 위해 붙여진다.

#### Working with Geometry Objects

**씬의 지오메트리(`SCNGeometry` 객체)들의 외형은 노드(`SCNNode` 객체)들과 메터리얼(`SCNMaterial` 객체)들로 조정한다.** 지오메트리 객체는 SceneKit에 의해 랜더링된 시각적 객체의 형태만을 제공한다. 지오메트리 표면의 색과 질감을 지정할 수 있으며 빛에 어떻게 반응하는지를 조정할 수 있고 메터리얼들을 붙여 특수 효과를 추가할 수도 있다. (자세한 사항은 [Managing a Geometry’s Materials](https://developer.apple.com/documentation/scenekit/scngeometry#1655143) 문서를 참고) 지오메트리를 `SCNNode` 객체에 붙임으로써 씬에서 지오메트리의 위치와 방향을 지정할 수 있다. 다수의 노드들이 같은 지오메트리 객체를 참조할 수 있도록 함으로써 그것이 씬 내에서 다양한 위치에서 보일 수 있도록 한다. 

**지오메트리를 쉽게 복사할 수 있으며 그들의 메터리얼 역시 쉽게 변경할 수 있다.** 지오메트리 객체는 변경 불가능한 정점 데이터와 변경 가능한 메터리얼간의 관계를 관리한다. 하나의 지오메트리를 다른 메터리얼의 구성으로 동일한 씬에서 한 번 이상 보이게끔 하려면 [`copy()`](https://developer.apple.com/documentation/objectivec/nsobject/1418807-copy) 메소드를 사용하라. 이러한 복사본은 원본의 점정 데이터를 공유하지만 메터리얼은 별개로 지정할 수 있다. 그러므로 상당한 비용이드는 랜더링 퍼포먼스 없이 하나의 지오메트리를 복사해서 사용할 수 있다. 

**지오메트리 객체에 애니메이션 효과를 줄 수 있다.** 지오메트리와 연관된 정점 데이터는 변경 불가능하지만 SceneKit은 지오메트리에 애니메이션 효과를 줄 수 있는 몇개의 방법을 제공한다. 당신은 [`SCNMorpher`](https://developer.apple.com/documentation/scenekit/scnmorpher) 혹은 [`SCNSkinner`](https://developer.apple.com/documentation/scenekit/scnskinner) 객체를 사용하여 지오메트리의 표면을 변형하거나 외부 3D 제작 도구에서 작성되고 씬 파일에서 로드된 애니메이션을 실행할 수 있다. 또한 [`SCNShadable`](https://developer.apple.com/documentation/scenekit/scnshadable) 프로토콜 메소드를 사용하여 SceneKit의 지오메트리 랜더링을 변경하는 사용자 정의 GLSL 쉐이더 프로그램을 추가할 수 있다. 



#### Obtaining a Geometry Object

SceneKit는 당신의 앱에서 사용할 수 있는 지오메트리 객체를 다양한 방법으로 제공한다. 

| Action                                               | For further Information                                      |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| 외부 3D 제작 도구로부터 만들어진 파일 로드           | [`SCNScene`](https://developer.apple.com/documentation/scenekit/scnscene), [`SCNSceneSource`](https://developer.apple.com/documentation/scenekit/scnscenesource) |
| SceneKit에 내장된 원시 도형 사용과 사용자 정의       | [`SCNPlane`](https://developer.apple.com/documentation/scenekit/scnplane), [`SCNBox`](https://developer.apple.com/documentation/scenekit/scnbox), [`SCNSphere`](https://developer.apple.com/documentation/scenekit/scnsphere), [`SCNPyramid`](https://developer.apple.com/documentation/scenekit/scnpyramid), [`SCNCone`](https://developer.apple.com/documentation/scenekit/scncone), [`SCNCylinder`](https://developer.apple.com/documentation/scenekit/scncylinder), [`SCNCapsule`](https://developer.apple.com/documentation/scenekit/scncapsule), [`SCNTube`](https://developer.apple.com/documentation/scenekit/scntube), [`SCNTorus`](https://developer.apple.com/documentation/scenekit/scntorus) |
| 2D 텍스트 혹은 베이저 곡선을 통해 3D 지오메트리 생성 | [`SCNText`](https://developer.apple.com/documentation/scenekit/scntext), [`SCNShape`](https://developer.apple.com/documentation/scenekit/scnshape) |
| 정점 데이터로부터 사용자 정의 지오메트리 생성        | [`SCNGeometrySource`](https://developer.apple.com/documentation/scenekit/scngeometrysource), [`SCNGeometryElement`](https://developer.apple.com/documentation/scenekit/scngeometryelement), [`init(sources:elements:)`](https://developer.apple.com/documentation/scenekit/scngeometry/1522803-init), [Managing Geometry Data](https://developer.apple.com/documentation/scenekit/scngeometry#1655243) |



####  [`SCNMaterial`](https://developer.apple.com/documentation/scenekit/scnmaterial)

지오메트리가 랜더링 되었을 때 그것의 외형을 정의하는 쉐이딩 속성들의 집합.

메터리얼을 생성할 때 씬에서 다양한 지오메트리에서 재사용할 수 있는 시각적인 속성들의 집합과 그들의 옵션을 정의한다. 

메터리얼은 몇몇의 시각적 프로퍼티들을 갖고 있고 각각은 SceneKit의 조명 및 음영 처리의 다른 부분을 정의한다. 각각의 프로퍼티들은 단색, 질감 혹은 기타 2D 컨텐츠를 제공하는 [`SCNMaterialProperty`](https://developer.apple.com/documentation/scenekit/scnmaterialproperty)  클래스의 인스턴스들이다. 그리고 메터리얼의 [`lightingModel`](https://developer.apple.com/documentation/scenekit/scnmaterial/1462518-lightingmodel) 프로퍼티는 SceneKit이 시각적 속성을 장면의 광원과 결합하여 랜더링 된 씬의 각 픽셀에 대한 최종 색상을 생성하는데 사용하는 수식을 결정한다. 랜더링 프로세스에 대한 자세한 내용은 [`SCNMaterial.LightingModel`](https://developer.apple.com/documentation/scenekit/scnmaterial/lightingmodel)를 참고하라.

[`SCNGeometry`](https://developer.apple.com/documentation/scenekit/scngeometry) 인스턴스의 [`firstMaterial`](https://developer.apple.com/documentation/scenekit/scngeometry/1523485-firstmaterial) 혹은 [`materials`](https://developer.apple.com/documentation/scenekit/scngeometry/1523472-materials) 프로퍼티들을 사용하여 한 개 혹은 그 이상의 메터리얼들을 지오메트리에 붙일 수 있다. 다수의 지오메트리들은 동일한 메터리얼을 참조할 수 있다. 이러한 경우 메터리얼의 속성을 변경하면 이를 참조하고 있는 모든 지오메트리의 외형을 변경하게 된다. 



#### [`SCNNode`](https://developer.apple.com/documentation/scenekit/scnnode)

지오메트리, 조명, 카메라 또는 기타 표현 가능한 컨텐츠를 첨부할 수 있는 3D 좌표계에서 위치와 변형을 나타내는 씬 그래프의 구조 요소. 

`SCNNode` 객체는  그 스스로는 어떠한 시각적인 요소를 갖고 있지 않는다. 이는 상위 노드 기준으로 오직 한 좌표 공간의 변형(위치, 방향 및 크기 조절)만을 나타낸다. 씬을 구성하려면 구조를 생성하기 위해 노드의 계층을 사용하고 조명과 카메라 그리고 지오메트리를 노드에 추가하여 시각적 요소를 생성한다. 

#### Nodes Determine the Structure of a Scene

씬 안의 노드의 계층 혹은 **씬 그래프**는 SceneKit를 사용하여 컨텐츠의 구성과 내용을 표현하고 조작할 수 있는 기능을 정의한다. SceneKit를 사용하여 코드로 제작하거나 3D 제작 도구로 만든 파일에서 불러오거나 혹은 이 두 방법을 함께 사용하여 노드 계층을 생성할 수 있다. SceneKit은 씬 그래프를 구성하고 탐색할 수 있는 여러 기능을 제공한다. 자세한 내용은 [Managing the Node Hierarchy](https://developer.apple.com/documentation/scenekit/scnnode#1655182)와 [Searching the Node Hierarchy](https://developer.apple.com/documentation/scenekit/scnnode#1655236) 문서에 나와있는 메소드를 참고하라.

씬의 [`rootNode`](https://developer.apple.com/documentation/scenekit/scnscene/1524029-rootnode) 객체는 SceneKit이 랜더링한 세계의 좌표계를 정의한다. 이 루트 노드에 추가한 각 자식 노드들은 자신들만의 좌표 시스템을 생성하고 이는 그 자식 노드들에게 상속된다. 당신은 노드의  [`position`](https://developer.apple.com/documentation/scenekit/scnnode/1408026-position), [`rotation`](https://developer.apple.com/documentation/scenekit/scnnode/1408034-rotation) 및 [`scale`](https://developer.apple.com/documentation/scenekit/scnnode/1408050-scale) 프로퍼티들을 사용하여 (또는 [`transform`](https://developer.apple.com/documentation/scenekit/scnnode/1407964-transform) 프로퍼티를 사용하여 직접) 좌표계 간의 변환(transformation)을 결정할 수 있다.

노드 계층과 변환을 사용하여 앱의 요구에 맞는 방식으로 씬의 컨텐츠들을 모델링 할 수 있다. 예를 들어 만약 당신의 앱이 움직이는 태양계 시스템의 뷰를 표현한다면 서로 연관되어 있는 천체를 모델링하는 노드 계층 구조를 만들 수 있다. 각 행성은 궤도와 태양의 좌표계에 정의 된 궤도상의 현재 위치를 갖는 노드가 될 수 있다. 하나의 행성 노드는 자신의 좌표 공간을 정의하는데 이는 자신의 회전 궤도와 달의 궤도 (각 행성의 자식 노드)를 지정하는데 유용하다. 이러한 뷰 씬 계층으로 씬에 현실적인 움직임을 쉽게 추가할 수 있다. 행성 주변의 달의 자전과 태영 주위의 행성에 애니메이션을 적용하면 달이 행성을 따라가는 애니메이션을 확인할 수 있다. 

#### A Node's Attachments Define Visual Content and Behavior

노드 계층은 씬의 공간과 논리적 구조를 결정하지만 시각적 컨텐츠는 정의하지 않는다. [`SCNGeometry`](https://developer.apple.com/documentation/scenekit/scngeometry) 객체를 노드에 붙임으로써 2D 그리고 3D 객체를 씬에 추가할 수 있다. (지오메트리들은 [`SCNMaterial`](https://developer.apple.com/documentation/scenekit/scnmaterial) 객체들에 의해서 외형이 결정된다.) 조명과 그림자 효과가 있는 씬에서 지오메트리를 음영 처리하기 위해선  [`SCNLight`](https://developer.apple.com/documentation/scenekit/scnlight)가 붙여진 노드를 추가하라. 랜더링 할 때 나타나는 뷰 포인트를 조정하려면 [`SCNCamera`](https://developer.apple.com/documentation/scenekit/scncamera) 객체가 붙여진 노드를 추가하라. 

SceneKit 컨텐츠에 물리학에 기반한 행동이나 특수 효과를 주기 위해선 다른 종류의 노드 부착물을 사용하라. 예를 들어 [`SCNPhysicsBody`](https://developer.apple.com/documentation/scenekit/scnphysicsbody) 객체는 물리 시뮬레이션을 위한 노드의 특성을 정의하고 [`SCNPhysicsField`](https://developer.apple.com/documentation/scenekit/scnphysicsfield) 객체는 노드 주변의 물리 구조체에 힘을 적용한다. [`SCNParticleSystem`](https://developer.apple.com/documentation/scenekit/scnparticlesystem) 객체는 노드가 정의한 공간 안에서 불, 비 혹은 떨어지는 낙엽과 같은 입자 효과를 랜더링한다. 

성능을 개선하기 위해서 SceneKit은 여러 노드들 간의 부착물을 공유할 수 있다. 예를 들어 각각의 동일한 모형을 갖는 레이싱 게임에서 씬 그래프는 많은 노드를 포함하는데 각각은 위치와 애니메이션을 지정하지만 모든 자동차 노드는 동일한 지오메트리 객체를 참조한다. 

이제 다시 코드로 돌아가보자. 이젠 전보다 쉽게 코드를 이해할 수 있을 것이다.

1. 씬을 생성한다. 
2. SceneKit에 내장된 원시 도형인 `SCNBox`를 생성한다. 여기서 단위는 미터 단위로 0.2는 20 센티미터를 나타낸다. 
3. 생성한 직육면체 의형을 결정하는 `SCNMaterial` 객체를 생성하고 빨간색을 지정한다. 
4. 직육면체가 위치하게 될 노드를 생성하여 직육면체를 할당하고 위치와 외형을 결정하는 속성 역시 부여한다. 
5. 씬의 노드 계층의 루트 노드에 생성한 노드를 추가해준다. 
6. 세션 구성을 생성하고 세션을 실행시킨다. 

기본적으로 좌표계는 다음 그림과 같이 X,Y 그리고 Z 축을 갖는다.

<img src="https://docs-assets.developer.apple.com/published/ffb3831f78/d937c9e9-05fd-46e3-b72d-eb353b79bfbd.png">

그럼 이제 프로젝트를 실행시키면 후면 카메라를 통해 빨간색의 직육면체를 확인할 수 있다. 