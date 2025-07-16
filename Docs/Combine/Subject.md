#Combine

>[!question]
>Combine에서 Subject는 무엇이고 왜 필요할까?

## #Subject 는 무엇일까?

Subject는 Combine 프레임워크에서 데이터를 보내고, 받을 수 있는 특별한 Publisher입니다. 일반적인 Publisher는 값을 “내보내기만” 하지만, Subject는 외부에서 값을 직접 주입할 수 있다는 점이 핵심적인 차이점!

#### Subject의 정의
```swift
protocol Subject: Publisher, AnyObject {
	func send(_ value: Output)
	func send(completion: Subscribers.Completion<Failure>)
}
```
- **정체성은 Publisher:** Publisher 프로토콜을 따름
- **값을 보낼 수 있다:** `send(_:)`를 통해 외부에서도 스트림에 값을 주입할 수 있음

#### 외부에서 값을 보낸다는 건?
대체 '외부'에서 값을 방출한다는 게 무엇일까요?(어디부터 내부였고 그래서 Subject가 왜 외부라는건데)

 Combine에서는 보통 Publisher - Subscriber 사이에서 데이터 흐름(Stream)이 만들어지는데, Subject는 값의 생성자이자 전달자 역할을 함께 합니다. 즉, 값을 자동 생성하는 것이 아니라, 우리가 코드에서 명령형으로 send()를 호출해 값을 흘려보낼 수 있는 것이죠.

예시 코드로 보는 게 더 이해가 빠를 겁니다. 
```swift
let subject = PassthroughSubject<String, Never>()

// 외부 코드: send로 값을 직접 넣는 코드
subject.send("Hello")

// 내부 Subscriber: 구독자는 이 값을 받음
subject.sink { value in
    print("받은 값: \(value)")
}
```

일반적으로 Publisher는 자동판매기처럼 정해진 시점에 자동으로 값을 내보내는 반면에,
Subject는 `subject.send()` 처럼 스트림에 직접 값을 내보내는 코드를 작성할 수 있다는 거!

그럼으로서 Publisher 내부에서 자동으로 값이 생성되는 것이 아니라, 외부의 코드(예: 뷰 컨트롤러나 뷰 모델)에서 원할 때 직접 `send()`로 값을 내보내고 처리하도록 할 수 있는 것이죠.

Subject는 우리가 **버튼을 눌러 값을 직접 흘려보내는 형태**라고 생각하면 이해가 쉽습니다


#### WWDC에서 설명하는 Subject

>**Behave like both Publisher and Subscriber**

![[Pasted image 20250624174743.png]]
Subject는 `send(_:)` 메서드를 이용해 Subscriber에게 값을 보낼 수도 있고, Subscriber처럼 상위 Publisher에 구독할 수도 있습니다. 
또한 sink, map같은 Operator를 사용해서 스스로 Subscriber를 만들 수도 있습니다.

>**Broadcast values to multiple subscribers**

여러 하위 Subscriber에게 방송(broadcast)하듯 값을 전달할 수 있으며, 값을 명령형으로 직접 보낼 수 있습니다. 

그 값이 상위 Publisher에서 생성된 경우에도 마찬가지!(Subscriber처럼 상위 Publisher 구독)

```swift
// Using Subjects with Combine
let trickNamePublisher = ... // Publisher of <String, Never>

let magicWordsSubject = PassThroughSubject<String, Never>()

trickNamePublisher.subscribe(magicWordSubject)

let canceller = magicWordsSubject.sink { value in
	// do something with the value
}

magicWordsSubject.send("Please")

let sharedTrickNamePublisher = trickNamePublisher.share()
```

## Subject의 종류
### #PassthroughSubject

> **A subject that broadcasts elements to downstream subscribers.**
> : 요소(element)를 하위 Subscriber들에게 방송(broadcast)하는 Subject

- 방출값을 저장하지 않고 downstream Subscriber에게 값을 보내줍니다(저장하지 않는다는 걸 굳이 언급한 걸 보면 저장하는 것도 있겠죠?)
- 주로 **일회성 이벤트**, **알림** 등에 사용

