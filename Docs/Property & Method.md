
> [!NOTE] Question
> GQ1. 속성(Property)과 메서드(Method)는 무엇인가?
>GQ2. 저장 속성(Stored Properties)은 무엇인가?
>GQ3. 지연 저장 속성(Lazy Stored Properties)은 무엇인가?

## 속성(Property)과 메서드(Method)는 무엇인가?

- 구조체(Stuct)와 클래스(Class)는 변수인 속성(Property)와 메서드(Method)를 가짐

- **==속성(Property)) ==** : 
	- 저장 속성(stored), 지연(lazy) 저장 속성
	- 계산 속성(computed)
	- 타입 속성(type)
- **==메서드(Method)) ==** : 
	- 인스턴트 메서드(instance method)
	- 타입 메서드(type method)
	- 서브스크립트(subscripts)
	- 생성자
		- 지정생성자(designated)
		- 편의생성자(convenience) : 상속과 연관
		- 필수생성자(required) : 상속과 연관
		- 실패가능생성자(failable)
		- memberwise생성자 : 저장속성에 관한 생성자
	- 소멸자(deinitialzer)

|                    | 구조체(struct)                                                                                                                             | 클래스(class)                                                                                                                                   |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| 속성<br>(Properties) | • 저장 속성(stored), 지연(lazy) 저장 속성<br>• 계산 속성(computed) : 읽기/쓰기<br>• 타입 속성(type) : 저장/계산 <br>• 속성 감시자(property observers) : willSet/didSet | • 저장 속성(stored), 지연(lazy) 저장 속성 <br>• 계산 속성(computed) : 읽기/쓰기 <br>• 타입 속성(type) : 저장/계산 <br>• 속성 감시자(property observers) :willSet/didSet     |
| 메서드<br>(Methods)   | • 인스턴스 메서드(instance method) <br>• 타입 메서드(type method) <br>• 서브스크립트(subscripts) <br>• 생성자 : memberiwse생성자, 지정생성자, 실패가능생성자                | • 인스턴스 메서드(instance method)<br>• 타입 메서드(type method)<br>• 서브스크립트(subscripts)<br>• 생성자 : 지정생성자, 편의생성자, 필수생성자, 실패가능생성자 <br>• 소멸자(deinitialzer) |




## 저장 속성(Stored Properties)은 무엇인가?

- **==저장 속성(Stored Properties) ==** : 
	- **값이 지연되는 일반적인 속성(변수)을 저장 속성이라고함**
	- 클래스/구조체 틀에서 찍어낸 인스턴스가 가지는 고유의 데이터 저장 공간
	- 변수(var)나 상수(let)으로 선언 가능
	- 객체의 초기화시, 각 저장 속성은 반드시 값을 가져야 함
		- 기본 값을 설정하거나, 생성자에서 설정, 또는 옵셔널 타입으로 선언하여 nill을 초기값으로 갖기 가능)
		- 열거형의 경우 따로 메모리 공간이 필요한 저장 속성(테이터)은 선언할 수 없음
	- 저장 속성은 구고체와 클래스 모두 동일
	- let(상수) 또는 var(변수)로 선언 가능
		- 만약 저장 속성을 let으로 선언하면 값을 바꿀 수 없음
	- 저장 속성(변수)은 각 속성자체가 고유의 메모리 공간을 가짐
	- 초기화 이전 값을 가지고 있거나, 생성자 메서드를 통해 값을 반드시 초기화 해야만 함
```swift
//구조체를 활용해서 저장 속성으로 일기 데이터 보관

struct DiaryEntry {
    // 저장 속성들
    let date: Date             // 일기 작성 날짜 (변경 불가)
    var title: String          // 제목
    var content: String        // 내용
    var mood: String           // 감정 상태 (예: "행복", "우울", "평온")

    // 메서드
    func summary() -> String {
        return "\(title) - \(mood)"
    }
}

// 새 일기 작성
var entry = DiaryEntry(
    date: Date(),
    title: "더위에 산책한 하루",
    content: "산책을 하며 더위를 느꼈다.",
    mood: "덥다"
)

// 저장 속성 값 접근 및 수정
entry.title = "여름 산책"
entry.mood = "개덥다"

// 요약 출력
print(entry.summary())  // 여름 산책 - 개덥다
```


