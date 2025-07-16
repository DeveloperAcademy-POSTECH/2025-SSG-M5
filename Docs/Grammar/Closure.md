>[!question]
>GQ1. Closure란 무엇이며, 일반 함수와 어떤 면에서 유사하고 어떤 면에서 다른가?
>GQ2. 클로저가 값을 캡처한다는 것은 어떤 의미일까?
>GQ3. Swift 클로저에는 어떤 종류들이 있을까?
>GQ4. 후행 클로저 문법은 무엇이고, 함수를 호출할 때 이를 사용하면 어떤 이점이 있을까?
>GQ5. 클로저는 언제 쓰일까?
>GQ6. @escaping 키워드는 언제 필요하며, @non-escaping 클로저와 어떤 차이가 있을까?

## Description
### GQ1. Closure 란?
- **클로저**(Closure, 일종의 익명 함수)는 함수처럼 동작하는 **코드 블록**. 즉, 특정 기능을 수행하는 코드를 하나의 블록으로 묶어서 나중에 실행하거나, 변수처럼 전달할 수 있다.
- 함수와 유사하게 매개변수와 반환값을 가질 수 있지만, **이름이 없거나** 필요한 경우에만 이름을 붙여 사용할 수 있다.
- 클로저는 생성될 때 주변의 변수나 상수를 **캡처**(capture)하여 저장한다. 그래서 원래 변수의 범위가 끝나더라도 클로저 내부에서 그 값에 접근하거나 변경할 수 있다.
- 스위프트에서 함수/클로저는 **일급 객체**(First-class citizen)이므로, 변수에 할당하거나 다른 함수의 인자/반환값으로 사용할 수 있다. 이 특성 덕분에 클로저를 활용하여 유연하고 재사용 가능한 코드를 작성할 수 있다.
```swift
// 우리가 아는 기본 함수 예시
func add(a: Int, b: Int) -> Int {
    return a + b
}

// 클로저 예시
let addClosure: (Int, Int) -> Int = { (a, b) in
    return a + b
}

print(add(a: 2, b: 3)) // 출력: 5
print(addClosure(2, 3)) // 출력: 5
```

### GQ2.  클로저가 값을 캡처한다는 것은 어떤 의미일까?
 클로저 sayHello는 외부 변수 `name`을 사용하므로 해당 값을 캡처한다. 
 
 `name`을 나중에 "Henry"로 변경한 후 클로저를 실행했지만, **클로저가 캡처한 변수의 최신 값**을 참조하기 때문에 "Hello, Henry!"가 출력된다.  
```swift
// 값 캡처(capture) 예시
var name = "Jun"

let sayHello = { 
    print("Hello, \(name)!") 
}  // 클로저가 name 변수를 캡처함

name = "Henry" 
sayHello()  // Hello, Henry!
```

### GQ3. Swift 클로저에는 어떤 종류들이 있을까?
 **1. 전역 함수 (Global Functions)**
- **전역 함수**는 이름이 있는 클로저로, 특정 함수 내부가 아닌 전역 범위에 정의된 함수이다. 사실상 Swift 함수는 클로저의 한 종류임
```swift
func globalFunction(name: String) {
    print("안녕하세요, \(name)!")
}

globalFunction(name: "준혁")  // 출력: 안녕하세요, 준혁!
```

 **2. 중첩 함수 (Nested Functions)**
- **중첩 함수**는 함수 내부에 정의된 함수이며, 자신을 둘러싼 외부 함수의 변수나 상수에 접근할 수 있다. 
```swift
func outerFunction() {
    var number = 5
    func innerFunction() {
        number += 1  // 외부 변수 number를 캡처하여 사용
        print("내부 함수에서 number: \(number)")
    }
    innerFunction()
    print("outerFunction 종료 후 number: \(number)")
}

outerFunction()
// 내부 함수에서 number: 6
// outerFunction 종료 후 number: 6
```
-  innerFunction이 number를 1 증가시킨 후 출력하고, outerFunction 종료 후에도 number 값이 6으로 증가. 중첩 함수 innerFunction이 number를 캡처(capture)하여 내부에서 수정했기 때문에, 외부 변수의 변화가 지속된 것이다.

**3. 클로저 표현식 (Closure Expressions)**
- **클로저 표현식**은 **이름 없이 직접 작성되는 클로저**를 말한다.  {}로 둘러싸여 있고, 내부에 매개변수 목록과 실행 코드를 포함한다. 이러한 표현식을 변수나 상수에 할당하거나 함수의 인자로 전달하여 사용한다.
```swift
let greet: (String) -> Void = { (name: String) in
    print("Hello, \(name)!")
}

greet("Swift")  // 출력: Hello, Swift!
```

