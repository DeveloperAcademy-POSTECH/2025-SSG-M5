제출일: 2025-07-01

>[!question]
>GQ1. NotificationCenter 의 처리과정은 무엇인가?
>GQ2. NotificationCenter 메모리 관리는 어떻게 이루어지나?
>GQ3. NotificationCenter 를 어떻게 관리해야하는가

## Description
- NotificationCenter 란?
	- 객체 간 느슨한 통신(loose coupling) 을 가능하게 하는 event broadcast system
	- 다수의 객체에 이벤트 상태변화를 알리는 방송이라고 생각하면 됨
- Notification 이란?
	- NotificiationCenter 가 전달하는 단일 이벤트의 데이터 단위
	- 앱 내부의 객체 간 메세지를 전달하는 역할로, 옵저버 패턴의 일부이다.

약간 요런 느낌이라해야하나..?
Android BroadCast
https://developer.android.com/develop/background-work/background-tasks/broadcasts?hl=ko
	

> [!NOTE]
> Notification 과 User Notification(UNNotification) 은 다른 개념임.


NotificationCenter 의 내부구조
```Swift

class NotificationCenter {
    private var observers: [NotificationName: [WeakObserver]] = [:]
    private let lock = NSLock() // 스레드 안전성
    
    // 실제 내부 구조 (단순화)
    private struct WeakObserver {
        weak var observer: AnyObject?
        let selector: Selector?
        let block: ((Notification) -> Void)?
        let queue: OperationQueue?
    }
}

```


| 항목        | `Notification`                       | `User Notification` (`UNNotification`)         |
| --------- | ------------------------------------ | ---------------------------------------------- |
| 소속 프레임워크  | Foundation (`NotificationCenter`)    | UserNotifications (`UNUserNotificationCenter`) |
| 주요 클래스    | `Notification`, `NotificationCenter` | `UNNotification`, `UNUserNotificationCenter`   |
| 이벤트 대상    | 앱 내부 객체 (코드)                         | 사용자 (시스템 알림 UI)                                |
| 발생 위치     | 앱 내부에서 직접 post                       | 시스템이 푸시/로컬 알림을 표시할 때                           |
| 용도        | 코드 간 통신 (모듈 간, VC ↔ VM 등)            | 사용자에게 알림을 보내고 응답 처리                            |
| 인터넷 필요 여부 | 필요없음                                 | 로컬일 경우 필요없음, 원격일 경우 필요                         |

```Swift

// UserNotification 을 이용해 알림을 하면서, 처리 후 내부 Notification 으로 전달하는 예시
// 푸시 알림 후 앱 내부에서 이벤트를 처리하는 예시임
// 사용자가 알림을 탭했을 때 → 앱 내부에 Notification 전달
func userNotificationCenter(_: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
    NotificationCenter.default.post(name: .didTapReminderNotification, object: nil, userInfo: response.notification.request.content.userInfo)
}


```


### GQ1. NotificationCenter 의 처리 과정
1. 관찰자를 등록한다 (addObserver)
   옵저버 객체는 특정 `Notification.Name`에 대해 알림을 수신하겠다고 등록
2. 알림 게시(post)
   알림 이름, 옵셔널한 발신자(sender), `userInfo`를 포함한 알림을 게시
3. 알림 전달
   등록된 모든 observer 에게 동기적으로 알림이 전달
   옵저버는 등록 시 설정한 selector/closer를 통해 콜백을 받음

```Swift
// 셀렉터 기반 등록
NotificationCenter.default.addObserver(
    self,
    selector: #selector(handleKeyboardWillShow),
    name: UIResponder.keyboardWillShowNotification,
    object: nil
)

// 클로저 기반 등록
let observer = NotificationCenter.default.addObserver(
    forName: .customEvent,
    object: nil,
    queue: .main
) { [weak self] notification in
    self?.handleCustomEvent(notification)
}

// 클로저 기반으로 옵저버 사용 시 나중에 수동으로 제거하기가 어렵기 때문에 토큰을 저장해서 관리하면 좋다.

```


다시 정리해서,

1. 등록 단계 (옵저버가 특정 알림 이름에 대해 콜백 등록)
2. 발송 단계 (post 메소드 호출 시에 해당 알림 이름의 모든 옵저버를 검색함)
3. 전달 단계 (등록된 순서대로 옵저버의 콜백을 실행시켜준다 - 동기적인 방식)
4. 완료 단계 (모든 옵저버가 처리되면 post 메세지를 반환시킨다)


### Observer Pattern?

