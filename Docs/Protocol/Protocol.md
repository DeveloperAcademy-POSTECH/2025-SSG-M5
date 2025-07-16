>[!question]
>GQ1. Swift Protocol은 어떤 역할을 하는가
>GQ2. Protocol 확장은 어떻게 사용되고 상속보다 나은 점은 무엇인가

@Sana
## Description

> A _protocol_ defines a **blueprint** of methods, properties, and other requirements that suit a particular task or piece of functionality. 
> The protocol can then be _adopted_ by a class, structure, or enumeration to provide an **actual implementation** of those requirements. 
> Any type that satisfies the **requirements** of a protocol is said to _conform_ to that protocol. 

- 어떤 class, structure, enumeration에 필적인 **요구사항을 선언**해두는 것. 구현x
- objective-C와 연결해서 selective method 등 기능 사용 가능
## 주요 기능 및 코드 예시
 기본 문법: 선언
```swift
protocol pName {
	var prop1: Int { get set } // get set: 읽고 쓸 수 있다
	var prop2: String { get set }
}
protocol p2Name {
	func method1()
	// mutating func method1()
}
```
기본 문법: 채택
```swift
struct sName : pName, p2Name {
	// structure definition
	var prop1: Int = 1
	var prop2: String = "n"

	//func method1() {
	mutating func method1() { // must be editable
		print(prop2)
	}
}
class cName : maybeSuperclass, pName, p2Name {
	// class definition
	var prop1: Int = 1
	var prop2: String = "n"
	
	func method1() {
		print(prop2)
	}
}
```

objective-C bridging: optional method/property
- class에만 사용 가능함
``` swift
@objc protocol pName {
    @objc optional func method1()
    @objc optional func method2()
}

class ViewController: NSObject, MyDelegate {
    func method1() {
        print("method1 used")
    }
    // method2는 구현하지 않아도 됨 optional이니까
}
```

## Keywords
- [[SOLID]]
+ [[POP]]
+ [[Delegate Pattern]]
+ [[Protocol]]

## References
- https://docs.swift.org/swift-book/documentation/the-swift-programming-language/protocols/