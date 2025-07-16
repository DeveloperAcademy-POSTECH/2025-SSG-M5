#swift #Concurrency 

제출일: 2025-06-12

>[!question]
>GQ1. Concurrency란 무엇일까?  
>GQ2. Task, TaskGroup, async let의 차이는 뭘까?  
>GQ3. 구조화된 concurrency(Structured Concurrency)의 장점은?  
>GQ4. 기존의 GCD(DispatchQueue)보다 어떤 점이 명확하고 유용할까?
## Description
- <span style="background:#fff88f">Swift Concurrency는 비동기(Asynchronous) 및 병렬(Parallel) 작업을 효율적이고 안전하게 작성할 수 있도록 지원하는 기능이다.</span>
- Concurrency는 여러 작업이 동시에 *진행 상태*가 되도록 설계하는 전략이며, 그 안에서 `async/await`, `Task`, `TaskGroup` 등을 활용해 논리적 동시 실행 구조를 만들 수 있다.

---
## 주요 기능
### GQ1. Concurrency란 무엇인가?
- **Concurrency(동시성)는 앱이 여러 작업을 동시에 진행 상태로 유지하고 효율적으로 관리하는 프로그래밍 모델**을 의미한다.
- 예를 들어, 사용자 입력에 즉각 반응하면서도 동시에 이미지 다운로드나 데이터 처리 같은 작업을 **지연 없이 처리**할 수 있도록 만든다.

- Swift에서는 5.5 이전까지 `GCD(Grand Central Dispatch)`나 `OperationQueue`를 사용해 Concurrency를 구현해왔지만,
- Swift 5.5부터는 `async/await, Task, actor`와 같은 **언어 수준의 동시성 기능**이 도입되어 더 안전하고 간결하게 비동기 작업과 병렬 실행을 다룰 수 있게 되었다.

#### ⚠️ 여기서 잠깐! 그럼 Concurrency = Asynchronous?
>  모든 Concurrency는 Async를 포함하지만, 모든 Async가 Concurrency인 것은 아니다.
>  📌 Concurrency != Async

**Async** 
아래는 순차적 비동기 작업이다.
```swift
func doSomething() async {
    let a = await fetchData()
    let b = await processData()
    print("A: \(a), B: \(b)")
}
```
- `fetchData()`가 끝난 후에야 `processData()`가 시작된다.
- 비동기지만, Concurrency는 아님
- CPU는 `doSomething` 작업 하나만 진행

**Concurrency**
아래는 비순차적 비동기 작업이다. 
```swift
func doSomethingConcurrently() async {
    async let a = fetchData()
    async let b = processData()

    let (resultA, resultB) = await (a, b)
    print("A: \(resultA), B: \(resultB)")
}
```
- `async let`으로 `fetchData()`와 `processData()` 이 둘이 동시에 시작된다.
- `await`은 두 작업이 완료되길 기다린다
- 동시적으로 진행되는 구조 → Concurrency

---
### GQ2. async let, Task, TaskGroup, 의 차이는 무엇일까?
Swift Concurrency에서는 여러 비동기 작업을 동시적으로 수행하기 위해 다양한 **비동기 실행 단위**를 제공한다.

그중 대표적인 세 가지가 바로 `async let`, `Task`, `TaskGroup`이다.

이들은 모두 **비동기 작업을 동시에 실행**할 수 있게 해주지만, **용도와 제약, 제어 방식이 다르다.**
- **병렬 작업이 2~3개 정도**이고 결과를 한번에 받으면 됨 → async let
- **UI 버튼 클릭 등 이벤트 기반으로 작업을 실행**해야 함 → Task
- **여러 개의 반복적인 작업을 병렬 처리**하고 싶을 때 → TaskGroup
#### 1. `async let` - 간단한 구조화된 병렬 처리
- `async let`은 컴파일러가 내부적으로 `Task`를 생성하며, 구조화된 concurrency를 자동으로 보장한다.
- 반드시 **같은 함수 스코프 내**에서 `await`으로 결과를 받아야 한다.
- 여러 비동기 작업을 병렬로 시작하고, 나중에 한 번에 결과를 기다릴 때 적합하다.
> `async let`은 **오류 처리가 단순한 경우에 적합**하며, 복잡한 에러 핸들링에는 적절하지 않을 수 있다.

