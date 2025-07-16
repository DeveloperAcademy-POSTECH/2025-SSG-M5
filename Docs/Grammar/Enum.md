

**enum 을 활용하여 네트워크 상태 관리를 하려면 어떻게 해야할까?**

>[!question]
>GQ1. enum 에서 상태에 따라 다른 타입의 정보를 담는 방법은 뭐가 있을까?
>GQ2. enum 에서 case 뿐만 아니라 associated value 의 동등성 여부 '==' 를 확인하려면 어떻게 해야할까?
>GQ3. enum 에서 직렬화하려면 어떻게 해야할까?

## Description


Enum이란?
관련된 값의 그룹을 공통된 타입을 정의하며, 이 값들을 타입 안정성 있게 다룰 수 있도록 한다.
Swift에서 Enum은 그 자체로가일급 객체(first-class type)이다.
컴파일 타임에 타입이 결정되어 런타임 오류를 방지하여 타입이 안정적이다.

각각의 케이스 값과 함께 저장될 연관된 모든 유형의 값을 지정할 수 있다. : *associated values*
동등성 여부를 확인하기 위해서는 Equatable 을 선언해주면 된다.




> [!일급객체의 조건]
> - 변수에 할당할 수 있는가?
> - 함수의 인자로 전달할 수 있는가?
> - 함수의 반환값으로 사용할 수 있는가?

> [!Enum을 쓸 수 있는 적절한 상황]
> 
> - 한정된 선택지가 있을 때
> - 상태(state)를 명확하게 할 때
> - 연관된 값들과 함께 표현할 때
> - Switch 문과 함께 명확한 분기처리를 할 때
> - 특정 타입의 의미를 명확하게 할 때

> [!직렬화란]
> 데이터를 보낼 수 있는 형태로 바꾸는 것
> 메모리에 존재하는 복잡한 자료(예: 클래스 인스턴스, 열거형, 딕셔너리 등)를 저장하거나 
> 네트워크로 전송할 수 있도록파일, 문자열, JSON, 바이너리 등의 형태로 변환하는 과정
> 
> 직렬화된 데이터를 원래 형태로 돌리는 것이 역직렬화(Deserialization)
> 

  

ENUM 을 쓰기 좋은 예시는 명확하고 한정적인 어떤 그룹을 모델링 할 때 활용하기에 적절하다.


## 코드 예시

###### GQ1
Swift 의 Enum은 associated value 를 가지고 있을 수 있다.
associated value 는 데이터를 동반한 복합적인 상태 표현이 가능하다.
상태마다 다른 정보 담을 수 있기 때문에 각 상태마다 다른 데이터를 표현해야할 때 유용하다.

```swift
enum NetworkState { 
	case idle // 아무것도 안함 
	case loading(progress: Double) // 로딩중 + 진행률 
	case success(data: String) // 성공 + 받은 데이터 
	case failure(message: String) // 실패 + 에러 메시지 
}
```


###### GQ2
동등성(Equatable)을 선언해주면, enum의 상태값과 내부의 값 까지 함께 값을 비교해준다.
예를 들어 상태값이 같더라도, associated value가 다르면 그것까지 확인해서 동등성을 비교할 수 있다.

```swift

extension NetworkState: Equatable {
    static func == (lhs: NetworkState, rhs: NetworkState) -> Bool {
        switch (lhs, rhs) {
        case (.idle, .idle):
            return true

        case (.loading(let leftProgress), .loading(let rightProgress)):
            return leftProgress == rightProgress

        case (.success(let leftData), .success(let rightData)):
            return leftData == rightData

        case (.failure(let leftMessage), .failure(let rightMessage)):
            return leftMessage == rightMessage

        default:
            return false // 서로 다른 case는 같지 않음
        }
    }
}


// 사용 예시 
let state1 = NetworkState.loading(progress: 0.5) 
let state2 = NetworkState.loading(progress: 0.5) 
let state3 = NetworkState.loading(progress: 0.3) 

print(state1 == state2) // true (같은 진행률) 
print(state1 == state3) // false (다른 진행률)
```


Swift에서 enum이 연관값을 가지는 경우 자동으로 Equatable을 채택할 수 없다.
이때는 Equatable을 수동으로 구현하여 각 case 및 연관값이 같은지 직접 비교해야 합니다.

Swift 5.3 이후로 조건만 충족되면 연관값이 모두 Equatable인 경우 자동으로 Equatable 채택이 가능하지만, 복잡한 비교 로직이 필요한 경우에는 여전히 수동 구현이 필요하다.


```swift
enum NetworkStateCustom: Equatable {
    case idle
    case loading(progress: Double)
    case success(data: String)
    case failure(message: String)
    
    // 커스텀 비교 로직 구현
    static func == (lhs: NetworkStateCustom, rhs: NetworkStateCustom) -> Bool {
        switch (lhs, rhs) {
        case (.idle, .idle):
            return true
            
        case let (.loading(lhsProgress), .loading(rhsProgress)):
            // 진행률 차이가 0.1 이하면 같은 것으로 취급
            return abs(lhsProgress - rhsProgress) <= 0.1
            
        case let (.success(lhsData), .success(rhsData)):
            // 데이터 길이만 비교 (실제 내용은 무시)
            return lhsData.count == rhsData.count
            
        case let (.failure(lhsMessage), .failure(rhsMessage)):
            // 에러 메시지는 대소문자 구분 없이 비교
            return lhsMessage.lowercased() == rhsMessage.lowercased()
            
        default:
            return false
        }
    }
}

// 커스텀 비교 로직 테스트
let custom1 = NetworkStateCustom.loading(progress: 0.51)
let custom2 = NetworkStateCustom.loading(progress: 0.59)
let custom3 = NetworkStateCustom.loading(progress: 0.70)

print(custom1 == custom2) // true - 차이가 0.1 이하
print(custom1 == custom3) // false - 차이가 0.1 초과
```

###### GQ2

```swift
func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        
        switch self {
        case .idle:
            try container.encode("idle", forKey: .type)
            
        case .loading(let progress):
            try container.encode("loading", forKey: .type)
            try container.encode(progress, forKey: .progress)
            
        case .success(let message):
            try container.encode("success", forKey: .type)
            try container.encode(message, forKey: .message)
            
        case .failure(let error):
            try container.encode("failure", forKey: .type)
            try container.encode(error, forKey: .error)
        }
    }

```



```swift
	init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        let type = try container.decode(String.self, forKey: .type)
        
        switch type {
        case "idle":
            self = .idle
            
        case "loading":
            let progress = try container.decode(Double.self, forKey: .progress)
            self = .loading(progress: progress)
            
        case "success":
            let message = try container.decode(String.self, forKey: .message)
            self = .success(message: message)
            
        case "failure":
            let error = try container.decode(String.self, forKey: .error)
            self = .failure(error: error)
            
        default:
            throw DecodingError.dataCorrupted(
                DecodingError.Context(codingPath: decoder.codingPath, 
                                    debugDescription: "알 수 없는 타입: \(type)")
            )
        }
    }
```


그리고, **값이 없는** 단순한 Enum은 Codable만 붙이면 알아서 직렬화/역직렬화가 된다

```swift
enum SimpleState: Codable {
    case waiting
    case working
    case done
    case failed
}
```




## References
- [swift.org](https://bbiguduk.gitbook.io/swift/language-guide-1/enumerations)

## Keywords

- 일급객체
- Equatable
- Codable
- Encoder
- [[Grammar]]


