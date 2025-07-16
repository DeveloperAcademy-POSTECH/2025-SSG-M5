

> [!NOTE] Question
> GQ1. Method Dispatch란 무엇인가?
>GQ2. Method Dispatch 종류

## Method Dispatch란 무엇인가?

- **==Method(메서드) ==** : 
	1. 메서드는 CPU에 대한 명령어 (코드 영역에만 존재)
	2. 어떤 객체(Object)가 수행할 수 있는 동작 또는 기능을 정의한 코드 블록
- **==Method Dispatch(메서드) ==** : 
	1. 어떤 메서드를 실제로 실행할지를 러타임에 결정하는 방식
	2. 코드 영역에 저장되어 있는 메서드를 실행하기 위해서 메서드의 주소가 필요함
	3. 코드가 컴파일되거나 실행될 때, 컴퓨터가 어떤 메서드를 호출할지 결정하고
	   그 메서드의 메모리 주소를 찾아 호출하는 과정



## Method Dispatch 종류

- **==Static Dispatch(정적 디스패치) ==** : 
	1. 컴파일 타임에 결정되는 모든 디스패치 방식을 통칭
	2. 컴파일 시점에 어떤 메서드가 실행될지 결정
	3. 빠르지만 오버라이딩 불가능
	4. 사용 예시) `struct` , `final class` , `static func` , `private func`
- Direct Dispatch(직접 디스패치) :
	1. Static의 하위 디스패치
	2. 함수 주소를 직접 호출
	3. 컴파일 시점에 코드 자체에 함수의 메모리 주소 삽입 또는 함수의 명령 코드를 해당 위치에 코드를 심음(in-line)
	4. 가장 빠름(0.0 ~ 2.13ns)
	5. 사용 예시) struct 메서드


- **Static Dispatch 사용 예시별 특징**
	- struct(구조체) : 값(Value) 타입으로, 데이터를 묶고 기능을 정의하는 데 쓰이는 타입
	- final class(파이널 클래스) : 이 클래스를 기반으로 다른 클래스 만들 수 없음
	- static func(정적 함수) : 타입 자체에 속한 함수, 인스턴스를 만들지 않고도 호출 가능
	- private func(비공개 함수) : 같은 파일이나 타입 내부에서만 사용 가능 함수, 외부에서 호출 불가

| 항목                   | 설명                | 오버라이딩 | 사용 맥락          |
| -------------------- | ----------------- | ----- | -------------- |
| **struct(구조체)**          | 값(value) 타입 상속 불가 | X     | 데이터 모델, View 등 |
| **final class(파이널 클래스)** | 클래스지만 상속 금지       | X     | 변경 불필요한 클래스    |
| **static func(정적 함수)**   | 타입에 직접 속한 함수      | X     | 유틸성 함수         |
| **private func(비공개 함수)** | 외부 접근 차단 함수       | X     | 내부 전용 기능 처리    |

- **Static Dispatch : struct - 값 타입이라 상속 불가 > 오버라이딩 불가**
```swift
struct Greeter {
    let name: String

    func sayHello() {
        print("안녕하세요, \(name)님!")
    }
}

let greeter = Greeter(name: "애플")
greeter.sayHello()  // Static Dispatch
```
 
- **Static Dispatch : final class - 상속 금지 > 오버라이딩 불가 : 상속 금지로 인해 고정된 구현**
```swift
final class Logger {
    func log(_ message: String) {
        print("[로그]: \(message)")
    }
}

let logger = Logger()
logger.log("앱 실행됨")  // Static Dispatch
```

- **Static Dispatch : static func - 타입 자체에 붙은 함수 > 오버라이딩 불가**
```swift
struct Math {
    static func square(_ x: Int) -> Int {
        return x * x
    }
}

let result = Math.square(4)
print("제곱 결과: \(result)")  // 제곱 결과: 16 (Static Dispatch)

```

- **Static Dispatch : private func - 외부 접근 & 오버라이딩 불가** 
```swift
class Authenticator {
    func login() {
        checkCredentials()  // Static Dispatch
        print("로그인 성공")
    }

    private func checkCredentials() {
        print("자격 확인 중...")
    }
}

let auth = Authenticator()
auth.login()

```


 - **==Dynamic Dispatch(동적 디스패치) ==** : 
	1. 런타임 시점에 어떤 메서드를 호출할지 결정
	2. 오버라이딩 지원
	3. 느리지만 유연함
	4. 사용 예시) `class`의 인스턴스 메서드
	
	**💡 Dynamic Dispatch가 필요한 이유 :**
	- 클래스를 상속하거나, 프로토콜을 채택했을때,
	- 같은 이름의 메서드를 여러 타입이 다르게 구현할 수 있음.
	- 이때, 어떤 메서드를 불러야 할지를 런타임에 판단하는것이 Dynamic Dispatch!
```swift
class Animal {
    func sound() {
        print("동물 소리")
    }
}

class Dog: Animal {
    override func sound() {
        print("멍멍")
    }
}

let pet: Animal = Dog()
pet.sound() // 어떤 메서드 실행될까요? Dog,sound()가 호출 됩니다

```
- pet은 Animal 타입처럼 보이지만, 실제는 Dog 객체
- 실행시 Dog.sound()가 호출 되어지며, 이것이 Dynamic Dispatch
	- 이 객체의 진짜 타입이 무엇인지 확인하고
	- 그 타입이 sound()를 어떻게 구현했는지
	- 그 메서드 주소를 찾아서
	- 거기로 점프해서 실행한 것


- **==Witness Table Dispatch(프로토콜 디스패치) ==** : 
	1. 프로토콜을 따를 때 사용
	2. 런타임에 프로토콜의 실제 구현을 테이블에서 참조
	
	**💡Witness Table Dispatch가 필요한 이유 :**
	- 프로토콜을 통한 다형성을 지원하기 위해 반드시 필요함
	- Swift는 구조체, 열거형처럼 상속이 불가능한 타입도 공통된 인터페이스로 사용하게 하기 위해 사용
```swift
protocol Swimmable {
    func swim()
}

struct Fish: Swimmable {
    func swim() { print("🐟 헤엄친다") }
}

struct Human: Swimmable {
    func swim() { print("🏊‍♂️ 수영한다") }
}

```
```swift
func makeThemSwim(_ swimmer: Swimmable) {
    swimmer.swim()
}
```
- swimmer는 Swimmable이라는 추상 타입
- 실제 구현이 Fish인지, Human인지 모름
- 하지만 swim()은 타입마다 다르게 구현되어 있음
- 이걸 런타임에 알아내서 호출하는 것이 Witness Table Dispatch

⭐️Witness Table이란 : Fish가 Swimmable을 어떻게 구현했는지를 저장한 테이블
- Swift는 Fish, Human 각각이 Swimmable의 swim()을 어떻게 구현했는지 미리 테이블에 기록하고
- Swimmable 타입으로 호출할 때 그 테이블을 참조해서 실행함

## Keywords
- Method Dispatch
- Dispatch
- [[Dispatch]]