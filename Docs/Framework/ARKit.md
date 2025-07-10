
> [!question]
> GQ1. ARKit은 무엇일까?
> GQ2. ARKit은 어떻게 사용할 수 있을까?
> GQ3. UIKit과 SwiftUI에서 사용법이 다를까?


## Description

##### ARKit은 무엇일까?
| Integrate hardware sensing features to produce augmented reality apps and games.

애플 공식문서에 따르면 하드웨어 센싱 기능을 통합하여 증강현실 앱과 게임을 만든다.라고 서술돼있다.

위 말을 쉽게 풀어서 다시 설명해보자면

증강현실(AR)은 스마트폰이나 태블릿 같은 기기의 센서(카메라 등)를 통해 보이는 실제 화면 위에 2D(평면)나 3D(입체) 이미지를 덧붙여서, 마치 그 이미지가 현실 위에 있는 것처럼 보여주는 기술이다.
ARKit은 기기의 움직임 추적, 주변 환경 인식, 장면 이해 등 여러 기능을 한데 모아서, 개발자가 쉽게 AR 앱을 만들 수 있도록 도와준다.

##### ARKit은 어떻게 사용할 수 있을까?
ARKit은 크게 4가지의 뼈대로 이루어져있다고도 볼 수 있다. 하나씩 살펴보자
- **ARSession**
	- ARKit의 **핵심**으로, AR 경험의 상태를 관리한다.
	- 카메라, 모션 센서, LiDAR 등 기기의 센서를 통해 현실 세계를 분석하고 AR 데이터(프레임, 앵커, 트래킹 상태 등)를 주기적으로 업데이트한다.
	- 앱은 ARSession을 통해 AR 데이터를 받고, AR 경험을 시작·중단·재설정할 수 있다.
	- ex: session.run(configuration) 으로 AR 세션을 시작

- **ARConfiguartion**
	- ARSession이 어떤 종류의 AR 경험을 할지 정의하는 **설정 객체**
	-  현실 세계에서 무엇을 인식하고 추적할지 결정
    
	- 종류:
	    - ARWorldTrackingConfiguration (가장 일반적, 6DoF 트래킹 + 평면 탐지)
        
	    -  ARImageTrackingConfiguration (이미지 추적용)
        
	    - ARFaceTrackingConfiguration (Face ID 센서 기반)
    
	- ex: let config = ARWorldTrackingConfiguration(); session.run(config)

- **ARAnchor**
	- AR 공간에서 가상 콘텐츠를 고정할 좌표계 기준점
	- Anchor은 ARKit이 받아온 현실 세계의 특정 지점(평면, 이미지, 얼굴 등)에 가상 객체를 연결해준다.
	- Anchor는 위치와 방향을 포함하며, ARKit은 Anchor를 지속적으로 추적하여 객체가 정확히 유지되도록 한다

- **ARFrame**
	- ARSession이 매 프레임마다 생성하는 AR 데이터 스냅샷
	- ARFrame에는 현재 카메라 영상, 트래킹 상태, Anchor 정보, LiDAR 데이터, 환경 정보 등이 포함
	- 앱은 ARFrame을 통해 매 순간 AR 상태를 분석하고 업데이트된 정보를 얻을 수 있음
	- ex: session.currentFrame으로 현재 프레임에 접근

###### 코드 예시
```swift
// 공식 문서의 Swift 소개 코드
import ARKit

// ARSession 생성 (AR 경험의 중심 엔진)
let session = ARSession()

// ARConfiguration 생성 (ARSession을 어떻게 동작시킬지 설정)
// 여기서는 WorldTrackingConfiguration 사용
let configuration = ARWorldTrackingConfiguration()
configuration.worldAlignment = .gravity  // 중력 방향으로 Y축 정렬

// ARSession 시작
session.run(configuration)

// ARAnchor 생성 (AR 공간상의 가상 객체 기준점)
// transform에는 위치와 방향 정보 포함
let anchorTransform = simd_float4x4(translation: [0, 0, -0.5]) 
let anchor = ARAnchor(transform: anchorTransform)

// Anchor를 세션에 추가
session.add(anchor: anchor)

// ARFrame 접근 (현재 카메라 영상과 AR 데이터 스냅샷)
if let currentFrame = session.currentFrame {
    // 예: 카메라 이미지, 트래킹 상태, Anchors 정보 얻기
    let cameraTransform = currentFrame.camera.transform
    let anchors = currentFrame.anchors
    // 여기서 cameraTransform, anchors 등을 활용 가능
}

```


##### UIKit과 SwiftUI에서 사용법이 다를까?
 > **ARKit은 UIKit을 기반이며, SwiftUI에서는 이를 감싸거나 RealityKit을 함께 쓰는 방식**
###### UIKit에서 ARKit 사용
- ARKit은 UIKit + SceneKit/SpriteKit과 자연스럽게 결합되도록 설계되었음
- UIKit 앱에서는 AR을 ARSCNView (SceneKit 기반 3D 렌더링) 또는 ARSKView (SpriteKit 기반 2D 렌더링)로 바로 추가 가능
	```swift
let arView = ARSCNView(frame: view.bounds)
view.addSubview(arView)

let config = ARWorldTrackingConfiguration()
arView.session.run(config)

```

###### SwiftUI에서 ARKit 사용
- SwiftUI에서는 ARKit을 직접 다루는 뷰가 없다.
- 대신 **UIViewRepresentable**을 사용해 ARSCNView(Deprecated - SceneView) 또는 ARView를 SwiftUI 뷰 계층에 삽입
- 요새는 RealityKit + ARView와 SwiftUI를 결합하는게 트렌드
	```swift
struct ARContainerView: UIViewRepresentable {
    func makeUIView(context: Context) -> ARSCNView {
        let arView = ARSCNView()
        let config = ARWorldTrackingConfiguration()
        arView.session.run(config)
        return arView
    }

    func updateUIView(_ uiView: ARSCNView, context: Context) {}
}

struct ContentView: View {
    var body: some View {
        ARContainerView()
            .edgesIgnoringSafeArea(.all)
    }
}
```
- 여기서 **ABSCNView가 iOS 26부터 Deprecated 되었으므로**, ARView를 채택해서 쓰는 것이 바르다.
```swift
struct ARContainerView: UIViewRepresentable {
    func makeUIView(context: Context) -> ARSCNView {
         let arView = ARView(frame: .zero)
        let config = ARWorldTrackingConfiguration()
        arView.session.run(config)
        return arView
    }

    func updateUIView(_ uiView: ARView, context: Context) {}
}

struct ContentView: View {
    var body: some View {
        ARContainerView()
            .edgesIgnoringSafeArea(.all)
    }
}
```

##### 정리
1. ARKit은 크게 ARSession, ARConfiguration, ARAnchor, ARFrame 4가지 뼈대로 이루어져 있다.
2. ARKit의 근간은 UIKit이어서, SwiftUI에서 쓰려면 UIViewRepresentable로 래핑해주어야 한다
3. 요새는 RealityKit + ARView와 SwiftUI를 결합하며 쓰는 방식이 현대적이다.

위 3가지 외에도, ARKit 내부에는 World Tracking, Plane Detection, Image/Object Detection 등 기능이 엄청 많다. 그거는 다음 시간에 알아보자.

## References

- [Apple 공식 Swift 문서](https://developer.apple.com/documentation/swift)
- ChatGPT
