#Combine 
>[!question]
>GQ1. Subscriber는 왜 AnyCancellable을 반환할까? Cancellable이 뭐길래?
>GQ2. Cancellable을 반환하는 subscriber는 어떻게 함수로 만들 수 있는걸까?

## GQ1. Subscriber는 왜 AnyCancellable을 반환할까?
sink나 assign같은 Subscriber의 정의를 보면 Subscirber는 따로 Combine이라는 클래스나 subscription을 반환하지 않고 AnyCancellable 타입을 return하는 것을 볼 수 있습니다.
```swift
func assign<Root>(
	to keyPath: [ReferenceWritableKeyPath]<Root, Self.[Output]>,
    on object: Root
) -> AnyCancellable
```

```swift
func sink(reciveValue: @escaping (Self.Output) -> Void) -> AnyCancellable
```

그런데 대체 AnyCancellable이 뭐길래 이런 구조일까요? 이를 이해하기 위해서는 Cancellable 프로토콜을 먼저 알아야 합니다.
### Cancellable vs AnyCancellable
#### Protocol Cancellable
>A protocol indicating that an activity or action supports cancellation.
>→ 어떤 작업이 취소를 지원함을 나타내는 **프로토콜**

프로토콜이라면 어떤 게 요구사항이 있을까요? 바로 cancel입니다

```swift
func cancel()
```
cancel() 메서드를 호출하면 해당 작업과 관련된 모든 리소스가 해제됩니다

이를 통해 타이머, 네트워크 요청, 디스크 IO 등에서 발생할 수 있는 불필요한 자원 낭비, 메모리 누수 같은 부작용을 막아줄 수 있습니다.

#### final Class AnyCancellable
> A type-erasing cancellable object that executes a provided closure when canceled.
> → Cancellable 프로토콜을 따르는 타입을 타입 지우기(type-erasing)하여 저장할 수 있는 **클래스**

- Cancellable을 채택하는 '클래스'
- **타입 지우기(type-erasing)**: 다양한 Cancellable 타입을 AnyCancellable로 래핑하여 동일한 타입으로 관리할 수 있습니다
- **자동 구독 취소**: AnyCancellable 인스턴스가 메모리에서 해제(deinit)될 때 자동으로 cancel이 호출되어 구독이 취소됩니다
- 이 때 cancel 호출될 때 실행할 작업을 AnyCancellable을 생성할 때 클로저로 지정해줄 수 있습니다
	- AnyCancellable의 생성자: `init(() -> Void)`
	- 생성자에서 클로저로 받은 '구독 취소 시 실행할 작업'이 `cancel()`이 호출될 때 실행할 수 있도록 구현되어 있습니다.


### 💡 AnyCancellable은 구독취소권!🎟️

**Publisher - Subscriber 흐름**
1. Publisher와 Subscriber가 연결되면 내부적으로 Subscription 객체가 생성됩니다.
2. Subscription은 데이터 스트림을 전달하는 통로이자, 구독 상태를 제어할 수 있는 핵심 객체
3. 근데 이제 개발자가 Subscription을 직접 다루는 경우는 드물다!

→ 내부적으로 Subscription을 생성하지만 직접 이를 리턴하지 않고 AnyCancellable을 리턴
→ Subscription을 직접 노출하지 않고, 구독을 제어할 수 있는 구독 취소권, AnyCancellable만 건네주는 구조

## GQ2. Cancellable을 반환하는 subscriber는 어떻게 함수로 만들 수 있는걸까?

```swift
init() {
	subscribeValidatedCredentials()
}

private var cancellables = Set<AnyCancellable>()

private func subscribeValidatedCredentials() {
    self.validatedCredentials
        .map { $0 != nil }
        .receive(on: RunLoop.main)
        .assign(to: \.isEnabled, on: signupButton)
        .store(in: &cancellables)
}
```
Subscriber 예시 코드를 보면 위와 같이 subscriber를 함수로 만들어두고 init에서 호출하는 형태로 사용하는 것을 볼 수 있었는데, 두 가지 질문이 생겼습니다.

1. Cancellable을 반환하는 걸 이렇게 함수 형태로 써도 되나?
2. init에서 꼭 호출해야되나? 왜 저렇게 하지?

결론은 이렇습니다!

1. 함수로 만드는 것 **완전 가능**. 복잡한 구독 로직을 별도 함수로 분리함으로써 코드 정리가 가능해짐
2. init()에서 호출하는 이유
   - Combine은 지연 실행(lazy) 이기 때문에, 스트림을 선언하거나 func로 정의해두기만 하면 아무 실행도 하지 않음
   - 함수로 정의해두고 init에서 호출하지 않으면, 컴바인 흐름이 일어나지 않음. (func으로 Combine 스트림을 만들어둬도 호출하지 않으면 아무 일도 일어나지 않음)

## 힘들게 만든 구독을 유지하려면, `store(in:)`

```swift
final func store(in set: inout Set<AnyCancellable>)

final func store<C>(in collection: inout C) where C : RangeReplaceableCollection, C.Element == AnyCancellable
```
Combine에서 구독을 유지하려면, **반환된 AnyCancellable 인스턴스를 반드시 저장해야** 합니다.

저장하지 않으면 AnyCancellable은 생성 직후 메모리에서 해제(deinit)되며,
**자동으로 cancel()이 호출**되어 구독이 즉시 취소됩니다. // Nooo....🙂‍↔️

#### 활용 방법
```swift
import Combine

class ViewModel {
    var cancellables = Set<AnyCancellable>()

    func start() {
        Just("Hello")
            .sink { print($0) }
            .store(in: &cancellables)
    }
}
```

1. `Set<AnyCancellable>` 만들기
2. Subscriber가 반환하는 AnyCancellable을 `.store(in:)`을 사용해 저장

→ 이렇게 `store(in:)`로 AnyCancellable을 저장하면,

- ViewModel 인스턴스가 살아 있는 동안 구독이 유지되고
- ViewModel이 해제되면, 구독도 자동으로 최소되어 메모리 누수를 방지할 수 있습니다!

## References
Apple Documentation)
[Cancellable](https://developer.apple.com/documentation/combine/cancellable)
[AnyCancellable](https://developer.apple.com/documentation/combine/anycancellable)
[store(in:)](https://developer.apple.com/documentation/combine/anycancellable/store(in:)-6cr9i)

Blogs)
[Zedd0202 - Combine(4) Cancellable](https://zeddios.tistory.com/981)
[(빠지지 않는..)코딩하는 체대생 - Cancellable, AnyCancellable](https://mini-min-dev.tistory.com/299)

Thanks to) GPT & Claude Sonnet👏

// 다음주는 Subscription으로 돌아올 예정