*예제 코드)*
```swift
func fecthUserAndPosts() async {
	async let user = fetchUserInfo()
	async let posts = fetchUserPosts()

	let (userInfo, userPosts) = await (user, posts)
	print("User: \(userInfo), Posts: \(userPosts)")
}

func fetchUserInfo() async throws -> String {
	try await Task.sleep(nanoseconds: 1_000_000_000) // 1초 대기
	return "John Doe"
}

func fetchUserPosts() async throws -> [String] {
	try await Task.sleep(nanoseconds: 2_000_000_000) // 2초 대기
	return ["Post 1", "Post 2", "Post 3"]
 }

fetchUserAndPosts()
```
- 위 함수를 실행하면, `fetchUserInfo()`와 `fetchUserPosts()`는 **동시에 시작**된다.
- `fetchUserInfo()`는 1초 후 완료되지만, 두 작업의 결과는 **가장 오래 걸리는 작업인 `fetchUserPosts()`의 완료 시점 (2초 후)에 함께 반환된다.**
- 결과적으로, **전체 수행 시간은 약 2초가 된다.**

#### 2. `Task` - 비구조화 동시 작업
- `Task {}`는 **명시적으로 새로운 비동기 작업**을 생성하며, 호출한 컨텍스트와 **독립적으로 실행**된다.
- `async let`과 달리 **부모 작업(Task)의 생명주기와 연결되지 않기 때문에**, 명시적인 **취소, 에러 처리, 완료 감시**가 필요하다.
- **UI 트리거, 알림 수신, 백그라운드 데이터 로딩**처럼 시스템 반응 기반의 동시 작업에 적합하다.
- `Task(priority:)`, `Task.detached(priority:)`로 우선순위, context 상속 여부를 설정할 수 있다.

> ❗️`Task`는 **자유도는 높지만**, 생명주기를 명시적으로 관리하지 않으면 메모리 누수, 작업 유실, 무의미한 동시 실행 등의 문제가 발생할 수 있다.

*예제 코드)*
```swift
func loadDataOnTap() {
    Task {
        let data = await fetchData()
        print("Fetched data: \(data)")
    }
}

func fetchData() async -> String {
    try? await Task.sleep(nanoseconds: 1_000_000_000)
    return "Data from network"
}
```

#### ⚠️  여기서 잠깐! Task는 자동으로 취소되지 않는다 — 직접 취소해줘야 한다
- Swift의 Task {}는 **비구조화된 동시 작업**이다.
- 즉, **부모 Task나 뷰의 생명주기와 자동으로 연결되지 않기 때문에**, 작업이 시작되면 외부에서 **직접 취소해주지 않는 한 계속 실행**된다.

> *예시 상황)*
> 사용자가 버튼을 눌러 Task {} 내부에서 이미지를 다운로드하는 작업을 시작한다.
> 그런데 **곧바로 화면에서 벗어나면**(예: 뒤로 가기), 해당 Task는 자동으로 취소되지 않고 **계속 실행**된다.

이는 **리소스 낭비**뿐 아니라, 이후에 UI 업데이트가 발생할 경우 **크래시나 메모리 누수**로도 이어질 수 있다.

따라서, `Task` 인스턴스를 변수로 저장하고,`onDisappear` 등에서 명시적으로 `.cancel()` 호출해준다.
```swift
@State private var task: Task<Void, Never>?

.onAppear {
    task = Task {
        await loadImage()
    }
}

.onDisappear {
    task?.cancel()  // 유저가 뒤로가면 Task 취소
}
```

또한, `Task{}` 내부에서 다음과 같이 `checkCancellation`을 이용해 취소 신호를 감지하고 중단해줘야함.
```swift
func loadImage() async {
    do {
        try Task.checkCancellation()
        try await Task.sleep(nanoseconds: 3_000_000_000)
        print("✅ Image downloaded")
    } catch {
        print("❌ Task was cancelled before completion")
    }
}
```

#### 3. TaskGroup - 여러 작업을 병렬로 반복 처리하기
- `TaskGroup`은 **여러 개의 비동기 작업을 병렬로 실행**하고, **모두 완료될 때까지 기다렸다가 결과를 모아주는 구조화된 concurrency 도구**이다
- 작업 중 하나라도 실패하면 **전체 그룹 작업이 자동 취소**되며, 에러도 throw된다.
- Swift의 **Structured Concurrency** 개념을 가장 잘 보여주는 도구 중 하나다.