### GQ4. 후행 클로저 문법은 무엇이고, 함수를 호출할 때 이를 사용하면 어떤 이점이 있을까?
**후행 클로저** 문법은 함수 호출 시 마지막 인자로 전달되는 클로저를 특별한 형태로 작성하는 방법.
함수의 괄호를 닫은 후에 클로저를 작성함으로써, 코드의 가독성을 높이고 중첩 괄호를 줄여준다. 
```swift
func repeatTask(times: Int, task: () -> Void) {
    for _ in 1...times {
        task()
    }
}

// 일반적인 클로저 전달 방식 (괄호 안에 클로저 작성)
repeatTask(times: 3, task: {
    print("Hello")
})

// 후행 클로저를 사용한 방식 (괄호 밖에 클로저 작성)
repeatTask(times: 3) {
    print("Hello")
}
```

- 후행 클로저를 사용하면 함수의 마지막 인자 이름 `(task:)` 을 생략하고, 바로 클로저 코드 작성 가능
- 코드가 더 읽기 쉬워지며 클로저의 코드가 길어질수록 장점 부각!

### **GQ5. 클로저는 언제 쓰일까?**
- **반복 동작을 간결하게 표현:** 고차함수에서 클로저를 자주 사용한다. 
  **map**(각 요소를 변환)이나 **filter**(조건에 따라 요소 추출) 함수에 클로저를 전달하여 배열의 데이터를 손쉽게 변형 가능

```swift
let numbers = [10, 3, 20, 7, 15]

let bigNumbers = numbers.filter { (num: Int) in 
    return num > 10 
}
print("10보다 큰 수: \(bigNumbers)")  
// 10보다 큰 수: [20, 15]

let squaredNumbers = numbers.map { (num: Int) in 
    return num * num 
}
print("제곱한 결과: \(squaredNumbers)")  
// 제곱한 결과: [100, 9, 400, 49, 225]
```
   
- **비동기 작업의 완료 처리:** 네트워크 요청이나 시간 지연 작업을 수행한 뒤 결과를 처리해야 할 때, **콜백 함수**를 클로저로 전달한다. 
  예) completion handler
```swift
func fetchData(completion: () -> Void) {
    print("데이터를 불러오는 중...")
    completion()
}

fetchData {
    print("UI 업데이트")
}
```
    
- **이벤트 처리와 UI 업데이트:** 버튼을 눌렀을 때 실행되는 코드, 애니메이션이 끝난 후 실행할 코드 등을 클로저로 작성하여, 특정 이벤트에 대한 동작을 간편하게 정의할 수 있다.
```swift
let buttonClickHandler = {
    print("버튼 눌림!")
}
```

### **GQ6. @escaping 키워드는 언제 필요하며, @non-escaping 클로저와 어떤 차이가 있을까?
Swift에서 함수의 인자로 전달되는 클로저는 기본적으로 **non-escaping**이다. 
즉, 클로저는 함수가 실행되는 동안에만 사용되고, 함수의 실행이 끝나면 메모리에서 해제된다. 
그러나 함수가 끝난 후 **클로저가 저장되거나 나중에 호출되는 경우**, 해당 클로저는 함수의 스코프를 "탈출"하게 되므로 반드시 `@escaping` 키워드를 붙여야 한다.

- **non-escaping**: 지금 당장 실행될 클로저 → 함수 안에서 끝남
- **@escaping**: 나중에 실행될 클로저 → 함수 밖에서 실행될 수 있음

```swift
func greetNow(message: String, action: () -> Void) {
    print("\(message)...")
    action()  // 바로 실행됨
}

greetNow(message: "Hello") {
    print("반가워요!")
}
// Hello...
// 반가워요!
```


```swift
var savedAction: (() -> Void)?

func greetLater(action: @escaping () -> Void) {
    print("클로저 저장 중...")
    savedAction = action  // 나중에 실행할 클로저 저장
}

greetLater {
    print("나중에 실행됐어요!")
}

print("잠시 후 실행!")
savedAction?()  // 여기서 실행됨

// 클로저 저장 중...
// 잠시 후 실행!
// 나중에 실행됐어요!
```


단 .@escaping 클로저는 함수가 끝난 후에도 실행되기 때문에, 해당 클로저 안에서 `self`를 직접 참조하면 강한 참조 순환(Retain Cycle)이 발생할 수 있다. → 메모리 누수

## Keywords
- 일급객체
- 고차함수
- Completion Handler
- 비동기 작업 (Async)
- escaping/nonescaping 클로저
- 순환참조
- [[Grammar]]

# References
- [Apple 공식 Swift 문서 - Closure](https://bbiguduk.gitbook.io/swift/language-guide-1/closures)
- [applebuddy 블로그](https://0urtrees.tistory.com/133)
- [beepeach 블로그](https://beepeach.tistory.com/14)
- [개발하는 정대리 Youtube ](https://www.youtube.com/watch?v=mtVoy_jRlYM&t=151s)