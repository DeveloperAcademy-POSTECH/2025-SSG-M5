제출일: 2025-06-25 (수)

>[!question]
>GQ1. `class`와 `struct`의 차이를 알기 위해서 사전에 알아야 할 지식은 뭐가 있을까?
>GQ2. `class`와 `struct`의 공통점과 차이점은 뭘까?
>GQ3. `class` 대신 `struct`를 더 빈번하게 사용하는 이유가 뭘까?
>GQ4. `class`와 `struct`를 각각 어느 상황에서 사용하는게 좋을까?

## GQ1. `class`와 `struct` 차이를 알기 위해서 사전에 알아야 할 지식은 뭐가 있을까?

1. 값 타입(Value Type) vs 참조 타입(Reference Type)
2. 메모리 저장 방식: Stack vs Heap
3. 상속(Inheritance)과 다형성(Polymorphism)이란 무엇인가?
4. 메모리 관리(ARC: Automatic Reference Counting)
5. 스레드 안전성(Thread Safety)

## GQ2. `class`와 `struct`의 공통점과 차이점은 뭘까?

### 공통점

- **프로퍼티 / 메서드 정의**
    - 저장 프로퍼티 (Stored Property)
    - 계산 프로퍼티 (Computed Property)
    - 타입 프로퍼티 (Type Property, static)
    - 인스턴스 메서드 / 타입 메서드 (static func)
- 생성자(Initializer) 정의
    - 커스텀으로도 구현 가능
    - 매개변수도 모두 지원
- **프로토콜 채택**
    - `Protocol`을 통해 기능 확장 가능.
- **확장성**: `extension`으로 기능 추가 가능.

### 차이점

| 특성           | `struct` (값 타입)                 | `class` (참조 타입)                        |
| ------------ | ------------------------------- | -------------------------------------- |
| **메모리 할당**   | Stack (빠른 할당/해제)                | Heap (동적 할당, ARC 관리 필요)                |
| **참조 동작**    | 복사 시 독립 인스턴스 생성                 | 복사 시 동일 인스턴스 참조                        |
| **스레드 안전성**  | 복사본 독립적 → **안전**                | 공유 자원 → 동기화 필수 (`DispatchQueue`)       |
| **상속**       | 불가능                             | 가능                                     |
| **메서드 디스패치** | 정적 디스패치 (컴파일 타임)<br>→ 상속 불가능하니까 | 동적 디스패치 (런타임 vtable 탐색)<br>→ 상속이 가능하니까 |
| **ARC 영향**   | 미적용 (값 복사만)                     | 적용 (참조 카운트 관리)                         |

- **정적 디스패치(Static Dispatch)란? (Perplexity)
    
    - 언제 결정?
        → 컴파일 타임(코드 작성 후 빌드할 때)
        
    - 어떻게 동작?
        → 컴파일러가 “이 메서드는 무조건 이 함수”라고 미리 결정해서, 직접 함수 주소를 코드에 박아 넣음
        
    - 어디서 사용?
        → struct, enum, final class, static 메서드 등
        
    - 장점
        - 빠름 (런타임에 검사/탐색 과정 없음)
        - 인라인 최적화 등 추가적인 성능 개선 가능
          
    - 비유
        → 네비 없이 사전에 정해진 길로 바로 출발! (경로 변동 없음)
        
- **동적 디스패치(Dynamic Dispatch)란? (Perplexity)
    
    - 언제 결정?
        → 런타임(프로그램 실행 중 실제로 호출할 때)
        
    - 어떻게 동작?
        → 실행 시점에 “이 객체의 실제 타입이 뭔지”를 확인해서, 그에 맞는 메서드를 찾아 호출 
        (클래스의 경우 vtable, 프로토콜의 경우 witness table 사용)
        
    - 어디서 사용?
        → class, 프로토콜 타입, 상속/오버라이딩, @objc 등
        
    - 장점
        - 다형성(Polymorphism) 지원: 같은 코드로 다양한 동작 가능
          
    - 단점
        - 느림 (런타임에 메서드 탐색 오버헤드 발생)
          
    - 비유
        → 네비 키고 목적지에 따라서 경로를 항상 새로 탐색!
        
- 각각 예제
    
    - 정적
        ```swift
        final class Circle {
            func draw() {
                print("원을 그립니다")
            }
        }
        
        let circle = Circle()
        circle.draw() // 컴파일 타임에 어떤 메서드가 호출될지 확정
        ```
        
    - 동적
        ```swift
        class Animal {
            func makeSound() {
                print("동물이 소리를 냅니다")
            }
        }
        
        class Dog: Animal {
            override func makeSound() {
                print("멍멍")
            }
        }
        
        class Cat: Animal {
            override func makeSound() {
                print("야옹")
            }
        }
        
        let animals: [Animal] = [Animal(), Dog(), Cat()]
        
        for animal in animals {
            animal.makeSound() // 실행 시점에 실제 타입에 따라 결정됨
        }
        ```
        