> 주로 “여러 데이터를 동시에 받아야 할 때”, “API를 반복 호출해 데이터를 모을 때”, “병렬로 무거운 작업을 처리해야 할 때” 유용하다.

*예제 코드)*
```swift
func fetchMultipleData() async -> [String] {  
	await withTaskGroup(of: String.self) { group in  
		var results: [String] = []  
  
		group.addTask { await fetchData1() }  
		group.addTask { await fetchData2() }  
  
		for await result in group {  
			results.append(result)  
		}  
		return results  
	}  
}  
  
Task {  
	let data = await fetchMultipleData()  
	print(data)  
}}
```
#### ⚠️ 여기서 잠깐! Task{ func(), func()} 실행하는 것과 TaskGroup은 뭐가 다른거야?
```swift
Task {
    let data1 = await fetchData1()
    let data2 = await fetchData2()
    print([data1, data2])
}
```
`fetchData1()`이 끝나야 `fetchData2()`가 시작됨. 즉 병렬 구조가 아님

```swift
func fetchMultipleData() async -> [String] {
    await withTaskGroup(of: String.self) { group in
        var results: [String] = []

        group.addTask { await fetchData1() }
        group.addTask { await fetchData2() }

        for await result in group {
            results.append(result)
        }

        return results
    }
}
```
둘다 동시에 시작되고, 느린 쪽이 끝날 때까지 기다림

> 하나씩 순서대로 해야 한다면 → `Task{ fetchA(); fetchB(); }`
> 병렬로 여러 개를 동시에 실행하고 결과를 모아야한다면 → TaskGroup

---
### GQ3. 구조화된 concurrency(Structured Concurrency)의 장점은?
**Structured Concurrency**란 비동기 작업을 하나의 함수 또는 스코프 내에서 생성하고, 그 범위 안에서 종료까지 관리하는 방식
- `async let`, `withTaskGroup`, `withThrowingTaskGroup` 등이 대표적인 구조화된 방식
- 비동기 코드 흐름을 마치 동기 코드처럼 위에서 아래로 명확하게 표현할 수 있도록 도와줌
- 모든 작업은 부모 작업의 생명주기에 따라 생성되고 종료됨
- 부모함수가 끝나면, 그 안의 비동기 작업도 자동으로 정리됨
- 에러 발생 시 나머지 작업을 자동 취소 해줌

> 안정성, 가독성 측면에서 GOAT!

반면, Unstructured Concurrency는
- `task.cancel()`를 이용하여 Task를 직접 취소 시켜야함
- 작업 취소 하지 않으면, 뷰가 사라져도 작업은 계속 진행되어 메모리 누수나 앱 크래시 발생 가능성 높음
- 개별 `try-catch` 또는 취소 감지 코드가 필요함

### GQ4. 기존의 GCD(DispatchQueue)보다 어떤 점이 명확하고 유용할까?

| 항목               | GCD (DispatchQueue 기반)                                 | async/await (구조화된 동시성)                                           |
|--------------------|-----------------------------------------------------------|-------------------------------------------------------------------------|
| 코드 가독성         | ❌ 클로저 중첩으로 흐름이 복잡 (콜백 지옥 발생)              | ✅ 동기 코드처럼 읽힘 (위→아래 흐름으로 자연스러움)                         |
| 에러 처리          | ❌ 각 클로저마다 에러 처리 필요                            | ✅ `try-catch`로 통일된 에러 처리 가능                                     |
| 작업 취소          | ❌ 수동 플래그(`isCancelled`)로 상태 관리                   | ✅ `Task.cancel()` + `Task.checkCancellation()`으로 명시적이고 일관됨     |
| 병렬 작업 처리      | ❌ DispatchGroup 사용 + 수동 완료 추적                      | ✅ `TaskGroup`으로 선언적 병렬 처리 가능, 자동 완료 추적                   |
| 작업 우선순위       | ❌ QoS 기반으로 제한적 제어 (`.background`, `.userInitiated`) | ✅ `Task(priority:)`로 세분화된 우선순위 설정 가능                         |
| 스코프 관리        | ❌ 개발자가 수동으로 관리                                  | ✅ Swift가 작업의 생명주기와 메모리를 자동 관리                           |
| 유지보수 용이성     | ❌ 흐름 추적 어려움, 에러 유입 위치 파악 어려움              | ✅ 직관적인 흐름과 자동 오류 전파로 디버깅/유지보수가 쉬움                |
#### ⚠️ 여기서 잠깐! GCD vs async/await 비교

