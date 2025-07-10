>[!question]
>GQ1. Combine은 어떨 때 사용할까?(Combine은 왜 필요할까?)
>GQ2. Combine을 어떻게 쓸까?

## Description
- Combine은 왜 언제 써먹어야 하는가
- Combine의 기본 구성요소: **Publisher**, **Operator**, **Subscriber**

## Combine은 왜 언제 써먹어야 하는가
1. 반응형 UI: 데이터가 바뀌면 UI도 자동으로 즉시 업데이트되어야 하는 상황
	ex) 닉네임 Textfield로 입력받아서 중복검사 버튼 안눌러도 자동으로 중복검사하기
2. 여러 데이터 조합해서 검사하기: 닉네임 중복검사 통과 & 비밀번호랑 비밀번호 확인창
3. 값이 시간이 지나면서 계속 바뀌는 상황: 타이머, 사용자의 텍스트 입력, 위치 업데이트 등

## Combine의 기본 구성요소

1. Publisher
2. Subscriber
3. Operator
        
### Publisher

> to expose values that can change over time
> : 내보내는 애!

```swift
protocol Publisher {
	associatedtype Output
	associatedtype Failure: Error
	 
	func subscribe<S: Subscriber>(_ subscriber: S)
		 where S.Input == Output, S.Failure == Failure
}
```

- `Output`: Publisher가 내보낸 값의 타입(The kind of values published by this publisher.)
- `Failure`: Publisher가 내보낼 수 있는 오류의 종류(The kind of errors this publisher might publish.)

- `subscribe<S: Subscriber>(_ subscriber: S)`
  : 지정한 Subscriber를 이 Publisher에 연결

### Subscriber

> to receive those values from the publishers
> : 받는 애!

```swift
protocol Subscriber {
	associatedtype Input
	associatedtype Failure: Error
	
	func receive(subscription: Subscription)
	func receive(_ input: Input) -> Subscribers.Demand
	func receive(completion: Subscribers.Completion<Failure>)
}
````

- `Input`: Subscriber가 받는 값의 타입
  (The kind of values this subscriber receives.)
- `Failure`: Subscriber가 받을 수 있는 에러의 종류
  (The kind of errors this subscriber might receive.)

- `receive(subscription: Subscription)`
  : Subscriber에게 Publisher를 성공적으로 구독했으며 항목을 요청할 수 있음을 알림
- `receive(_ input: Input) -> Subscribers.Demand`
  : Subscriber에게 Publisher가 요소를 생성했음을 알림
- `receive(completion: Subscribers.Completion<Failure>)`
  : Publisher가 정상적으로 또는 오류로 게시를 완료했음을 Subscriber에게 알림

- 편리하게 Subscriber를 사용하고 싶다면? [[Built-in Subscribers]]!

### Publisher ↔ Subscriber 흐름
- Subscriber와 Publisher를 연결하기 위해서는 Input과 Output의 타입이 일치해야 하고, Failure도 일치해야 한다!

![[스크린샷 2025-06-18 오전 1.29.11.png]]
1. 🙋🏻 Subscriber: "나 구독할래"
	Subscriber는 subscribe 메서드를 통해 Publisher가 방출하는 데이터를 받겠다고 구독(subscribe)을 신청한다.
2. 🙆🏼 Publisher: 구독증🎟️ 드릴게
	Publisher는 Subscription(구독증) 객체를 만들고, Subscriber의 `receive(subscription:)`의 파라미터를 통해 Subscription을 보낸다. 이를 통해 Subscriber가 데이터를 요청하거나 구독을 해제할 수 있게 한다.
3. 구독증을 받은 Subscriber🙋🏻는 해당 구독증(subscription🎟️)으로 Publisher🙆🏼에게 N개의 값을 요청(request)한다
4. Publisher🙆🏼는 Subscriber🙋🏻가 요청한 만큼 N개의 값을 보낸다(`receive(_:)`)
5. Publisher🙆🏼가 더 이상 값을 publish하지 않거나 에러가 발생해서 관계가 종료되면 Publisher🙆🏼가 `receive(completion:)`을 호출해 관계의 끝을 알린다.

### [Operator](Operator_Combine)
- Publisher가 방출한 값을 변환
- Publisher 프로토콜을 conform
	⇒ Subscriber에게 값을 방출하는 자체적인 Publisher로서 동작할 수 있음
	1. 상위 Publisher(upstream) 구독
	2. 변환
	3. 하위 Subscriber(downstream)에게 결과 값 보내는 식
	
- 다양한 operator
	- 값 필터링: `.filter`, `.removeDuplicates`
	- 시간 제어: `.debounce`, `.throttle`
	- 값 변경: `.map`, `.flatMap`
	- 스트림 합치기: `.combineLatest`, `.merge`, `.zip`
	- 에러 처리: `.catch`, `.retry`


## 코드 예시

#### Publishers
```swift
// MARK: - Publisher
@Published private var userName: String = ""
@Published private var password: String = ""
@Published private var passwordAgain: String = ""

