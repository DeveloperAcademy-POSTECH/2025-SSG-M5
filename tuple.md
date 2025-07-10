
> [!NOTE] Question
> GQ1. tuple이란 무엇인가?
>GQ2. tuple을 쉽게 사용하는 방법
>GQ3. tuple은 어디에 쓸까?


## tuple이란 무엇인가?

- **==tuple(튜플) ==** : 
	1. **2개 이상의 연관된 데이터(값)를 저장**하는 Compound(복합/홉합) 타입
	2. 원하는 연관된 데이터 조합으로 어떤 형태든 만들 수 있는 타입
	3. **개발자가 원하는 형태로 만들 수 있음**
	4. 변수 선언과 동시에 해당 멤버(데이터 종류 및 갯수)는 결정되므로 추가/삭제 불가)
	5. ex) 애플(string), 나이 20세(Int), 사는지역 포항(String) > (홍길동, 20, 포항)
```swift
let twoNumbers: (Int, Int) = (1, 2)
type(of: townumbers)
//type(of: )는 변수의 타입을 알수 있는 것 (Int, Int).type

let threeNumbers: (Int, Int, Int) = (1, 2, 5)
type(of: threeNumbers)
//(Int, Int,Int)는 자동 추론이 되기 때문에 굳이 쓰지 않아도 괜찮음

var threeValues = ("애플", 20, "포항")
type(of: threeValues)

threeValues = ("사과", 30, "서울")
//var로 하면 새롭게 데이터를 변경할 수 있음
//하지만 데이터 형태가 변하면 에러가 나서 똑같은 형태로 데이터 받아야함
```

- ==**연관된 값(tuple)의 각각 데이터 접근 :==**
	1. 순서는 0번째 부터 메겨야 함, 우리는 개발자니까요.....
	2. 변수명.0 → 첫번째 데이터
	3. 변수명.1 → 두번째 데이터
	4. 변수명.2 → 세번째 데이터
```swift
var threeValues = ("애플", 20, "포항")

threeValues.0 //애플
threeValues.1 //20
threeValues.2 //포항
```




## tuple을 쉽게 사용하는 방법

- ==**이름이 매겨진 튜플(Named tuple):**==
	1. tuple을 만들때 각각의 데이터 마다 이름을 매긴것
	2. 코드의 가독성이 높아짐
```swift
//타입이 정해져 있지 않을때, 초함될 데이터 갯수 마음대로 정의 가능
let towNumbers: (Int, Int) = (1, 2)
let threeNumbers = (1, 2, 5)
var threeValues = (name: "애플", age: 20, address: "포항")


//코드의 가독성 높이기

```

- 튜플에 이름 붙여 가독성 높이기
	1. 튜플을 만들때 이름을 붙이면 .0, .1처럼 인덱스로 접근 필요 없음
	2. 의미의 명확성 증가
```swift
let iOS = (language: "Swift", version: 5)
print(iOS.language) //"Swift"
print(iOS.version) //5
```

- **==튜플의 분해(Decomposition) :==**
	1. 튜플의 데이터 묶음을 1개씩 분해해 상수나 변수에 저장 가능 (상수, 변수 모두 사용 가능)
	2. 모아놓았던 데이터를 분해해서 사용하고 싶은 경우 사용
	3. 반복되는 let 혹은 var 키워드를 여러번 사용하지 않아도 됨
	4. 변수나 상수가 많을때 한줄로 정리 가능
	5. 타입 앨리어스(typealias)를 이용해서 치환하여 사용 가능
	   typealias GridPoint = (Int, Int)
```swift
let (first, second, third) = (5, 6, 7)
//let first = 5, let second = 6, let third = 7

//위에 코드와 아래 코드는 같음
let first = 5
let second = 6
let third = 7
```
- 필요한 값만 꺼낼 수도 있음
```swift
let (x, _, z) = (1, 2, 3)
//두번째 값은 무시함
```

