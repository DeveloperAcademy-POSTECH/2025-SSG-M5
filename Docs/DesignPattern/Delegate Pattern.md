>[!question]
>GQ1.  Objective-C 기반 Delegate 구조는 어떤 한계점이 있는가  
>GQ2. Swift의 protocol+extension은 delegate 패턴을 어떻게 안전하고 유연하게 만들었는가

@Sana
## Description
- swift code의 design pattern 중 하나로, Protocol을 통해 객체가 다른 객체에게 작업 책임을 넘기는 것
- ex ) 이벤트 발생! --> "일터졌대 너가 어떻게 좀 해봐 (+프로토콜) " --> "ㅇㅋ 내가 처리함"

==GQ1.  Objective-C 기반 Delegate 구조는 어떤 한계점이 있는가==  
1. **타입 안전성**: type check를 runtime에 하니까 냅다돌리고 냅다 crash 발생 가능
2. **optional method**: 얘도 runtime에 유무를 확인해야해서 비효율
3. **class에서만 사용 가능**: NSObject 반드시 상속해야함
4. **SOLID 1원칙 위반**: SRP(Single Responsibility Principle)
5. **재사용성 쓰레기**

==GQ2. Swift의 protocol+extension은 delegate 패턴을 어떻게 안전하고 유연하게 만들었는가==
1. protocol을 compiler가 체크함
2. value type에서도 delegate pattern 적용 가능
3. extension으로 @objc optional을 대체
4. SOLID 4원칙 준수 > ISP(Interface Segregation Principle)

## 주요 기능
**Delegate Protocol** << delegate가 구현해야 하는 method를 정의
**Delegator** << 이벤트를 발생시키는 주체
**Delegate** << 이벤트에서 로직을 처리하는 객체
**Protocol** << delegator이 delegate에게 요구할 행동 규약

## 코드 예시
```swift
// define protocol
protocol DelegateProtocol: AnyObject{
	func method1()
}

// delegator
class Delegator{
	weak var delegate: DelegateProtocol?

	func doSomething(){
		// 이벤트 감지하고 책임전가
		delegate?.method1()
	}
}

// delegate
class Delegate: DelegateProtocol{
	let delegator = Delegator()

	init(){
		delegator.delegate = self
	}
	
	func method0(){
		delegator.doSomething()
	}
	
	func method1(){
		print("여기서 실제 로직처리가 발생함용")
	}
} 

let test = Delegate()
test.method0()
	
```

## Keywords
+ 파생된 키워드들을 작성
+ [[DesignPattern]]

## References
- https://www.youtube.com/watch?v=qiOKO8ta1n4&pp=ygUWc3dpZnQgRGVsZWdhdGUgUGF0dGVybg%3D%3D
- https://www.youtube.com/watch?v=kF7rQmSRlq0
- 