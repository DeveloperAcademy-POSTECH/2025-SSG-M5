제출일: 2025-06-16

>[!question]
>GQ1. Extension과 상속(Inheritance)의 차이점은 무엇인가?
>GQ2. 사용성을 높이기 위해 어떤 extension 을 쓸 수 있나?
>GQ3. Extension을 너무 많이 만들면 어떤 문제가 발생할 수 있는가

## Description

### Extionsion 이란?

다른 일부 언어에도 유사한 기능이 있지만, Swift의 extension은 특히 강력하고 유연하게 설계되어있다.
- 기존 Type 에 기능을 추가한다.
- 기존 class, struct, enum 또는 protocol 타입에 새로운 기능을 추가한다. 기존 소스 코드에 접근 권한이 없는 타입 확장 기능도 포함된다.(retroactive modeling)
- objective-C의 카테고리와 유사하다고 한다.

> [!NOTE]
>   **원본 소스 코드 없이도 타입을 확장**할 수 있다


### Extension 에서 할 수 있는 것
- 계산된 instance property 와 계산된 type property 추가 (저장은 X)
- Instance method와 type method 정의
- 새로운 초기화 구문 제공 (단, 역시 저장 property 추가는 안됨)
- Subscript 정의
- 새로운 중첩 타입 정의 및 사용
- 기존 type이 프로토콜을 준수하도록 함

### Extension과 상속(Inheritance)의 차이점
- Extension과 상속(Inheritance)은 모두 OOP에서 기능을 확장하는 방법으로 사용하지만 각각의 의도, 사용 방식, 설계 철학이 다르다.

| 항목                     | **Extension**                          | **상속 (Inheritance)**                      |
| ---------------------- | -------------------------------------- | ----------------------------------------- |
| **정의**                 | 기존 타입에 새로운 기능을 추가하는 방법 (기능 확장)         | 기존 클래스(부모)를 기반으로 새로운 클래스를 만드는 방법 (기능 재사용) |
| **적용 대상**              | 클래스, 구조체, 열거형, 프로토콜 등 **모든 타입**에 적용 가능 | 클래스만 적용 가능 (struct, enum은 상속 불가)          |
| **구현 방식**              | 타입 외부에서 기능을 **확장**                     | 부모 클래스를 **상속받아** 자식 클래스 생성                |
| **재정의(Override)**      | 기존 기능을 **재정의할 수 없음**                   | 부모 클래스의 메서드, 속성 등을 **재정의 가능**             |
| **다형성 (Polymorphism)** | 지원하지 않음 (컴파일 시점에 결정됨)                  | 지원함 (런타임에서 동적 디스패치 가능)                    |
| **사용 목적**              | 반복되는 기능 추가, 유틸리티 확장, 기능 보완             | 공통된 속성과 메서드를 공유하고 **계층 구조**를 표현           |
| **원본 코드 수정 여부**        | 원본 타입을 **수정하지 않고** 기능 추가 가능            | 부모 클래스의 코드를 **직접 수정**하거나 상속               |
| **상태(state) 추가**       | **저장 프로퍼티 추가 불가** (계산 속성만 가능)          | 저장 프로퍼티 및 메서드 자유롭게 추가 가능                  |
| **접근성 제어**             | 확장 기능은 파일 내 어디든 정의 가능 (같은 접근 수준)       | 상속은 부모의 접근 제어자(private 등)에 영향을 받음         |
| **복잡도 관리**             | 간단한 기능 확장에 적합                          | 복잡한 관계나 행동 모델링에 적합                        |
| **의존성 방향**             | 기존 타입 → extension (단방향)                | 부모 클래스 ← 자식 클래스 (상호 종속)                   |
| **OOP(객체지향) 관점**       | **합성(Composition)** 기반 설계에 가까움         | **계층 구조(Inheritance)** 기반 설계              |

## 주요 기능

### 사용성을 높이기 위해 어떤 extension 을 쓸 수 있나?
반복되는 로직을 줄이고, 가독성과 재사용성을 높이며, 표현력을 향상 시킬 때 쓸 수 있다.
유효성 검사, 날짜나 스트링 포맷 처리, 배열 안전 접근, 유틸리티성 기능, 반복적인 표현 등.
단, 기능이 복잡하거나 상태를 가져갈 때는 오히려 class, struct 로 추출하는 것이 더 적합하다.