- ==**튜플의 값 비교 :**==
	1. 실제 사용하는 경우 흔하지 않음
	2. 두개의 튜플 비교 가능 → 왼쪽 멤버부터 한번에 하나씩 비교 후, 같을 경우 다음 멤버 비교
	3. 튜플은 최대 6개 이하만 비교 가능(애플 라이브러리 가능)
	   공식적으로 튜플 비교는 6개 이하만 가능, 7개 이상은 직접 비교 연산 하거나, struct로 바꿔야함
- **튜플끼리 비교 연산자(, !=, <, > 등) 사용**
	1. 각 자리의 값이 모두 값으면 true
	2. 각 자리의 값이 하나라도 다르면 false
```swift
let a = (1, 2)
let b = (1, 2)
print(a == b) //true
```
- **같은 튜플인지 비교**
```swift
let score1 = (100, 90)
let score2 = (100, 90)
let score3 = (95, 90)

print(score1 == score2) //true
print(score1 == score3) //false
```
- **순서를 고려한 크기 비교**
```swift
let a = (1, 5)
let b = (2, 0)
print(a < b)
//true → 1 < 2 이므로 true

let x = (3, 4)
let y = (3, 7)
print(x < y)
//true → 3 == 3, 두번째 요소 비교: 4 < 7
```
- **비교가 안되는 경우** (7개 요소 이상일때, 이럴때는 튜플대신 struct 사용)
```swift
let a = (1, 2, 3, 4, 5, 6, 7)
let b = (1, 2, 3, 4, 5, 6, 7)
//print(a == b)
//에러: 튜플 비교는 6개 까지 가능함
```
- **튜플 비교 가능 조건**
	1. Swift에서 튜플 요소들이 비교 가능한 타입(Equatable, Comparable 등)일때만 비교 지원
	2. 각 요소 타입이 Equatable(같다/다르다) 이면, == , != 가능
	3. 각 요소 타입이 Comparable < , > 등도 가능
	4. 튜플 요소가 6개 이하일 때만 비교 가능!

| ==**구분**==     | ==의미==   | ==예시 연산자==      | ==대표 타입==           |
| -------------- | -------- | --------------- | ------------------- |
| ==**Equatable**==  | 같다 / 다르다 | == , !=         | Int, String, Bool   |
| ==**Comparable**== | 크다 / 작다  | < , > , <= , >= | Int, Double, String |



## tuple는 어디에 쓸까?

- **==함수의 여러 값 반환==**
	1. 함수에서 값을 여러개 한꺼번에 반환할때 가장 많이 사용
	2. 복잡한 구조 없이 간단하게 여러 데이터를 전달할 때 사용
```swift
func UserInfo() -> (name: String, age: Int) {
    return ("애플", 4)
}

let user = UserInfo()
print(user.name) // "애플"
print(user.age)  // 4
```

- ==**여러 값을 한줄에 선언하기 (분해)**==
	1. 튜플을 사용하면 변수 여러 개를 한줄에 동시에 초기화 가능
	2. 함수 리턴값을 바로 분리하거나 값 그룹을 편하게 다룰 때 사용
```swift
let (x, y, z) = (10, 20, 30)
print(x) //10
print(y) //20
print(z) //30
```

- ==**값 교환(swap) 간단히 처리하기==**
	1. 임시 변수 없이, 한 줄로 값 교환 가능
	2. 스왑할 때 가장 짧고 간결한 방법
```swift
var a = 1
var b = 2
(a, b) = (b, a)
print(a) // 2
print(b) // 1
```

- ==**switch문과 튜플 패턴 매칭==**
	1. 여러 조건을 한꺼번에 비교해야 할 때 swift + tuple 조합 좋음
	2.  위치, 상태 등 다중 조건 분기에 좋음
```swift
let point = (1, 0)

switch point {
case (0, 0):
	print("원점")
case (_, 0):
	print("X축 위")
case (0, _):
	print("Y축 위")
default:
	print("좌표 평면 위")
}
```

- **==간단한 값 묶음용 구조==**
	1. 별도의 struct 만들기 귀찮을 때, 튜플로 간단한 데이터를 묶어서 전달 가능
```swift
let product = (name: "iPhone", price: 1299.0)
print("상품: \(product.name), 가격: \(product.price)만원")
```