>[!question]
Combine에서 가장 많이 쓰이는, 유용한 Operator는 어떤 게 있을까?

### Combine에서 Operator는 무엇인가
가장 먼저, Combine에서 Operator가 무엇인지부터 정리하고 시작하겠습니다.

> [!Combine 문서에 정리해두었던 Operator] 
> - Publisher가 방출한 값을 변환
> - Publisher 프로토콜을 conform
>   ⇒ Subscriber에게 값을 방출하는 자체적인 Publisher로서 동작할 수 있음
> 1. 상위 Publisher(upstream) 구독
> 2. 변환
> 3. 하위 Subscriber(downstream)에게 결과 값 보내는 식

쉽게 말하면 Operator는 데이터의 흐름 자체인 Combine 파이프라인의 중간에서 데이터를 원하는 대로 바꾸고 제어하는 중간처리장치 역할을 합니다. 

공장의 컨베이어벨트가 돌아가고 있다고 상상해봅시다.
1. Publisher (데이터소스): 원재료를 벨트에 올리는 기계
2. Operator : 컨베이어벨트가 지나가면서 원재료를 자르고, 색칠하고, 검사하는 기계들
3. Subscriber: 완성된 제품을 받는 사람

이렇게 원재료를 자르고, 색칠하고, 검사하는 기계들이 할 수 있는 게 정확히 무엇인지, 어떤 기계들이 있는지를 알아보고(1탄), 이 기계들을 긴 컨베이어벨트에 한 개만 배치하는 게 아니라 여러 개의 기계를 연결(chaining)하는 방법까지(2탄) 알아보겠습니다.

### GQ. Combine에서 가장 많이 쓰이는, 유용한 Operator는 어떤 게 있을까?

아래 다섯 가지 operator를 소개합니다!
1. #map 🧑‍🍳
2. #filter 🕵
3. #flatMap 🚀
4. #debounce ⏳
5. #combineLatest 🤝

#### 🧑‍🍳 #map
map은 컨베이어벨트를 지나가는 재료인 데이터를 원하는 모양으로 바꾸는 재료 손질 담당입니다. map으로 들어온 데이터 하나를 받아서 → 새로운 데이터 하나로 변환해서 내보냅니다.

swift 문법을 공부하다보면 map, filter, reduce를 접해보신 적이 있을텐데요, 그 map과 정확히 같은 기능을 한다고 보시면 됩니다.

예를 들어 숫자 `1`, `2`, `3`이 컨베이어 벨트를 지나간다고 생각해봤을 때
map이라는 기계는 이 숫자들을 받아서 `"1번 사과"`, `"2번 사과"`,`"3번 사과"`처럼 문자열로 바꿔서 보낼 수 있습니다. 

```swift
import Combine

[1, 2, 3].publisher
    .map { number in
        return "Item: \(number)" // 숫자를 받아서 문자열로 변환
    }
    .sink { value in
        print(value) // "Item: 1", "Item: 2", "Item: 3" 이 차례로 출력됨
    }
```

##### map 사용 예시
- 서버에서 받은 숫자 코드(ex. 200)를 사용자에게 보내줄 메시지(ex. "성공")로 바꿀 때
- 사용자가 입력한 텍스트를 모두 대문자로 변환할 때

### 🕵 #filter
filter는 컨베이어 벨트를 지나가는 재료들 중에서 조건에 맞는 것만 통과시키는 품질검사원이라고 보시면 됩니다. 조건에 맞지 않는 데이터는 그냥 publisher가 발행했다고 해도 버려집니다.

위의 map도 map, filter, reduce에서 보셨던 거랑 같은 기능이라고 했는데, filter도 그러합니다.

1, 2, 3, 4, 5 사과가 벨트를 지나갈 때 짝수만 통과시키는 규칙을 가지고 2, 4만 통과시키고 1, 3, 5는 벨트에서 떨어져 나가도록 만들 수 있는 operator가 바로 filter입니다.

```swift
import Combine

[1, 2, 3, 4, 5].publisher
    .filter { number in
        return number % 2 == 0 // 짝수인지 검사해서 '참(true)'인 것만 통과
    }
    .sink { value in
        print(value) // 2, 4만 출력됨
    }
```

##### filter 사용 예시
- 사용자가 입력한 아이디가 5글자 이상일 때만 다음 단계를 진행하고 싶을 때
- 점수 목록에서 80점 이상인 학생들만 추려낼 때

### 🚀 #flatMap

flatMap은 한 종류의 publisher를 구독자에게 전송되는 새로운 publisher로 변환합니다.

들어온 데이터 하나를 가지고 새로운 컨베이어벨트를 만들어내는 연산자라고 생각하시면 되고, 주로 네트워크 요청처럼 시간이 걸리는 비동기 작업을 처리할 때 사용됩니다.

예를 들어보면 이렇습니다.
1. 사용자 아이디("msseock")가 벨트를 타고 flatMap에 도착하면
2. flatMap은 그 아이디를 받아서 "이 아이디로 서버에 프로필 정보를 요청해"라는 **새로운 작업**(새로운 publisher)을 시작합니다.
3. 그리고 그 새로운 작업이 끝나서 프로필 정보가 도착하면, 그 결과를 원래 벨트로 흘려보내줍니다.

