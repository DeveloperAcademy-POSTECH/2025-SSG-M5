>[!question]
>GQ1. Generic은 무엇일까?
>GQ2. Generic은 어떤 경우에 사용하는가?
>GQ3. Generic은 왜 쓰는 것인가?

## Description

##### Generic은 무엇일까?

Swift.org 원문에 따르면 제네릭이란
**Write code that works for multiple types and specify requirements for those types.**
("여러 자료형에서 작동할 수 있는 코드를 작성하되, 그 자료형들이 충족해야 할 조건(요구사항)을 명확히 해라.")

이를 더 쉽게 풀어서 설명해보자면
"타입에 의존하지 않는 유연한 범용 코드를 작성할 때 사용할 수 있으며, 이를 통해 중복을 피할 수 있다" 이다.

애플 원문에 따르면 Swift에서 가장 강력한 기능 중 하나로, 
Swift 표준 라이브러리의 많은 부분이 제네릭으로 작성돼있다고 한다. 

우리가 흔히 사용하는 1) Array 2) Dictionary 타입도 모두 제네릭 컬렉션으로 이루어져 있다.
생각해보면 우리는 어떤 타입의 배열과 딕셔너리를 아무렇지 않게 정의하며 쓰고 있었는데
이는 모두 타입에 제한을 두지 않는 제네릭으로 작성돼있기 때문이다

---

#####  그렇다면, Generic은 어떤 경우에 유용할까?

 아래 두 Int 값을 Swap 하는 함수가 있다고 가정해보자

```swift
func swapTwoInts(_ a: inout Int, _ b: inout Int) { 
	let temporaryA = a 
	a = b 
	b = temporaryA 
}
```

인자로 들어오는 a와 b의 값이 모두 Int인 경우에는 이상없이 a와 b의 숫자가 바뀔 것이다.

그렇다면 String과 Double의 값 교환도 가능한가?
-> Swift는 타입 안전 언어로, **다른 타입 변수 간의 교환을 허용하지 않는다**

따라서, 아래와 String, Double을 인자로 받는 아래 두 함수를 별도로 만들어줘야 한다.

```swift
func swapTwoStrings(_ a: inout String, _ b: inout String) { 
	let temporaryA = a 
	a = b 
	b = temporaryA 
} 

func swapTwoDoubles(_ a: inout Double, _ b: inout Double) { 
	let temporaryA = a 
	a = b 
	b = temporaryA 
}
```

함수 안의 내용은 동일하지만, 인자가 다르다고 해서,
별도로 두 개의 함수를 다시 만들어야하는 번거로움이 있다. 이를 해결하기 위해 "제네릭"을 사용한다

```swift
func swapTwoValues<T>(_ a: inout T, _ b: inout T) { 
	let temporaryA = a 
	a = b 
	b = temporaryA 
}
```

여기서 제네릭은 실제 타입 이름 대신, "플레이스 홀더 타입 이름 (코드에서는 T)"을 사용한다.
플레이스 홀더 (줄여서 T라고 명시), 
T는 어떤 타입 이어야 하는지는 제공하지 않지만 , 뒤 같은 타입이 들어와야 한다는 것을 명시한다.

또 다른 큰 차이점은, 꺽쇠(<>)로 T를 감싼다는 점인데,
	이는 Swift에게 "**T는 새로운 타입이 아니므로 실제 타입을 찾지마!**" 라는 의미이다.

여기서 T가 아닌 다른 문자로 선언해도 문제는 없다

```swift
func exampleGeneric<Banana>(_ a: inout Banana, _b: inout Banana) {
	// 코드생략
}
```

다만, 코드의 가독성을 위해 보통 `T`, `V` 등 단일문자, 혹은 Upper Camel Case를 많이 사용한다.

정리하자면, 제네릭 함수를 사용한다는 것은
- **똑같은 내용의 함수를 오버로딩 할 필요가 없어짐**
- **코드 중복을 피하고 유연하게 코드를 짤 수 있다.**

## 코드 예시
- 타입에 따라 정의하는 Array의 예시
```swift

let intArray: [Int] = [1, 2, 3, 4]
let stringArray: [String] = ["apple", "banana", "cherry"]
let doubleArray: [Double] = [3.14, 2.71, 1.62]

```

- Array 정의 원문
```swift
struct Array<Element> {
    // 내부 구현은 생략
}
```

## Keywords
+ Array
+ Dictionary
+ [[Grammar]]

## References
- https://babbab2.tistory.com/136
- https://docs.swift.org/swift-book/documentation/the-swift-programming-language/generics/#The-Problem-That-Generics-Solve