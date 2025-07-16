#Combine
>[!question]
>GQ1. Combine에서 Subscription의 역할은 뭘까?
>GQ2. Publisher와 Subscriber를 연결하는 흐름이 어떻게 될까?

## GQ1.  Combine에서 Subscription의 역할
Combine에서 Publisher와 Subscriber가 연결되는 흐름을 보면 갑자기 Subscription이라는 개념이 언급됩니다. 너무 많은 용어가 나와서 혼란스러울 수도 있지만 사실 Combine에서 정말 중요한 역할을 담당하고 있는 친구인 Subscription에 대해 알아보겠습니다.

이전에 Combine에서 정리했던 걸 보면 Publisher가 subscription을 Subscriber에게 넘긴 다음에도 Subscriber와 직접 값을 주고받는 것처럼 나와있는데요, 사실 자세히 들여다보니 그게 전부가 아니었습니다.

![[스크린샷 2025-06-18 오전 1.29.11.png]]

### Publisher → Subscription → Subscriber
실제로는 Publisher가 Subscription을 만들어서 Subscription에게 넘겨주고 나면,
Publisher와 Subscriber가 직접 연결되지 않고 중간에서 Subscription이 중개 역할을 하게 됩니다.

![[Pasted image 20250618012805.png]]

즉, Publisher는 단순히 값을 '**발생시키는** 주체'일 뿐, 그 값을 어떻게 얼마나 전달할지는 Subscription이 담당합니다.


### 그래서 Subscription은 뭣을 하느냐?

1. 값 전달 조율
	- Publisher가 새 값을 만들었을 때, Subscription이 Subscriber에게 전달할지를 판단합니다.
	- 이 판단의 기준은 Subscriber가 요청한 양(Demand)
	- Subscriber가 요청한 양보다 더 많은 값을 받지 않도록 흐름을 제어합니다.

2. 참조 관리
	: Subscription은 Subscriber를 보유(retain)하고, 필요 없어지면 해제해줍니다.

3. Publisher: 바지사장👖
	- Subscription이 연결되고 나면, Publisher는 실제 값 전달에는 거의 관여하지 않습니다.
	- 실질적인 동작은 Subscription과 Subscriber가 주고받으며 이루어지고, Publisher는 Subscription에게 값을 주거나 이벤트를 전달할 뿐(시작점의 역할만!), 직접 Subscriber와 소통하지는 않습니다.

> [! 정리]
>-  Combine에서 실제 작업을 수행하는 건 대부분 Subscription과 Subscriber이다.
>-  Publisher는 일종의 시작점일 뿐!(바지사장)
>- Subscription이 Subscriber의 요청을 받아서 값을 전달하거나 직접 작업을 수행하는 등의 중개자 역할을 맡는다.

### 공식문서 파보기
공식문서의 설명을 보면 다음과 같이 나와있습니다.

> A protocol representing the connection of a subscriber to a publisher.
> : Publisher와 Subscriber 사이의 연결을 나타내는 프로토콜

```swift
public protocol Subscription: Cancellable, CustomCombineIdentifierConvertible {
    func request(_ demand: Subscribers.Demand)
}
```

- `request(_:)` 
  : Publisher에게 더 많은 값을 Subscriber에게 보내도 된다고 알린다.
	- Subscriber가 **“이만큼 받을게요”** 라고 말하면, Subscription이 그걸 받아서 **Publisher에게 더 보내도 된다**고 알려주는 역할
	- 처음 연결될 때도, 중간중간 더 받을 준비가 됐을 때도 이 request()를 호출

>[! Overview]
>1. **Subscriptions are class constrained** because a Subscription has identity, defined by the moment in time a particular subscriber attached to a publisher.
>2. Canceling a Subscription must be thread-safe.
>	- You can only cancel a Subscription once.
>	- Cancelling a subscription frees up any resources previously allocated by attaching the Subscriber

##### **Subscription은 클래스 타입으로만 만들 수 있다.**
왜냐하면 Subscription은 정체성(identity)을 가지기 때문인데, 
그 정체성은 특정 Subscriber가 Publisher에 연결된 ‘시점’에 의해 정의되기 때문
	
- Subscription은 단순히 데이터를 전달하는 도구가 아니라 특정 Subscriber가 Publisher에 연결되면서 생긴 고유한 연결 관계 자체
- 누가(Subscriber) 어떤 Publisher를 구독했는지, 언제 연결되었는지, 얼만큼의 데이터를 요청했는지와 같은 요소들에 따라 구분됨
- 구독이라는 행위의 결과로 생긴 고유한 인스턴스

