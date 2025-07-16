>[!question]
> swift에서 에러처리방법에는 어떤것이 있을까?
> 어떤 경우에 error handling을 사용하는가?


## Description
Swift의 에러는 Error 프로토콜을 준수하는 타입의 값입니다.

에러를 직접 정의하는것도 가능합니다. 애플에서 권장하는 형태는 Enum을 에러로 정의하는 것 입니다.

## 주요 기능

### 에러를 던지는 함수

코드안에서 에러를 처리할 부분이 있다는것을 명시하는 부분이 있을때 
함수의 경우 반환타입을 명시하는 -> 앞에 throws를 붙이는것으로 표시합니다.

이러한 함수 형태를 Throwing Function이라고 합니다.


```swift
//에러 열거형
enum VendingMachineError: Error {
    case invalidSelection
    case insufficientFunds(coinsNeeded: Int)
    case outOfStock
}

struct Item {
    var price: Int
    var count: Int
}

class VendingMachine {
    var inventory = [
        "Candy Bar": Item(price: 12, count: 7),
        "Chips": Item(price: 10, count: 4),
        "Pretzels": Item(price: 7, count: 11)
    ]
    var coinsDeposited = 0

    func vend(itemNamed name: String) throws {
        guard let item = inventory[name] else {
            throw VendingMachineError.invalidSelection
        }

        guard item.count > 0 else {
            throw VendingMachineError.outOfStock
        }

        guard item.price <= coinsDeposited else {
            throw VendingMachineError.insufficientFunds(coinsNeeded: item.price - coinsDeposited)
        }

        coinsDeposited -= item.price

        var newItem = item
        newItem.count -= 1
        inventory[name] = newItem

        print("Dispensing \(name)")
    }
}
```

### do-catch
do안에 오류가 발생할 가능성이 있는 코드를 담고 try에 에러가 발생 할 수 있는 코드를 실행시킨다.

```swift
do {
    try expression code here
    statement code here
} catch error pattern code{
	statement code here
}
```

### 에러 전파
```swift
do {
    try expression code here
    statement code here
} catch error pattern code{
	statement code here
} catch error pattern code{
	statement code here
} catch error pattern code{
	statement code here
} catch error pattern code{
	statement code here
}

```

try에서 에러가 발생하면 그 즉시 catch로 이동한다.

하나의 catch 에서 do안에서 발생할 수 있는 **모든 에러를 처리할 필요는 없다.** 하지만 catch에서 에러를 처리하지 않으면 error는 error 처리가 가능한곳을 찾을때까지 error를 전파하고 다닌다. 그러다 

**_에러가 처리되지 않고 범위의 최상위로 전파되면 런타임 에러가 발생하게 된다._**

그래서 기본적으로는 일반 함수에서는 do-catch 구문에서 에러를 처리해야 하고 던지기 함수(throwing fucntion)에서는 do-catch 구문이나 호출자가 에러를 처리해야 한다

정리하자면 에러를 처리하는 do-try-catch구조에서 에러가 처리될 수 있는 catch를 찾아 퍼져나가는 것을 에러전파라고 한다.

### try?
try에 물음표?나 느낌표!를 넣는 이유는 무엇일까요?
에러를 옵셔널로 처리해서 사용하기 위함입니다.

try?는 

```swift
func someThrowingFunction() throws -> Int {
    // ...
}

let x = try? someThrowingFunction()

let y: Int?

do {
    y = try someThrowingFunction()
} catch {
    y = nil
}

func fetchData() -> Data? {
    if let data = try? fetchDataFromDisk() { return data }
    if let data = try? fetchDataFromServer() { return data }
    return nil
}
```
### try!

```swift
let photo = try! loadImage(atPath: "./Resources/John Appleseed.jpg")
```

try!는 강제언래핑할때 사용하는 키워드인 만큼  간단하게 옵셔널 에러를 처리할 수 있게 해줍니다 하지만 강제 언래핑 인 만큼 nil값이 들어오게 된다면 런타임에러가 발생할 수 있습니다.



## 코드 예시


## Keywords
+ Enum
+ Error
+ do-catch
+ [[Grammar]]

## References
- https://bbiguduk.gitbook.io/swift/language-guide-1/error-handling