// First Publisher
// Passwords must matched & > N Characters
var validatedPassword: AnyPublisher<String?, Never> {
    return $password
        .combineLatest($passwordAgain)
        .map { password, passwordAgain in
            guard password == passwordAgain, password.count > N else { return nil }
            return password
        }
        .eraseToAnyPublisher()
}

// Second Publisher
// User name is valid according to server
var validatedUsername: AnyPublisher<String?, Never> {
	return $username
		.debounce(for: 0.5, scheduler: RunLoop.main)
		.removeDuplicates()
    .flatMap { userName in
	    return Future { promise in
	      self.userNameAvailable(username) { available in
	        promise(.success(available ? username : nil))
	      }
	    }
		}
		.eraseToAnyPublisher()
}

// Thrid Publisher
// Enabled if username and password valid
var validatedCredentials: AnyPublisher<(String, String)?, Never> {
	return validatedUsername
		.combineLatest(validatedPassword)
		.map { userName, password in
			guard let userName, let password else { return nil }
			return (userName, password)
		}
		.eraseToAnyPublisher()
}
```

#### Subscribers
```swift
// MARK: - Subscriber (assign)
private func subscribeValidatedCredentials() {
	self.signupButtonStream = self.validatedCredentials
		.map { $0 != nil }
		.receive(on: RunLoop.main)
		.assign(to: \.isEnabled, on: signupButton)
}

// MARK: - UI Components
private let signupButton = UIButton()
signupButton.backroundColor = $0.isEnabled ? .systemGreen : .systemGray

// MARK: - Subscriber (sink)
private func subscribeValidatedUserName() {
	validatedUsername
		.receive(on: RunLoop.main)
		.sink { [weak self] validatedUsername in
			// UI 변화 어쩌구저쩌구
			self?.usernameImage.tintColor = (validatedUsername == nil) ? .systemRed : .systemGreen
		}
		.store(in: &cancellables)
}

private func subscribeValidatedPassword() {
	validatedPassword
		.receive(on: RunLoop.main)
		.sink { [weak self] validatedPassword in
			// UI 변화 어쩌구저쩌구
			self?.passwordImage.tintColor = (validtedPassword == nil) ? .systemRed : .systemGreen
		}
		.store(in: &cancellables)
}
```

## Keywords
- Publisher
- [[Subscription]]
- Subscriber
- [[Built-in Subscribers]]
- [Operator](Operator_Combine)
- [[Cancellable&AnyCancellable]]
- [[Subject]]

## References
- [Apple Documentation) Combine](https://developer.apple.com/documentation/combine)
- [Apple Documentation) Receiving and Handling Events with Combine](https://developer.apple.com/documentation/combine/receiving-and-handling-events-with-combine)
- [WWDC19 Introducing Combine](https://developer.apple.com/videos/play/wwdc2019/722)
- [코딩하는 체대생 Combine (1/3)](https://mini-min-dev.tistory.com/288)
- [코딩하는 체대생 Combine (2/3)](https://mini-min-dev.tistory.com/289)
- [코딩하는 체대생 Combine (3/3)](https://mini-min-dev.tistory.com/290)