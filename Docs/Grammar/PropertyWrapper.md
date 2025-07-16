제출일: 2025-06-04

>[!question]
>- Property Wrapper의 특징은 어떤 것이 있을까?
>- 왜 사용하는걸까?
>- 어떤 경우에 주로 사용할까?

## Description
- 클래스나 구조체의 연산 프로퍼티의 동작을 캡슐화하여 코드 재사용성과 가독성을 높인다.
- 쉽게 말해, 프로퍼티에 **특별한 규칙**이나 **추가 기능**을 붙이는 도구입니다.
- 
- Property Wrapper 안에는 **반드시** 값을 접근하거나 지정하는 (=get, set 코드를 포함하는) **wrappedValue 프로퍼티**를 가져야 한다.
- 클래스, 구조체, 열거형에서 사용가능
- 
## 주요 기능
- SwiftUI와 Combine에서 @State, @Published 등으로 UI 상태 관리와 데이터 바인딩하는것도 프로퍼티 래퍼이다.
- @propertyWrapper 속성을 사용해 구조체로 정의하며, wrappedValue를 통해 값을 관리한다.
- 반복적인 로직을 줄여 유지보수를 쉽게 하고, 코드의 간결함을 유지한다.


## GQ1. Property Wrapper의 특징은 어떤 것이 있을까?

- 캡슐화
- 재사용
- 가독성
- projectedValue: $를 이용해서 추가 데이터 처리 가능함

~~~swift
@propertyWrapper
struct Lowercase {
    private var value: String
    
    init(wrappedValue: String) {
        self.value = wrappedValue.lowercased()
    }
    
    var wrappedValue: String {
        get { value }
        set { value = newValue.lowercased() }
    }
    
    var projectedValue: Int {
        value.count
    }
}

struct TextEditor {
    @Lowercase var text: String
}

var editor = TextEditor(text: "HeLLo")
print(editor.text)       // 출력: hello
print(editor.$text)      // 출력: 5
editor.text = "SWIFT"
print(editor.text)       // 출력: swift
~~~

## GQ2. Property Wrapper 왜 사용하는걸까?

>코드의 효율성과 유지보수성을 높이기 위해서

## GQ1. Property Wrapper 어떤 상황에 사용할까?

- **값 검증/제한**: 숫자 범위 제한, 문자열 포맷팅 등.(연산 프로퍼티 추가기능 느낌?)
- **상태 관리**: SwiftUI의 @State, @Binding, @Environment로 UI 상태 동기화.
- **데이터 저장**: @AppStorage로 UserDefaults에 값 저장.
- **변경 추적/로깅**: 값 변경 시 로깅이나 상태 추적.
- **Combine 반응성**: @Published로 데이터 스트림 생성.
- **지연 초기화/캐싱**: 값 계산을 지연시키거나 캐싱.

> 정리하자면 공통된 로직이 많은 상황에서 사용하거나 동작을 단순화하고 싶을때 사용한다고 합니다.
> 그리고 우리가 사용하는 SwiftUI나 Combine에서 주로 사용합니다.

## 코드 예시
~~~ swift
@propertyWrapper 
struct Uppercase { 
	private var value: String

	init(wrappedValue: String) { 
		self.value = wrappedValue.uppercased() 
	}

	var wrappedValue: String { 
		get { value } 
		set { value = newValue.uppercased() } 
	} 
}

struct User { 
	@Uppercase var name: String 
}

var user = User(name: "swift") 
print(user.name) // 출력: SWIFT 

user.name = "hello" 
print(user.name) // 출력: HELLO
~~~

## Keywords
+ Class
+ Struct
+ Enum
+ @**State**, @**Binding**, @**Published**: SwiftUI에서 UI 상태를 관리할 때 사용.
+ **@AppStorage**: UserDefaults에 값을 저장하고 동기화.
- **@Environment**: SwiftUI 환경 값을 주입.
- **@NSCopying**: Objective-C와의 상호작용을 위해 객체를 복사.
-  [[Grammar]]

## References
- https://docs.swift.org/swift-book/documentation/the-swift-programming-language/properties/
- Grok

## 작성자
- #steve