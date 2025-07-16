제출일: 2025-06-03

>[!question]
>GQ1. SwiftUI에서 앱 생명주기를 감지하는 방법에는 어떤 것이 있는가?
>GQ2. @Environment(.scenePhase)를 사용한 상태 감지 코드는 어떻게 구성되는가?

## Description
- SwiftUI는 선언형 UI 프레임워크로, 뷰 계층구조 내에서 환경(Environment) 기반의 상태 전달을 통해 생명주기 변화를 감지한다. 
- ScenePhase 라는 환경 값(EnvironmentValue)를 통해 씬(Scene)이 활성화, 비활성화, 백그라운드 상태 전환을 추적한다.
- @Environment(\.scenePhase) 프로퍼티 래퍼는 현재 씬의 상태를 자동으로 반영하고 있다.
- .onChange(of:) 뷰 모디파이어를 이용해 상태 변화 시점을 감지하여 반응형 로직을 선언적으로 구현할 수 있다.


> [!NOTE]
> ## SwiftUI의 선언형(Declarative) UI 원리
> SwiftUI는 **선언형(Declarative) UI 프레임워크**입니다. 즉, UI를 상태(state)와 데이터에 기반해서 선언적으로 작성한다.
> 뷰는 상태가 변할 때마다 **자동으로 재계산(렌더링)** 되어 화면에 반영된다.
> 상태 관리(State Management)와 뷰 갱신을 핵심으로 보고, 이 과정에서 내부적으로 상태 변화 감지 및 뷰 트리 업데이트를 최적화한다.


## 주요 기능

> [!내부 동작 과정]
> 1. **시스템(운영체제)이 앱 상태 변화를 감지** (예: 앱이 백그라운드 진입)
> 2. **SwiftUI 런타임이 ScenePhase 값을 갱신** (`.active` → `.background` 등)
> 3. **`@Environment(\.scenePhase)` 프로퍼티를 사용하는 뷰에 값 변경 알림 전달**
> 4. . **SwiftUI가 이 값을 참조하는 뷰를 자동으로 다시 그리기(렌더링) 시작**
> 5. 뷰 내부 코드가 상태별 UI를 조건문 등으로 작성했다면, 이에 따라 UI가 즉시 갱신됨

    
## 코드 예시

```swift

import SwiftUI

@main
struct LifecycleAwareApp: App {
    @Environment(\.scenePhase) private var scenePhase

    var body: some Scene {
        WindowGroup {
            RootView()
        }
        .onChange(of: scenePhase) { newPhase in
            handleScenePhaseChange(newPhase)
        }
    }

    private func handleScenePhaseChange(_ phase: ScenePhase) {
        switch phase {
        case .active:
            print("앱 활성 상태 - 사용자 상호작용 가능")
            // 네트워크 재연결, UI 갱신 등 수행
        case .inactive:
            print("비활성 상태 - 전환 중 혹은 간헐적 중지 상태")
            // 빠른 임시 저장 또는 타이머 정지 권장
        case .background:
            print("백그라운드 상태 - 리소스 최소화 및 데이터 저장")
            // 데이터 영속화, 긴급 작업 종료 등
        @unknown default:
            print("알 수 없는 상태 발생 - 향후 업데이트 대비")
        }
    }
}


```

## Keywords
+ Environment
+ ScenePhase
+ [[LifeCycle]]

## References
- https://developer.apple.com/documentation/uikit/managing-your-app-s-life-cycle
- https://developer.apple.com/documentation/swiftui/environment

## 작성자
- #TAENI 