#### 옵저버 패턴은 한 객체의 상태 변화를 감지 후, 다른 객체들이 자동으로 반응하도록 연결하는 패턴

➡️ 어떤 일이 발생했을 때, 관심있는 객체들에게 자동으로 알려주는 구조

변화가 발생하면 자동으로 반응하게 하는 시스템.
여러 객체가 한 이벤트에 대해 동시에 반응한다 (1:N)
![[스크린샷 2025-07-01 오후 5.10.20.png]]


##### 옵저버 패턴 구현 예시임<<<
``` Swift

// Subject 의 상태가 바뀔 때마다, 모든 Observer 가 자동으로 반응함
protocol Observer {
    func update(value: Int)
}

class Subject {
    private var observers = [Observer]()
    private var value = 0

    func addObserver(_ observer: Observer) {
        observers.append(observer)
    }

    func changeValue(to newValue: Int) {
        value = newValue
        notifyObservers()
    }

    private func notifyObservers() {
        observers.forEach { $0.update(value: value) }
    }
}

```


이것을 NotificationCenter 로 보자면 이와 같음

NotificationCenter 자체가 Subject 
ViewController, ViewModel = Observer
Notification = Event

``` Swift
NotificationCenter.default.addObserver(self, selector: #selector(...), name: .eventHappened, object: nil)
```

옵저버 패턴의 장점
- 느슨한 결합 : 주체와 반응하는 객체가 서로 모른채로 동작
- 유연성 : 여러 객체가 같은 이벤트에 반응 가능 (1:N 구조)
- 확장성 : 옵저버를 자유롭게 추가/제거 가능
옵저버 패턴의 단점
- 예상치 못한 순환 참조가 일어날 수 있음 (클로저 내부에서 self를 잡을 경우) -> 해결방안 `[weak self]` 사용
- 옵저버 제거 누락 시 메모리 누수될 수 있음 (NotificationCenter에서 옵저버를 수동으로 제거해야함) -> deinit 에서 removeObserver 호출 처리
- 디버깅이 어려움 -> 네이밍 전략, 구조화 필요



### GQ2. NotificationCenter의 메모리 관리

@@@ 알아둘 것!!!!

NotificationCenter 는 위에 설명했듯이, 옵저버에 강한 참조를 갖지 않는다.
옵저버가 **셀렉터 기반**(`addObserver(_:selector:name:object:)`)이면 **자동으로 해제되지 않았다.** → 
iOS 9+ 부터는 셀렉터 기반도 자동으로 해제된다. 하지만 명시적으로 하는 것이 좋다고 한다.

```Swift
NotificationCenter.default.addObserver(self, selector: ..., name: ..., object: ...)


// iOS9+ 부터는 이렇게만 쓰고 removeObjerver 안해도 해제됨
NotificationCenter.default.addObserver(self, selector: #selector(handle), name: .custom, object: nil) 


// iOS9+ 부터 deinit에서 수동 제거 불필요 (하지만 명시적으로 하는게 좋음) 
deinit { 
	NotificationCenter.default.removeObserver(self) 	// 권장사항 
}

```


메모리 관리 방법
- **항상 `[weak self]` 사용**: 클로저 기반 옵저버에서 순환 참조 방지
- **토큰 저장**: 클로저 기반 옵저버의 반환 토큰을 저장하여 수동 해제 가능하게 함
- **명시적 해제**: deinit에서 명시적으로 옵저버 제거
- **적절한 생명주기**: 옵저버의 등록/해제 시점을 명확히 함


### GQ3. NotificationCenter 관리 방법

+ 적절한 활용 시점
  1. 여러 객체가 한 이벤트에 반응할 때(하단에 예시코드 써둠)
  2. 이벤트 소스를 모를 때 : 누가 이벤트를 발생시켰는지 확인할 필요 없다. 그냥 이벤트 발생 여부만 알면 됨.
  3. 시스템 이벤트들을 처리할 때 (키보드 이벤트 처리할 때 자주 사용한다 하단에 예시코드 써둠)


```Swift

// 순환 참조를 예방하려면 이렇게 쓸 것!

class ViewController: UIViewController {
    private var observers: [NSObjectProtocol] = []
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // 순환 참조 위험이 있다. 이렇게 쓰면 안됨
        let badObserver = NotificationCenter.default.addObserver(
            forName: .dataUpdate,
            object: nil,
            queue: .main
        ) { notification in
            self.handleUpdate() // strong reference cycle!
        }
        
        // 안전한 방법
        let safeObserver = NotificationCenter.default.addObserver(
            forName: .dataUpdate,
            object: nil,
            queue: .main
        ) { [weak self] notification in
            self?.handleUpdate() // weak reference
        }
        
        observers.append(safeObserver)
    }
    
    deinit {
        // 클로저 기반은 명시적 제거 필요
        observers.forEach { NotificationCenter.default.removeObserver($0) }
    }
}

```