## GQ3. `class` 대신 `struct`를 더 빈번하게 사용하는 이유가 뭘까?

### 1. 안전하고 예측 가능함 → Value Type

- **struct는 값 타입**이어서 변수에 할당하거나 함수에 전달할 때, 값 자체가 복사된다. → 각 인스턴스가 독립적으로 동작하고, 한 곳의 변경이 다른 곳에 영향을 주지 않음.
- **class는 참조 타입**이기 때문에 여러 변수나 상수에서 같은 인스턴스를 공유할 수 있어, 예기치 못한 부작용이 발생할 수 있음

### 2. 메모리 관리 및 성능

- **struct는 스택(Stack)에 저장**되어 할당과 해제가 매우 빠르고, 각 스레드가 독립적으로 관리하므로 thread-safe하다.
- **class는 힙(Heap)에 저장**되고, **ARC**로 메모리를 관리해야 하며, 할당/해제 및 참조 관리에 오버헤드가 발생할 수 있어 유의해야 한다.

### 3. 스레드 안전(Thread Safety)

- 스택에 저장되는 struct는 각 스레드가 독립적으로 관리하므로, 동시성 이슈 없이 안전하게 사용할 수 있다.
- class는 여러 곳에서 같은 인스턴스를 참조할 수 있어, 동시 접근 시 동기화 처리가 필요하다.

### 4. 상속이 필요 없는 경우가 대부분

- struct는 상속이 불가능하지만, Swift에서는 상속이 꼭 필요한 상황이 많지 않다.
- 코드의 재사용과 확장은 **프로토콜과 익스텐션**으로 충분히 구현할 수 있다. → 굳이 클래스?

### 5. Swift 공식 권장 사항

- Swift 공식 문서와 Apple의 가이드라인에서는 "**기본적으로 struct를 사용**하고, 상속이나 참조 공유가 필요한 경우에만 class를 사용"할 것을 권장한다.
    
	![[2025-06-25_08-57-08.png]]
    
- Swift 표준 라이브러리의 Array, Dictionary, String 등도 모두 struct로 구현되어 있다.
    

## GQ4. `class`와 `struct`를 각각 어느 상황에서 사용하는게 좋을까?

### struct를 사용하기 좋은 상황 및 용도

1. 불변 데이터 모델, 단순 데이터 그룹핑
    - 예: 사용자 정보(User), 상품(Product), 위치(Point), 날짜(Date) 등
    - 값이 복사되어도 상관없고, 각 인스턴스가 독립적으로 동작해야 할 때
2. Swift 표준 라이브러리 타입처럼 값 복사가 자연스러운 경우
    - 예: Array, Dictionary, String, Int, Double 등
    - 데이터 전달 시 복사되어도 문제 없고, 데이터의 변경이 다른 곳에 영향을 주면 안 될 때
3. 작고 경량의 데이터 타입
    - 예: 좌표(CGPoint), 크기(CGSize), 사각형(CGRect), RGB 색상 등
    - 스택에 저장되어 빠른 할당/해제와 성능이 중요할 때
4. 불변성, 스레드 안전이 중요한 경우
    - 여러 스레드에서 동시에 사용해도 안전해야 할 때
    - 값 복사 특성 덕분에 동시성 이슈가 없음
5. 상속이 필요 없는 경우
    - 데이터 모델, DTO, ViewModel 등
    - SwiftUI의 View, Identifiable 등 대부분 struct로 설계

### class를 사용하기 좋은 상황 및 용도

1. 상속(다형성)이 필요한 경우
    - 예: UIKit의 UIView, UIViewController, 커스텀 UI 컴포넌트 등
    - 기존 클래스를 확장하거나, 여러 타입이 공통 인터페이스를 공유해야 할 때
2. 참조 공유가 필요한 경우
    - 여러 객체가 같은 인스턴스를 참조하고, 한 곳에서 변경하면 모든 곳에 반영되어야 할 때
    - 예: 싱글톤 패턴(네트워크 매니저, 오디오 플레이어), 앱 전체에서 공유하는 상태 객체 등
3. 객체의 정체성(Identity)이 중요한 경우
    - 두 객체가 같은 데이터라도 “동일 인스턴스인지”가 중요한 상황
    - 예: 데이터베이스 연결, 네트워크 세션, 리소스 핸들러 등
4. Objective-C와의 상호 운용이 필요한 경우
    - @objc, NSObject 기반 API와 연동해야 할 때
    - 예: UIKit, AppDelegate, NotificationCenter 등