#### 예시 코드
PassthroughSubject가 어떻게 흘러가는지를 알아보기 위해 다음과 같은 예시 코드를 준비했습니다.
1. String과 Never 타입의 PassthroughSubject 객체를 생성
2. sink 를 이용하여 subscription1을 생성
3. sink 를 이용하여 subscription2을 생성
4. 생성한 PassthroughSubject로 값을 보내기
```swift
example(of: "PassthroughSubject") {

// 1. String과 Never 타입의 PassthroughSubject 객체를 생성합니다.
let subject = PassthroughSubject<String, Never>()

// 2.sink 를 이용하여 subscription1을 생성합니다.
let subscription1 = subject
	.sink(
		receiveCompletion: { completion in
			print("Received completion (1)", completion)
		},
		receiveValue: { value in
			print("Received value (1)", value)
		}
	)

// 3.sink 를 이용하여 subscription2을 생성합니다.
let subscription2 = subject
	.sink(
		receiveCompletion: { completion in
			print("Received completion (2)", completion)
		},
		receiveValue: { value in
			print("Received value (2)", value)
		}
	)

//4. 값을 보냅니다.
subject.send("Hello")
subject.send("World")

}
```

이렇게 PassthroughSubject를 사용하면 `send(_:)`를 통해 필요에 따라 새로운 값을 게시할 수 있습니다.

broadcast니까 subject를 구독하고 있는 모든 subscriber에게 값을 보내고 그에 맞는 결과가 찍혀서 아래와같은 출력문이 나옵니다

```
Received value (1) Hello
Received value (2) Hello
Received value (1) World
Received value (2) World
```
### #CurrentValueSubject

> **A subject that wraps a single value and publishes a new element whenever the value changes.**
> : **최근 값을 저장**하고, 값이 바뀔 때마다 새로운 값을 발행하는 Subject

- `PassthroughSubject`와 달리 초기값과 최근 발행된 element에 대한 buffer를 가지고 있습니다
- 초기값을 설정해야 합니다
- 마지막으로 수신된 값의 기록을 저장하고 값이 변경될 때마다 새로운 값을 publish합니다.
- `value` 프로퍼티로 현재 값을 직접 읽거나 수정할 수 있습니다.

```swift
var subscriptions = Set<AnyCancellable>()

example(of: "CurrentValueSubject") {
	// 1. Int와 Never 타입을 갖는 CurrentValueSubject를 생성합니다.
	let subject = CurrentValueSubject<Int, Never>(0)
	
	// 2. sink를 통해 subject를 구독합니다.
	subject
		.sink(receiveValue: { print($0) })
		.store(in: &subscriptions)

	// 3. 새 값 발행하기
	subject.send(1)
	subject.send(2)
	
	// 4. 현재 값 확인하기
	print(subject.value) // 2

	// 5. value에 새 값을 직접 할당
	subject.value = 3
}

```

주석 추가설명
1. CurrentValueSubject는 0이라는 초기값과 함께 생성됨
2. Failure Type이 Never일때 receiveCompletion은 생략 가능
3. PassThroughSubject와 마찬가지로 `send(_:)`를 호출해서 새 값을 발행 가능
4. 이렇게 발행된 값은 현재 값으로 설정되고, 이 설정된 현재 값은 value 값으로 확인할 수 있음
5. value에 새 값을 직접 할당하는 방법을 통해서도 값을 발행하고 현재 값을 변경할 수 있음

### PassthroughSubject vs. CurrentValueSubject
그렇다면 이 두가지의 Subject는 어떤 상황에서 어떻게 선택하면 좋을까요?
선택의 기준은 값이 필요한지의 여부에 따라 결정됩니다.
##### 값을 저장하면서 상태를 유지하고 싶다면? → #CurrentValueSubject
- 현재 상태/값이 중요한 경우
- 새로운 Subscriber가 즉시 최신 값을 받아야 하는 경우
- 값 기반 로직 처리가 필요한 경우

