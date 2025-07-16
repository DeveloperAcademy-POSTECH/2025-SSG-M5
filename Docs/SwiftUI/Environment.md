제출일: 2025-06-03

>[!question]
>GQ1. SwiftUI 에서 가져올 수 있는 Environmenet 는 어디까지 있지?

## Description
- `@Environment`는 시스템이 제공하는 **환경 값(environment value)** 을 주입받아 사용하는 속성 래퍼이다.
- SwiftUI의 선언적 구성에 적합하다.
- 프로퍼티 선언 시 `EnvironmentValues`의 **키 경로**를 지정한다.
- 시스템 환경, 접근성 설정, 로케일, 앱 생명주기 상태 등 다양한 정보를 가지고 있으며, `@Environment`는 **읽기 전용**이다다.
- 대부분은 값을 읽을 수는 있지만 직접 설정할 수 없다.
- 일부 값은 `.environment(_:_:)` 뷰 모디파이어를 통해 **직접 설정/덮어쓰기**할 수도 있다.
- 앱 전체에서 일관된 설정을 유지하면서도, 특정 뷰에서는 해당 정보를 바탕으로 **동적으로 UI**를 바꾸도록 할 수 있다.

> [!custom 환경값을 설정하고 싶을 경우]
> https://developer.apple.com/documentation/swiftui/entry()
>Entry 키워드 사용한다.


## 주요 기능

| 용어                  | 내용                                       |
| ------------------- | ---------------------------------------- |
| @Environment        | 시스템 또는 부모 뷰에서 전달되는 값을 **주입받는 속성 래퍼**     |
| EnvironmentValues   | SwiftUI가 제공하는 모든 환경 키의 집합                |
| EnvironmentKey      | 커스텀 환경 값을 정의할 때 사용되는 프로토콜                |
| @Environment(\.key) | `EnvironmentValues` 안에 있는 값을 뷰에 주입할 때 사용 |


| 키                                          | 타입                          | 설명                                                   |
| ------------------------------------------ | --------------------------- | ---------------------------------------------------- |
| `\.scenePhase`                             | `ScenePhase`                | 앱의 현재 상태 (active, inactive, background)              |
| `\.colorScheme`                            | `ColorScheme`               | 다크 모드/라이트 모드 여부                                      |
| `\.horizontalSizeClass`                    | `UserInterfaceSizeClass?`   | 가로 방향의 크기 클래스 (compact, regular 등)                   |
| `\.verticalSizeClass`                      | `UserInterfaceSizeClass?`   | 세로 방향의 크기 클래스                                        |
| `\.locale`                                 | `Locale`                    | 현재 로케일 (언어/지역 정보)                                    |
| `\.calendar`                               | `Calendar`                  | 사용 중인 달력 정보                                          |
| `\.timeZone`                               | `TimeZone`                  | 현재 시간대                                               |
| `\.accessibilityReduceMotion`              | `Bool`                      | 사용자 설정에서 "동작 줄이기" 켰는지 여부                             |
| `\.accessibilityDifferentiateWithoutColor` | `Bool`                      | 색상 외 구분 요소 필요 여부                                     |
| `\.isEnabled`                              | `Bool`                      | 상위 뷰의 enabled 상태 (비활성화 여부)                           |
| `\.undoManager`                            | `UndoManager?`              | 실행 취소 매니저 (텍스트 입력 등에서 사용)                            |
| `\.openURL`                                | `OpenURLAction`             | URL 열기 동작 수행                                         |
| `\.dismiss`                                | `DismissAction`             | 현재 뷰를 닫는 동작 수행 (`.sheet`, `.fullScreenCover` 등에서 사용) |
| `\.presentationMode`                       | `Binding<PresentationMode>` | 뷰의 표시/해제 상태 제어 (iOS 14 이하에서 사용)                      |
| `\.editMode`                               | `Binding<EditMode>?`        | 목록 등에서 편집 모드 제어                                      |
| `\.defaultMinListRowHeight`                | `CGFloat`                   | 리스트 row의 기본 최소 높이                                    |
| `\.managedObjectContext`                   | `NSManagedObjectContext`    | CoreData 컨텍스트 (CoreData 사용 시 필요)                     |
| `\.widgetFamily`                           | `WidgetFamily`              | 위젯 크기 정보 (위젯 개발 시 사용)                                |

> [!@unknown default: 는 뭘까?]
> `Swift`에서 `switch` 문을 사용할 때 **미래에 추가될 수 있는 열거형(enum) 값에 대비하기 위해 사용하는 문법**
> 


> [!그래서... ObservableObject를..! ]
> `Environment` 프로퍼티 래퍼를 이용하면, 뷰 계층 구조 내에서 **ObservableObject(관찰 가능한 객체)**를 주입하고 사용할 수 있다.
> 
> - 주입된 객체는 `Observable` 프로토콜(또는 SwiftUI에서 `ObservableObject`)를 준수해야한다.
> - 객체는 뷰 환경에 직접 넣거나, 특정 키 경로(key path)를 통해 넣을 수 있다
> 

## 코드 예시

```swift
 import SwiftUI

@main
struct MyExampleApp: App {
    @Environment(\.scenePhase) private var scenePhase

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .onChange(of: scenePhase) { newPhase in
            switch newPhase {
            case .active:
                print("앱 active")
            case .inactive:
                print("앱 inactive")
            case .background:
                print("앱 background 이동")
            @unknown default:
                print("알 수 없음")
            }
        }
    }
}

struct ContentView: View {
    @Environment(\.openURL) private var openURL
    @Environment(\.colorScheme) private var colorScheme

    var body: some View {
        ZStack {
            // 다크 모드에 따라 배경색 변경
            (colorScheme == .dark ? Color.black : Color.white)
                .ignoresSafeArea()

            VStack(spacing: 20) {
                Text("SwiftUI 환경값 예제")
                    .font(.title)
                    .foregroundColor(colorScheme == .dark ? .white : .black)

                Button("Apple 홈페이지 열기") {
                    if let url = URL(string: "https://www.apple.com") {
                        openURL(url)
                    }
                }
                .padding()
                .background(Color.blue)
                .foregroundColor(.white)
                .clipShape(Capsule())
            }
        }
    }
}


```




## Keywords
+ LifeCycle
+ ScenePhase
+ dismiss
+ openURL
+ ObservableObject
+ [[SwiftUI]]

## References
- https://developer.apple.com/documentation/swiftui/environment
- 

## 작성자
- #TAENI
-