5. 리소스 관리, 해제(deinit)가 필요한 경우
    - 객체가 해제될 때 리소스 정리 작업이 필요한 경우
    - 예: 파일 핸들, 네트워크 커넥션, 메모리 캐시 등

## GQ+. 재미로 AI에게 물어본 struct, class 성능 벤치마크

### 1. 벤치마크 코드 예시

아래는 struct와 class 각각 10개의 Int 프로퍼티를 가진 타입을 1,000만 번 더하는 연산을 측정하는 코드이다.

```swift
// Struct 버전
struct Int10Struct {
    var a, b, c, d, e, f, g, h, i, j: Int
    init(_ value: Int) {
        a = value; b = value; c = value; d = value; e = value
        f = value; g = value; h = value; i = value; j = value
    }
    static func + (lhs: Int10Struct, rhs: Int10Struct) -> Int10Struct {
        return Int10Struct(lhs.a + rhs.a)
    }
}

// Class 버전
class Int10Class {
    var a, b, c, d, e, f, g, h, i, j: Int
    init(_ value: Int) {
        a = value; b = value; c = value; d = value; e = value
        f = value; g = value; h = value; i = value; j = value
    }
    static func + (lhs: Int10Class, rhs: Int10Class) -> Int10Class {
        return Int10Class(lhs.a + rhs.a)
    }
}

// 벤치마크 함수
func measure(name: String, block: () -> ()) {
    let t0 = CACurrentMediaTime()
    block()
    let dt = CACurrentMediaTime() - t0
    print("\\\\(name) -> \\\\(dt)")
}

// 실행 예시
measure(name: "struct (10 fields)") {
    var y = Int10Struct(0)
    for _ in 1...10_000_000 {
        y = y + Int10Struct(1)
    }
}

measure(name: "class (10 fields)") {
    var x = Int10Class(0)
    for _ in 1...10_000_000 {
        x = x + Int10Class(1)
    }
}

```

---

## 2. 실제 벤치마크 결과 (실제 기기, Release 빌드 기준)

- struct 버전: 약 0.0000000417초 (약 4.17e-08초)
- class 버전: 약 2.06초

**struct가 class보다 수십만~수백만 배 빠른 결과**가 나옴.

---

## 3. 성능 차이의 원인 분석

- **메모리 할당 방식**
    - struct는 스택에 할당되어 할당/해제가 매우 빠름.
    - class는 힙에 할당되어 동적 메모리 탐색, ARC(참조 카운트), 동기화 등 오버헤드가 큼.
- **메서드 디스패치**
    - struct는 정적 디스패치(컴파일 타임에 함수 결정)로 직접 호출되어 빠름
    - class는 동적 디스패치(런타임에 vtable 탐색)로 약간의 오버헤드 발생
- **ARC 오버헤드**
    - class는 인스턴스 참조가 변경될 때마다 ARC 연산이 발생해 추가 비용이 듦
    - struct는 값 복사만 일어나므로 ARC가 적용되지 않음.

---

## 4. 실무 적용 팁 및 주의사항

- struct는 값 타입이므로, 복사 비용이 데이터 크기에 따라 커질 수 있음. 하지만 Swift의 Copy-on-Write 최적화 덕분에, 실제로는 대부분의 경우 struct가 훨씬 빠름
- class가 반드시 필요한 상황(상속, 참조 공유, Objective-C 연동 등)이 아니라면 struct 사용이 성능과 안정성 모두에서 유리함
- 실제 앱에서는 struct와 class의 성능 차이가 눈에 띄지 않을 수도 있으니, **핫스팟(성능 병목) 구간에서는 직접 프로파일링**하는 것이 중요하다

---

## 요약

|항목|struct|class|
|---|---|---|
|메모리 할당|스택(빠름)|힙(느림, 오버헤드 有)|
|메서드 호출|정적 디스패치(빠름)|동적 디스패치(느림)|
|ARC 적용|X|O|
|복사 비용|작음(값 복사)|큼(참조+ARC)|
|벤치마크 결과|수십만~수백만 배 빠름|느림|

# 추가 생각나는 GQ

### 1. Swift의 Copy-on-Write 최적화란 뭘까?
### 2. 내 생각이 너무 struct 쪽으로 편향되어있는 건 아닐까?
읽어볼 만한 아티클
- https://showcove.medium.com/swift-struct-vs-class-1-68cf9cbf87ca

## Keywords
+ [[Grammar]]

## References
- [https://developer.apple.com/documentation/swift/choosing-between-structures-and-classes](https://developer.apple.com/documentation/swift/choosing-between-structures-and-classes)
- https://stackoverflow.com/questions/24232799/why-choose-struct-over-class
- https://hasensprung.tistory.com/181


## 작성자
- Wade (@hgkim215)