**예시 코드**
1. 로그인한 사용자 정보 저장 및 업데이트
   : 앱 전체에서 로그인 상태를 추적해야 할 때
```swift
let currentUser = CurrentValueSubject<User?, Never>(nil)

// 구독: 사용자 정보에 반응하는 뷰모델 등
currentUser
    .sink { user in
        print("현재 사용자: \(user?.name ?? "없음")")
    }
    .store(in: &cancellables)

// 로그인 성공 시
currentUser.send(loggedInUser)
```

2. 서버에서 받아온 데이터 캐싱 및 변경 추적
   : 서버 응답 이후의 데이터를 상태로 보존하고 싶을 때
```swift
class WeatherService {
    private let weatherSubject = CurrentValueSubject<Weather?, Never>(nil)
    
    var currentWeather: AnyPublisher<Weather?, Never> {
        weatherSubject.eraseToAnyPublisher()
    }
    
    func fetchWeather() {
        let newWeather = Weather(temperature: 25, condition: "Sunny")
        weatherSubject.send(newWeather)
    }
    
    // 현재 캐시된 날씨 정보에 즉시 접근
    var latestWeather: Weather? {
        weatherSubject.value
    }
    
    // CurrentValueSubject의 활용 예시들
    func shouldShowWeatherAlert() -> Bool {
        guard let weather = latestWeather else { return false }
        return weather.temperature > 35 || weather.temperature < -10
    }
    
    func getWeatherSummary() -> String {
        guard let weather = latestWeather else { 
            return "날씨 정보 없음" 
        }
        return "\(weather.temperature)°C, \(weather.condition)"
    }
    
    func isWeatherDataStale() -> Bool {
        // 현재 값이 있는지 즉시 확인
        return latestWeather == nil
    }
}

// 실제 활용 예시
let weatherService = WeatherService()

// 1. 네트워크 요청 전에 캐시된 데이터 확인
if weatherService.isWeatherDataStale() {
    print("캐시된 데이터 없음, API 호출 필요")
    weatherService.fetchWeather()
} else {
    print("캐시된 데이터 사용: \(weatherService.getWeatherSummary())")
}

// 2. 현재 날씨 기반으로 즉시 결정
if weatherService.shouldShowWeatherAlert() {
    print("극한 날씨 경고 표시!")
}

// 3. 구독과 동시에 현재 값도 활용
weatherService.currentWeather
    .sink { weather in
        print("구독으로 받은 업데이트: \(weather?.temperature ?? 0)°C")
        
        // 구독 내에서도 현재 값에 접근 가능
        let summary = weatherService.getWeatherSummary()
        print("현재 요약: \(summary)")
    }
    .store(in: &cancellables)
```

##### 그냥 이벤트를 흘려보내고 싶다면 → #PassthroughSubject
- 이벤트 기반 통신 (버튼 탭, 네트워크 완료 등)
- 상태보다는 액션이 중요한 경우
- 과거 값이 의미없는 일회성 이벤트

1. 버튼 탭 처리
   : 사용자가 버튼을 누를 때마다 이벤트를 방출하지만, 이전에 누른 건 기억할 필요가 없음
```swift
let didTapButton = PassthroughSubject<Void, Never>()
```

2. Alert 또는 Toast 띄우기
   : “에러 메시지” 같은 건 그 순간만 나타내고 말기 때문에 값을 저장할 필요 없음.
```swift
let showAlert = PassthroughSubject<String, Never>()
```

3. 다시 시도 요청
   : “다시 시도” 버튼을 눌렀을 때 네트워크 요청을 다시 보내는 트리거로 사용.
```swift
let retryRequested = PassthroughSubject<Void, Never>()
```


## References
official
[WWDC19 Combine in Practice](https://developer.apple.com/videos/play/wwdc2019/721/)
[Apple Documentation) Subject](https://developer.apple.com/documentation/combine/subject/)
blog
[코딩하는 체대생 Combine Subject](https://mini-min-dev.tistory.com/290)
[날진 Subject](https://sujinnaljin.medium.com/combine-subject-a974340cb582)