| 작업 목적                       | GCD 방식                                                   | async/await 방식                                              |
|-------------------------------|-------------------------------------------------------------|--------------------------------------------------------------|
| 백그라운드 작업 실행           | `DispatchQueue.global().async { ... }`                     | `Task { ... }`                                                |
| 메인 스레드에서 UI 업데이트    | `DispatchQueue.main.async { ... }`                         | `@MainActor`, `MainActor.run { ... }`                        |
| 지연 실행 (Delay)              | `DispatchQueue.main.asyncAfter(deadline:)`                | `try await Task.sleep(nanoseconds:)`                         |
| 병렬 작업                      | `DispatchGroup`                                            | `withTaskGroup`, `withThrowingTaskGroup`                     |
| 작업 취소                      | 상태 플래그로 수동 제어 (`isCancelled`)                    | `Task.cancel()`, `Task.isCancelled`, `Task.checkCancellation()` |
| 우선순위 설정                  | `DispatchQueue.global(qos: .background)`                  | `Task(priority: .background) { ... }`                        |

##### 1. 백그라운드 작업 + 메인 스레드 UI 업데이트
**🔴 GCD 방식**
```swift
DispatchQueue.global().async {
    let result = fetchDataSync()
    DispatchQueue.main.async {
        self.label.text = result
    }
}
```

**🟢 async/await 방식**
```swift
Task {
    let result = await fetchData()
    await MainActor.run {
        self.label.text = result
    }
}
```

##### 2. 지연(delay) 실행
**🔴 GCD 방식**
```swift
DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
    print("2초 후 실행")
}
```

**🟢 async/await 방식**
```swift
Task {
    try await Task.sleep(nanoseconds: 2_000_000_000)
    print("2초 후 실행")
}
```

##### 3. 여러 작업을 병렬 처리 후 하나로 합치기
**🔴 GCD 방식**
```swift
let group = DispatchGroup()
var result1 = ""
var result2 = ""

group.enter()
DispatchQueue.global().async {
    result1 = fetchData1()
    group.leave()
}

group.enter()
DispatchQueue.global().async {
    result2 = fetchData2()
    group.leave()
}

group.notify(queue: .main) {
    print("Results: \(result1), \(result2)")
}
```

**🟢 async/await 방식**
```swift
func fetchAll() async {
    await withTaskGroup(of: String.self) { group in
        group.addTask { await fetchData1() }
        group.addTask { await fetchData2() }

        for await result in group {
            print("받은 데이터: \(result)")
        }
    }
}
```

##### 4. 우선순위 작업 실행
**🔴 GCD 방식**
```swift
DispatchQueue.global(qos: .background).async {
    // 낮은 우선순위 작업
}
```
**🟢 async/await 방식**
```swift
Task(priority: .background) {
    // 낮은 우선순위 작업
}
```

##### 5. 비동기 작업 취소 처리
**🔴 GCD 방식**
```swift
var isCancelled = false

func cancel() {
    isCancelled = true
}

DispatchQueue.global().async {
    while !isCancelled {
        // 반복 작업
    }
}
```
**🟢 async/await 방식**
```swift
let task = Task {
    try Task.checkCancellation()
    // 작업 수행
}

task.cancel()
```
## Keywords
+ `async/await`
+ `async let`
+ `Task{}`
+ `TaskGroup`
+ `GCD`
+ `Structured Concurrency`

## References
- [Swift Concurrency 공식 문서](https://bbiguduk.gitbook.io/swift/language-guide-1/concurrency)
- [WWDC21 Swift Concurrency: Update a sample app](https://developer.apple.com/kr/videos/play/wwdc2021/10194)
- [WWDC21 Explore structured concurrency in Swift](https://developer.apple.com/kr/videos/play/wwdc2021/10134)
- [WWDC21 Discover concurrency in SwiftUI](https://developer.apple.com/kr/videos/play/wwdc2021/10019)
- [DelightRoom 블로그](https://medium.com/delightroom/swift-concurrency-1%ED%83%84-async-await-task-taskgroup%EA%B0%9C%EB%85%90-%EC%A0%95%EB%A6%AC-2e77e86e7e13)

## 작성자
- Jun