###  그럼 어떨 때 extension을 써야하나?

|상황|적절한 선택|
|---|---|
|기능 단순, 상태 없음|`extension`|
|상태 필요, 값 저장 필요|`struct` 또는 `class`|
|기능 복잡, 로직 많음|`class`, `ViewModel`, `Manager` 등 별도 타입|
|객체 간 협업 필요|명확한 책임을 가진 `class` 또는 `protocol` 기반 설계|


### Extension을 너무 많이 만들면 어떤 문제가 발생할 수 있는가

1. 코드 구조의 혼란 (모듈성 저하) - 여러 파일에 흩어져있으면 전체 기능을 한눈에 보기 힘들어져 가독성이 떨어짐.
2. 타입 오염(Type Pollution) - 기본 타입 (String, Array, Date) 등에 너무 많은 extension을 추가하면, 이 타입이 모든 걸 다 처리하는 만능 타입처럼 보여서 역할이 불분명해짐. -> 책임이 명확하지 않은 코드가 됨
3. 네이밍 충돌 / 중복 정의 위험 - 동일한 이름의 메서드나 computed property를 정의할 수 있고, 이 경우 컴파일 에러 또는 예기치 못한 동작이 발생하고 컴파일 에러가 나거나 정의된 순서에 따라 원치 않는 결과가 발생됨
4. 디버깅, 유지보수, 테스트의 어려움 - 실제 타입 내부 코드처럼 보이지 않기 때문에 추적이 어렵다. 의존성이 늘어나면 테스트 격리가 더 어려워진다.

## 코드 예시

### Extension 에서 할 수 있는 것에 대한 예시들
```Swift
// Computed Properties
extension Double {
    var km: Double { self * 1_000 }
    var m: Double { self }
    var cm: Double { self / 100 }
}

let distance = 5.0.km
print(distance) // 5000.0
```

```Swift
// Instance/Type Methods
extension String {
    func reversedString() -> String {
        return String(self.reversed())
    }
    
    static func greeting() -> String {
        return "Hello world!"
    }
}

let name = "TAENI"
print(name.reversedString()) // IAEAT
print(String.greeting())     // Hello world!

```

```Swift
// Initializer
struct Point {
    var x: Double
    var y: Double
}

extension Point {
    init(value: Double) {
        self.x = value
        self.y = value
    }
}

let p = Point(value: 3)
print(p) // Point(x: 3.0, y: 3.0)

```


```Swift
// Subscripts

extension Array {
    subscript(safe index: Int) -> Element? {
        return indices.contains(index) ? self[index] : nil
    }
}

let array = [1, 2, 3]
print(array[safe: 1]) // Optional(2)
print(array[safe: 5]) // nil

// 배열에 안전하게 접근할 수 있도록
```

```Swift
// Nested Types
extension Character {
    enum Kind {
        case vowel, consonant, other
    }

    var kind: Kind {
        switch self.lowercased() {
        case "a", "e", "i", "o", "u": return .vowel
        case "b"..."z": return .consonant
        default: return .other
        }
    }
}

let letter: Character = "e"
print(letter.kind) // vowel

```

```Swift
// 프로토콜 채택

protocol Identifiable {
    var id: String { get }
}

struct User {
    let name: String
}

extension User: Identifiable {
    var id: String {
        return "user:\(name)"
    }
}

let user = User(name: "Taeni")
print(user.id) // user:Taeni

// `Identifiable` 프로토콜을 채택하도록 확장

```



> [!NOTE]
> Extension 은 말그대로 있는 기능에서 '확장'을 할 때만 쓰여야한다. 모듈별로 접두어와 파일 이름을 명확히 하여 구분하도록 하고, 유틸리티성의 기능으로만 확장해야한다.


## Keywords
+ Subscript
+ OOP
+ Override
+ Inheritance
+ [[Extension]]

## References
- https://bbiguduk.gitbook.io/swift/language-guide-1/extensions
- https://docs.swift.org/swift-book/documentation/the-swift-programming-language/extensions/
- with.. ChatGPT

## 작성자
- #TAENI 