## 지연 저장 속성(Lazy Stored Properties)은 무엇인가?

- **==지연 저장 속성(Lazy Stored Properties) ==** : 
	- **메모리 공간을 나중에 만들어 내는 것을 지연 저장 속성이라고함**
	- 지연 저장 속성은, 선언시점에 기본값을 저장
	- 해당 속성이 반드시 처음부터 초기화가 필요하지 않은 경우,
	  (일반적으로 많은 메모리 공간을 차지하는 이미지 등)에 초기화를 지연시킴
		- 불필요한 성능 저하나 메모리 공간의 낭비를 줄일 수 있음
	- 값에 대한 접근이 있어야 초기화 가능(메모리 공간 생성)
	- lazy var로만 선언 가능(lazy let은 안됩니다!)
	- 생성자에서 초기화하지 않기 때문에 반드시 기본값이 필요
		- 기본값은 표현식의 어떤 형태든 return값만 일치하면 가능
		  (함수 실행문, 계산식, 클로저 실행문 등)

```swift
struct Bird1 {
    var name: String
    lazy var weight: Double = 0.2 //접근하는 순간에 초기값을 갖음
    
    init(name: String) {
        self.name = name
        //self.weight = 0.2
    }
    
    func fly() {
        print("날아갑니다.")
    }
}



var aBird1 = Bird1(name: "새")   // weight 속성 초기화 안됨
aBird1.weight  // ←해당 변수에 접근하는 시점에 초기화 (메모리 공간이 생기고 숫자가 저장됨)

```

- ==저장 속성과 지연 저장 속성의 차이==
	- **저장 속성은 속성인데, 지연(lazy)의 의미는 무엇인가?**
		- 지연 저장 속성은 "해당 저장 속성"의 초기화를 지연시키는 것
		- 인스턴스가 초기화 되는 시점에 해당 속성 값을 갖고 초기화 되는것이 아닌
		  (메모리 공간과 값을 갖는 것이 아닌)
		- 해당 속성(변수)에 접근하는 순간(해당 속성만) 개별적으로 초기화 되는 것
		- 상수(let)으로 선언은 안되고 변수(var)로의 선언만 가능하여 lazy var만 가능
		  (지연 저장 속성은 변화하는 것이기에 lazy var만 가능)
		- 위에 있는 코드의 'weight'이라는 속성은 초기화 시점에 메모리 공간이 생기는 것이 아님
		- 예시로, 인스턴스가 생기고 난 후, aBrid.weight라고 접근하는 순간 메모리 공간을 만들고 숫자를 저장하는 것
	- **지연 저장 속성을 사용하는 이유는 무엇일까?**
		- 메모리 공간의 낭비를 막을 수 있음
			- ex) 메모리 공간을 많이 차지하는 이미지 등의 속성에 저장할때
		- 지연 저장 속성으로 선언된 속성이 다른 저장 속성을 이용할 때
			- 초기화 시점에 모든 속성들이 동시에 메모리 공간에 저장되어 어떤 한가지 속성이 다른 속성에 접글 할수가 없을때, 지연 저장 속성을 사용하면 지연으로 저장된 속성은 먼저 초기화된 속성에 접근 할 수 있음
			- 초기화 시점이 더 늦으므로, 먼저 초기화된 저장 속성 사용 가능함
			- ex) b변수는 초기화 할때, a변수를 활용 가능함(아래 코드 예시 확인)
```swift
class AView {
	var a: Int

	// 1) 메모리를 많이 차지할때
	lazy var view = UIImageView() // 객체를 생성하는 형태
	
	// 2) 다른 속성을 이용해야할때(다른 저장 속성에 의존해야만 할때) 
	lazy var b: Int = { 
		return a * 10 
	}()
	init(num: Int) {
		self.a = num
	}
}
```




## Keywords
- Property
- Method
- Stored Properties
- Lazy Stored Properties