⇒ 클래스는 참조 타입이기 때문에 객체 상태 공유 및 추적이 용이하기 때문에 subscription은 클래스 타입으로만 만들 수 있다!

##### Subscription의 취소(cancel)는 스레드 안전(thread-safe)해야 한다
- Cancellable 프로토콜을 채택하고 있어서 구독을 중단하고 리소스를 해제하는 `cancel()` 메서드를 가지고 있음
- 여러 스레드에서 동시에 구독을 취소하려는 상황이 생길 수 있는데 그 때에도 문제 없도록 한 번만 안전하게 취소되어야 한다!

## GQ2. Publisher와 Subscriber를 연결하는 흐름이 어떻게 될까?

백문이 불여일견. 이제 실제 코드를 통해 Publisher, Subscriber, Subscription이 어떻게 구현되고 서로 연결되는지 직접 눈으로 확인해보겠습니다.
#### Publisher
✔️ Output이 제너럴로 구현되어 있음
✔️ receive 메서드 안에서 Subscription이 만들어져서 subscriber의 receive 함수로 건네주고 있음
```swift
extension URLSession {
  struct DecodedDataTaskPublisher<Output: Decodable>: Publisher {
    typealias Failure = Error

    let urlRequest: URLRequest

    func receive<S>(subscriber: S) where S : Subscriber, Failure == S.Failure, Output == S.Input {
      let subscription = DecodedDataTaskSubscription(urlRequest: urlRequest, subscriber: subscriber)
      subscriber.receive(subscription: subscription)
    }
  }

  func decodedDataTaskPublisher<Output: Decodable>(for urlRequest: URLRequest) -> DecodedDataTaskPublisher<Output> {
    return DecodedDataTaskPublisher<Output>(urlRequest: urlRequest)
  }
}
```
#### Subscriber
✔️ Subscriber는 `subscription.request(_:)`를 호출해야 값이 온다
✔️ 수명 관리: subscription은 저장해서 취소 가능하게 해야 한다.
✔️ 값을 받아도 다시 요청하지 않으면 더이상 오지 않는다: `receive(_:)`에서 .none을 리턴하면 demand를 더 늘리지 않겠다는 뜻
```swift
class DecodableDataTaskSubscriber<Input: Decodable>: Subscriber, Cancellable {
	typealias Failure = Error
	
	var subscription: Subscription?
	
	func receive(subscription: Subscription) {
		print("Received subscription")
		self.subscription = subscription // 저장
		subscription.request(.unlimited)
	}
	
	func receive(_ input: Input) -> Subscribers.Demand {
		print("Received value: \(input)")
		return .none
	}
	
	func receive(completion: Subscribers.Completion<Error>) {
		print("Received completion \(completion)")
		cancel()
	}

	// 취소
	func cancel() {
		subscription?.cancel()
		subscription = nil
	}
}
```

#### Subscription
✔️ `request(_:)`
	: Subscriber가 값을 요청하고, 그 demand가 0보다 크면 네트워크 요청이 시작됨
✔️ 작업 완료 후 `cancel()` 호출
	- dataTask의 응답을 처리한 후에는 defer를 이용해 cancel()을 호출
✔️ 값 전달과 완료/실패 처리까지 Subscription이 맡는다(중개자 역할)
```swift
extension URLSession.DecodedDataTaskPublisher {
  class DecodedDataTaskSubscription<Output: Decodable, S: Subscriber>: Subscription
    where S.Input == Output, S.Failure == Error {

    private let urlRequest: URLRequest
    private var subscriber: S?

    init(urlRequest: URLRequest, subscriber: S) {
      self.urlRequest = urlRequest
      self.subscriber = subscriber
    }

    func request(_ demand: Subscribers.Demand) {
      if demand > 0 {
        URLSession.shared.dataTask(with: urlRequest) { [weak self] data, response, error in
          defer { self?.cancel() }

          if let data = data {
            do {
              let result = try JSONDecoder().decode(Output.self, from: data)
              self?.subscriber?.receive(result)
              self?.subscriber?.receive(completion: .finished)
            } catch {
              self?.subscriber?.receive(completion: .failure(error))
            }
          } else if let error = error {
            self?.subscriber?.receive(completion: .failure(error))
          }
        }.resume()
      }
    }

    func cancel() {
      subscriber = nil
    }
  }
}
```

## References
[Documentation) Subscription](https://developer.apple.com/documentation/combine/subscription)
[Understanding Combine’s publishers and subscribers](https://www.donnywals.com/understanding-combines-publishers-and-subscribers/)
[Publisher와 Subscriber간 연결 과정 딥다이브](https://codingmon.tistory.com/71)
