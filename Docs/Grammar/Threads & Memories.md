>[!question]
>GQ1. GQ를 쓰세요
>GQ2. GQ를 쓰세요


Swift의 메모리 관리와 스레드 동작 원리를 이해하는 것은 `class`와 `struct` 차이를 파악하기 위한 **필수 기반 지식**입니다. 실무에서 자주 접하는 핵심 개념을 단계별로 설명합니다.

---

# 1. 메모리 관리: **ARC (Automatic Reference Counting)**

Swift는 클래스 인스턴스의 메모리를 관리하기 위해 **ARC**를 사용합니다.

**동작 원리**:

- **참조 카운트 추적**: 객체가 참조될 때마다 카운트 증가, 참조 해제 시 감소.
- **카운트 0 도달 시 메모리 자동 해제**.

```swift
class MyClass { }
var obj1: MyClass? = MyClass()  // 참조 카운트 1
var obj2 = obj1                 // 참조 카운트 2
obj1 = nil                      // 참조 카운트 1
obj2 = nil                      // 참조 카운트 0 → 메모리 해제

```

**주의 사항**:

- **순환 참조**: 클래스 간 강한 참조로 인한 메모리 누수 → `weak`, `unowned`로 해결.
- **구조체(struct)**: 값 타입이므로 `ARC 미적용` → 복사 시 독립 인스턴스 생성.

---

# 2. 스레드 동작 원리

### 가. **Main Thread vs Background Thread**

- **Main Thread**: UI 업데이트 **반드시** 메인 스레드에서 처리.
- **Background Thread**: 네트워크/파일 I/O 등 무거운 작업은 백그라운드로 분리.

```swift
// GCD 예시
DispatchQueue.global(qos: .background).async {
    // 백그라운드 작업
    let data = fetchData()

    DispatchQueue.main.async {
        // UI 업데이트는 메인 스레드로 복귀
        updateUI(data)
    }
}

```

### 나. **Swift Concurrency (async/await)**

iOS 15+ 도입된 현대적 동시성 모델:

- **비동기 작업 간소화**: `async`/`await`로 콜백 지옥 해결[4].
- **Structured Concurrency**: 작업 계층 구조 관리.

```swift
func fetchData() async -> Data {
    let data = await networkRequest()
    return data
}

```

### 다. **스레드 안전성 (Thread Safety)**

- **값 타입(struct)**: 복사 시 독립적 → **스레드 세이프**.
- **참조 타입(class)**: 공유 자원 접근 시 `DispatchQueue`/`actor`로 동기화 필요.

```swift
// Actor 사용 예시 (Swift 5.5+)
actor Counter {
    private var value = 0

    func increment() {
        value += 1
    }
}
```

---

# 3. 메모리 할당: **Stack vs Heap**

|특성|Stack|Heap|
|---|---|---|
|**할당 속도**|빠름 (포인터 이동만 필요)|느림 (동적 메모리 탐색/잠금)|
|**생명 주기**|함수 종료 시 자동 해제|참조 카운트 관리 필요|
|**사용 사례**|구조체, 기본 타입|클래스, 클로저|

**예시**:

```swift
struct Point { var x, y: Double }  // Stack 할당
class ImageLoader { }              // Heap 할당 + 참조 카운트 관리

```

---

# 4. 실무 팁

1. **구조체 우선 사용**: 작은 데이터/독립적 상태 → 스택 할당으로 성능 향상.
2. **클래스 사용 시점**:
    - 상속/다형성 필요 시.
    - 공유 자원 관리 시 `actor`/`DispatchQueue`로 동기화.
3. **ARC 최적화**:
    - `weak`/`unowned`로 순환 참조 방지.
    - `deinit`에서 리소스 정리 명시적 구현.

---

이해해야 할 **핵심 포인트**:

✅ **ARC는 클래스 전용 메커니즘**

✅ **값 타입은 스레드 세이프, 참조 타입은 동기화 필수**

✅ **메모리 할당 방식 차이가 성능에 직결**

## Keywords
+ [[Grammar]]