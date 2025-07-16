#Concurrency 

제출일: 2025-06-26

>[!question]
>GQ1. Data Race란 무엇인가?
>GQ2. Actor란 무엇인가?
>GQ3. MainActor란 무엇인가?

## Description
- 개요와 설명을 작성

## 주요 기능
### GQ1. Data Race란 무엇인가?
Data Race(데이터 경합)은 여러 스레드가 동시에 같은 데이터에 접근하고, 그 중 하나 이상이 데이터를 변경할 때 발생하는 문제. 
동기화가 제대로 이루어지지 않으면 데이터가 꼬이거나, 예측할 수 없는 버그 발생함
- 멀티스레드 환경에서 자주 발생하는 동시성 버그의 대표적인 예
- Swift에서는 Data Race를 방지 하기 위해 새로운 동시성 모델(Actor, MainActor등) 이 도입됨

```swift
class Counter{
	var value = 0
	func increment(){
		value += 1
	}
}

let counter = Counter()
DispatchQueue.concurrentPerform(iterations: 10000) { _ in
    counter.increment()
}
```
현재 코드는 여러 쓰레드가 동시에 `counter.value` 라는 같은 변수에 접근해서 값을 증가시키고 있다.
여기서 문제는 여러 스레드가 동시에 `value`를 읽고, 동시에 `value += 1`연산을 수행한다.

두 쓰레드가 동시에 값을 읽고, 동시에 값을 덮어쓰면 중간에 더한 값이 사라질 수 있다. 즉, 10,000번을 더해도 실제 결과가 10,000이 아닐 수 있다. 

> 즉, 동시에 여러 쓰레드가 같은 메모리에 접근하고 최소한 하나가 쓰기 작업을 할 때 동기화 없이 실행하면 Data Race가 발생한다. 

### GQ2. Actor란 무엇인가?
앞서 설명한 `Data Race` 문제를 안전하게 해결하기 위해 Swift에 도입된 것이 Actor 이다. 

Actor는 Swift 5.5에서 도입된 Concurrency 기능으로, 여러 쓰레드에서 동시에 접근할 수 있는 데이터를 안전하게 보호하는 새로운 타입이다.
기존의 `class`, `struct`, `enum` 처럼 사용할 수 있지만, 가장 큰 차이는 프로퍼티에 대한 접근이 자동으로 직렬화* 된다는 점이다.
(*여러 작업(스레드, Task 등)이 동시에 Actor 내부 데이터에 접근하려고 해도 Swift가 자동으로 한 번에 하나씩만 접근하도록 처리해 준다)

```swift
actor SafeCounter {
    private var value = 0

    func increment() {
        value += 1
    }

    func getValue() -> Int {
        return value
    }
}

let safeCounter = SafeCounter()

await withTaskGroup(of: Void.self) { group in
    for _ in 0..<1000 {
        group.addTask {
            await safeCounter.increment()
        }
    }
}
let result = await safeCounter.getValue()
print("최종 값: \(result)") // 항상 1000이 출력 
```
한 번에 하나씩만 Actor 내부에 접근 하도록 해주기 때문에 항상 1000이 출력됨.

### GQ3. MainActor란 무엇인가?
MainActor는 Global Actor로 항상 메인 쓰레드에서 실행되어야 하는 코드 (UI 업데이트 등)를 안전하게 처리하기 위한 Concurrency 도구이다. 

`@MainActor` 를 통해 UI 업데이트나 메인 쓰레드에서만 접근해야하는 리소스에 대한 Data Race를 방지할 수 있다.

`@MainActor` 사용 시 기존의 `DispatchQueue.main.async` 를 반복적으로 쓰지 않아도 됨.

```swift
@MainActor
class AccountViewModel: ObservableObject {
    @Published var username = "Anonymous"
    @Published var isAuthenticated = false

    func updateUsername(_ name: String) {
        self.username = name
    }
}

struct AccountView: View {
    @StateObject var viewModel = AccountViewModel()
    var body: some View {
        VStack {
            Text("Welcome, \(viewModel.username)!")
            Text(viewModel.isAuthenticated ? "You are logged in." : "You are not logged in.")
        }
    }
}

```

AccountViewModel에 `@MainActor`를 붙이면, 모든 프로퍼티와 메서드가 항상 메인 스레드에서 실행되어 UI 데이터 레이스를 방지한다. 

여러 ViewModel에 `@MainActor`를 사용할 수 있다. 단지, 메인 쓰레드에서 실행되고 UI 업데이트 관련된 데이터 레이스를 방지하기 위해 만들어짐
## Keywords
- [[Concurrency]]

## References
- [Eliminate data races using Swift Concurrency](https://developer.apple.com/videos/play/wwdc2022/110351)
- [Beyond the basics of structured
- concurrency](https://developer.apple.com/videos/play/wwdc2023/10170)
## 작성자
- Jun