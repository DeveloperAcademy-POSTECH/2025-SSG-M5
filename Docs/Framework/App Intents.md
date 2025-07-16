#Swift #Framework
>[!question]
>GQ1. App intents가 뭐지?
>GQ2. SwiftUI에선 어떻게 쓸 수 있지?

## Description

**App intents는 사용자가 Siri, Spotlight, Shrotcuts, App의 위젯, Live Activities, Apple Watch 등 다양한 경로를 통해 앱 기능에 빠르게 접근할  수 있도록 해주는 시스템이다.

공식 문서의 내용을 바탕으로 좀 더 자세히 풀어보자면

"App Intents 프레임워크는 Siri, Spotlight, 위젯, 컨트롤 등 다양한 시스템 경험 전반에 걸쳐 앱의 동작과 콘텐츠를 깊이 있게 통합할 수 있는 기능을 제공합니다. Apple Intelligence 및 App Intents의 향상된 기능 덕분에, Siri는 사용자가 앱의 기능을 발견할 수 있도록 앱의 동작을 제안하고, 앱 내 또는 여러 앱에 걸쳐 동작을 수행할 수 있는 능력을 갖추게 된다."

예를 들어서

사용자가 "오늘 할 일을 보여줘"라고 Siri에게 말하면
- **App Intents가 설정된 할 일 앱이 자동으로 실행되고,**
- **Apple Intelligence가 그 사람의 상황을 고려해 적절한 기능을 추천할 수도 있는 구조이다.**

🧐 그렇다면 이걸 어디다가 쓰는거지? 라는 의문이 든다면

➡️ "내 앱 기능 중 어떤 걸 Siri나 Shortcuts, Spotlight 같은 시스템 기능과 연결할 수 있을까?" 
라는 것을 고민하게 된다면 App intents를 사용하면 된다.

---

#### chatGPT 발췌 내용
##### ✅ 1. **앱을 열지 않고도 기능 실행하고 싶을 때**

- 👉 Siri나 단축어 앱에서 “음성”이나 “자동화”로 실행
    
- 예: “오늘 수영 1km 했어” → 앱 실행 없이 기록 완료
    

##### ✅ 2. **사용자가 자주 하는 동작을 Spotlight나 Siri가 추천하게 하고 싶을 때**

- 👉 “앱을 쓰다 보면 자동으로 그 기능을 추천받고 싶다”
    

##### ✅ 3. **위젯이나 Live Activity에서 버튼 하나로 기능 실행**

- 👉 SwiftUI `Button(intent:)` 가능
    
- 예: “+” 버튼 → 바로 기록 추가

---

###### App Intents의 구성 요소
1. AppIntent
	- 실행 가능한 기능을 나타냄. (ex: "오늘 운동 기록을 보여줘)
	- 사용자에게 노출될 이름과 설명, 그리고 실행 로직을 정의함

2. AppEntitiy
	- 인텐트에서 사용할 수 있는 데이터 모델.
	- Siri나 Shortcuts에서 식별할 수 있게 하기 위해 제공됨.
	- 예: 특정 운동 기록, 책, 알람 등

3. AppShortcutsProvider
	- 인텐트를 단축어로 구성하여 시스템이 추천하거나 Spotlight에 노출할 수 있게 만듦.
	- 예: "오늘의 추천 운동 보여줘"와 같은 단축어 설정 가능

4. 매개변수(Parameters)
	- 사용자가 인텐트 실행 시 직접 지정하거나, Siri가 유추할 수 있는 입력 값들


#### SwiftUI에선 어떻게 쓸 수 있지?

###### AppIntent 클래스를 정의하고 perform() 메서드에서 ViewModel이나 앱의 핵심 로직을 호출할 수 있음
```swift
// 공식 문서의 Swift 소개 코드
struct LogWaterIntent: AppIntent {
    static var title: LocalizedStringResource = "Log Water"

    func perform() async throws -> some IntentResult {
        // ViewModel 또는 서비스 호출
        WaterTracker.shared.logWater(amount: 250)
        return .result()
    }
}
```
- Siri로 "물 마신 양 기록해줘"라고 말하면 AppIntent가 트리거 되어, SwiftUI 상태나 저장소 갱신

###### AppShortcuts로 SwiftUI 기능을 Spotlight/Siri 추천에 노출
```swift
struct HealthShortcutsProvider: AppShortcutsProvider {
    static var appShortcuts: [AppShortcut] {
        AppShortcut(
            intent: LogWaterIntent(),
            phrases: ["Log water", "Add water intake"],
            shortTitle: "Log Water",
            systemImageName: "drop.fill"
        )
    }
}
```

## WWDC 2025에서 App Intents

1.  Interactive Snippets - SnippetIntent 프로토콜 도입

	- Action 결과나 확인을 포함한 스니펫을 SwiftUI 요소처럼 구성할 수 있게 돼, 시스템 공간 전체에서 즉각 반응하는 UX 구현이 쉬워짐(음식 주문, 물 마시기 등)
	
	예제 영상: https://developer.apple.com/videos/play/wwdc2025/275/

2. Undoable Intent
	- 인텐트를 되돌릴 수 있는 기능, 사용자 취소 동작 제스처로 인텐트를 취소할 수 있게 되어 실수해도 쉽게 복구 가능
	예를 들어,
	- Siri에서 "오늘 물 500ml 마셨어"라고 했더니 → 앱에 기록됨
    - 근데 곧바로 “아, 실수했네”라고 말하거나 손 제스처로 취소
    - 앱이 그 동작을 되돌림 (기록 삭제)

3. 그 외에도 1) Visual Intelligence와 통합 2) macOS에서 Spotlight Intent 제공 3) Shortcuts 내에 Apple Intelligence 모델을 호출한 액션 가능 등

정리하자면,
이전 AppIntent는 그냥 실행하거나, 배경에서 처리하는 정도였지만
WWDC25 이후, 더 세밀하게 앱 흐름을 제어하거나 SwiftUI View에서 인텐트 실행결과에 따른 유연한 대응이 가능해졌다.

## Keywords
[[Framework]]
## References
- [Apple 공식 Swift 문서](https://developer.apple.com/documentation/appintents)
- [WWDC25 - App Intents](https://developer.apple.com/videos/play/wwdc2025/275/?utm_source=chatgpt.com)
- ChatGPT