#### AsyncSequence 를 활용하기

``` Swift
// 비동기 스트림으로 알림 처리
class AsyncNotificationHandler {
    func startListening() async {
        for await notification in NotificationCenter.default.notifications(
            named: .dataUpdate,
            object: nil
        ) {
            await handleDataUpdate(notification)
        }
    }
    
    private func handleDataUpdate(_ notification: Notification) async {
        // 비동기 처리할 것들을 작성한다!!!!
    }
}

```

## 코드 예시

```Swift

//여러 객체가 한 이벤트에 반응할 때


// 로그인 이벤트 → 여러 화면이 동시에 업데이트
extension Notification.Name {
    static let userDidLogin = Notification.Name("userDidLogin")
}

class UserManager {
    func login(user: User) {
        authenticateUser(user)
        
        // 여러 화면에 로그인 완료 알림
        NotificationCenter.default.post(
            name: .userDidLogin,
            object: user,
            userInfo: ["loginTime": Date()]
        )
    }
}

```


```Swift
// 키보드 이벤트 - 시스템에서 직접 발송
// 자주 쓰임!
NotificationCenter.default.addObserver(
    forName: UIResponder.keyboardWillShowNotification,
    object: nil,
    queue: .main
) { [weak self] notification in
    if let keyboardFrame = notification.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? CGRect {
        self?.adjustUI(for: keyboardFrame.height)
    }
}




// 이렇게 보통 쓰임
struct ContentView: View {
    @State private var message = ""
    @State private var isKeyboardVisible = false
    
    var body: some View {
        VStack {
            Text(message)
            TextField("입력", text: .constant(""))
        }
        .padding(.bottom, isKeyboardVisible ? 250 : 0)
        .onReceive(NotificationCenter.default.publisher(for: .messageUpdate)) { notification in
            if let newMessage = notification.object as? String {
                message = newMessage
            }
        }
        .onReceive(NotificationCenter.default.publisher(for: UIResponder.keyboardWillShowNotification)) { _ in
            isKeyboardVisible = true
        }
        .onReceive(NotificationCenter.default.publisher(for: UIResponder.keyboardWillHideNotification)) { _ in
            isKeyboardVisible = false
        }
    }
}

```

```Swift
// 5단계 깊이의 뷰 구조에서 최하단 → 최상단 통신
struct DeepNestedComponent: View {
    var body: some View {
        Button("전역 새로고침") {
            // @Binding으로 5단계 전달하기엔 너무 복잡
            NotificationCenter.default.post(name: .globalRefresh, object: nil)
        }
    }
}

```







#### Extension 조합

```Swift
// 실무에서 자주 사용하는 패턴
extension Notification.Name {
    static let userDidLogin = Notification.Name("userDidLogin")
    static let dataDidUpdate = Notification.Name("dataDidUpdate")
    static let networkStatusChanged = Notification.Name("networkStatusChanged")
}

class UserManager {
    func login(user: User) {
        // 로그인 처리
        authenticateUser(user)
        
        // 여러 화면에 로그인 완료 알림
        NotificationCenter.default.post(
            name: .userDidLogin,
            object: user,
            userInfo: ["loginTime": Date()]
        )
    }
}

class ProfileViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // 로그인 알림 감지
        NotificationCenter.default.addObserver(
            forName: .userDidLogin,
            object: nil,
            queue: .main
        ) { [weak self] notification in
            guard let user = notification.object as? User else { return }
            self?.updateUI(for: user)
        }
    }
}
```



#### 사용하지 말아야할 상황

```Swift

// 1. 단순한 1:1 통신
protocol DataSource {
    func fetchData() -> [Item]
}
// 대신 protocol/delegate 사용

// 2. SwiftUI에서 상태 관리
@StateObject var viewModel = ViewModel()
// 대신 @StateObject, @ObservableObject 사용

// 3. 직접적인 부모-자식 관계
struct ParentView: View {
    @State private var data = ""
    
    var body: some View {
        ChildView(data: $data) // 직접 전달
    }
}
```


## Keywords
+ NotificationCenter
+ Notification
+ Keyboard
+ ObserverPattern
+ Combine

## References
- https://developer.apple.com/documentation/foundation/notificationcenter
- 예시코드는 내칭구 지희지씨(GPT)

## 작성자
- #TAENI 