⇒A라는 데이터를 받아서 B라는 새로운 비동기 작업을 실행하고, 그 B의 결과를 받아 처리하고 싶을 때 사용하면 됩니다!

```swift
import Combine
import Foundation

// 사용자 ID로 네트워크 요청을 해서 사용자 이름을 가져오는 가짜 함수
func fetchUserName(for id: String) -> AnyPublisher<String, Error> {
    // 실제로는 URLSession을 사용하겠지만, 여기서는 간단히 표현
    return Just("\(id)_name")
        .setFailureType(to: Error.self)
        .eraseToAnyPublisher()
}

Just("userABC")
    .flatMap { userId in
        return fetchUserName(for: userId) // ID를 받아서 새로운 Publisher(네트워크 요청)를 반환
    }
    .sink(receiveCompletion: { _ in }, receiveValue: { name in
        print(name) // "userABC_name" 이 출력됨
    })
```

##### flatMap 사용 예시
- 로그인 성공 후 받은 인증 토큰으로 사용자 정보를 요청할 때
- 검색어를 입력받아 그 검색어로 API를 호출하고 결과를 받아올 때

### ⏳ #debounce

debounce는 짧은 시간 동안 이벤트가 마구 쏟아져 들어올 때, 잠시 기다렸다가 **마지막** 이벤트 하나만 처리해주는(개인적으로 combine에서 가장 마음에 드는) 연산자입니다.

사용자가 검색창에 '스', '스마', '스맡', '스마트'라고 빠르게 입력하면 이벤트가 4번 발생하겠죠? debounce는 이 이벤트들을 바로 처리하지 않고, 0.5초 동안 추가 입력이 없으면 그때 마지막으로 처리할 수 있도록 기다리는 역할을 해줍니다. 결국 사용자가 '스마트'라고 입력하고 잠시 멈추면, 그때 딱 한 번 '스마트'라는 값만 다음 단계로 보내주는 것이죠

```swift
import Combine
import Foundation

let subject = PassthroughSubject<String, Never>()

subject
    .debounce(for: .seconds(0.5), scheduler: RunLoop.main) // 0.5초 대기
    .sink { value in
        print("API 요청: \(value)")
    }

// 사용자가 빠르게 입력하는 상황을 흉내
DispatchQueue.main.asyncAfter(deadline: .now() + 0.1) { subject.send("스") }
DispatchQueue.main.asyncAfter(deadline: .now() + 0.2) { subject.send("스마") }
DispatchQueue.main.asyncAfter(deadline: .now() + 0.3) { subject.send("스마트") }
// 0.3초에 마지막 입력 후 0.5초가 지나면(총 0.8초 시점) sink가 호출됨
// 결과: "API 요청: 스마트" 라고 한 번만 출력됨
```

##### debounce 이런 때 사용하세요
- 검색창 자동완성 기능(사용자가 타이핑을 멈췄을 때만 API 요청)
- ID 중복검사 자동요청
- 창 크기 조절이나 스크롤 이벤트가 끝났을 때만 특정 로직을 실행하고 싶을 때

### 🤝 #combineLatest

combineLatest는 두 개 이상의 Publisher를 동시에 지켜보다가, 어느 쪽이든 새로운 데이터가 들어오면 각각 '최신'데이터를 묶어서 보내주는 Operator

**아이디** 입력 벨트, **비밀번호** 입력 벨트 두 개가 있다고 했을 때,
combineLatest는 이 둘을 모두 지켜보다가 사용자가 아이디를 입력하면, `(최신 아이디, 최신 비밀번호)`를 묶어서 보내줍니다. 잠시 후 비밀번호를 입력하면, `(최신 아이디, 진짜최신 비밀번호)`를 묶어서 보내주는거죠.

어느 한쪽이라도 바뀌면, 양쪽의 가장 마지막 값을 조합해서 알려주는 겁니다.

```swift
import Combine

let usernamePublisher = PassthroughSubject<String, Never>()
let passwordPublisher = PassthroughSubject<String, Never>()

usernamePublisher
    .combineLatest(passwordPublisher)
    .map { (username, password) in
        // 아이디는 3글자 이상, 비밀번호는 8글자 이상일 때만 true
        return username.count >= 3 && password.count >= 8
    }
    .sink { isValid in
        print("로그인 버튼 활성화: \(isValid)")
    }

usernamePublisher.send("my")      // 출력: 로그인 버튼 활성화: false (비번이 아직 없음)
passwordPublisher.send("1234")    // 출력: 로그인 버튼 활성화: false
usernamePublisher.send("myID")    // 출력: 로그인 버튼 활성화: false
passwordPublisher.send("12345678") // 출력: 로그인 버튼 활성화: true
```

##### combineLatest 이런 때 사용하면 됩니다
- 로그인/회원가입 폼 유효성 검사: 아이디와 비밀번호가 모두 유효한 조건을 만족할 때만 '로그인' 버튼을 활성화할 때