#Concurrency

제출일: 2025-06-05

>[!question]
>GQ1. Swift의 async, await키워드의 기본 개념과 역할을 이해해보자
>GQ2. 기존의 클로저 기반 비동기 처리 방식과 비교하여 async/await의 장점을 이해해보자
>GQ3. async/await을 사용해 네트워크 통신, 지연 작업등을 구현해보자
>GQ4. 그래서 async/await은 기존 방식이랑 비교했을 때 단점이 없을까?

## Description
- 비동기의 꽃인 `async/await`
- Swift `async/await`는 Swift 5.5부터 도입된 새로운 비동기 처리 방식
- 기존의 클로저 (`completionHandler`) 및 GCD (Grand Central Dispatch)을 사용하는 방식보다 코드가 더 읽기 쉽고 안전하게 비동기 처리 가능
- **가독성과 안정성, 에러처리 일관성, 동시성을 향상시켜주는 최신 비동기 프로그래밍**

## Keywords
+ `async`: 비동기 함수임을 나타냄. 작업 도중 일시 중단 (suspend) 가능
+ `await`: async 함수 호출 시 사용. 해당 함수가 완료될 때 까지 기다림
+ `Task`: 비동기 작업을 감싸는 단위

 ---
## 주요 기능 & 코드 예시

### ! 사전 필수 내용. 비동기란 무엇인가?
썸네일 이미지를 다운로드 하는 과정은 크게 4가지 작업으로 나눌 수 있다.
1. **thumbnailURLRequest** - URL을 통해 썸네일 이미지를 요청하기 위한 URLRequest를 생성
2.  **dataTask(with: completion:**) - URLSession의 dataTask 메서드에서 URLRequest에 대한 데이터를 가지고 오기
3. **UIImage(data:)** - 가지고 온 데이터를 UIImage의 init(data:)를 통해 이미지로 만들기
4. **prepareThumbnail(of: completionHandler:)** - 만들어진 이미지를 prepareThumbnail 메서드로 썸네일을 만들어서 화면에 보여주기
![[Pasted image 20250602005614.png]]

하지만 1 & 3번 작업은 빠르게 처리 되지만, 2 & 4번 작업은 시간이 좀 걸릴 수 있다. 특히, 2번 작업의 경우엔 이미지의 크기가 크다면 상당히 오래 걸릴 수 있음. 또한, 여러 개의 이미지 데이터를 다운로드 하려면 시간이 더 필요할 수 있음

	→ 따라서, 이미지를 다운로드 하는 동안 다른 작업을 수행하기 위해 비동기 코드를 작성해야함


### ! 사전 필수 내용:  completionHandler 이용해서 비동기 함수를 작성해보자
위의 요구사항을 기본 방식 (completionHandler)를 이용하여 코드를 작성해보면...
![[Pasted image 20250602014928.png]]
이렇게 모든 예외상황에 completion을 호출해줘야 어떤 상황이 발생하더라도 원하는대로 작업이 수행됨

위와 같이 에러를 포함한 정상 처리까지 총 5개의 completion 메서드 호출을 작성해야 원하는 작업을 만들 수 있어 코드가 지저분해짐

### GQ1. 그렇다면,`async` `await` 를 사용한다면?

일단 `async` `await` 이 무엇인지 알아보자

`async`
- 비동기 함수(`async` 함수)는 실행 도중 일시 중지(suspend)될 수 있는 특별한 함수이다.
- 비동기 함수는 필요할 때 실행을 멈췄다가 나중에 다시 재개(resume)할 수 있으며, 이러한 점에서 동시성 (concurrency)이 가능해진다.
- 비동기 함수를 선언하려면 함수 매개변수 목록 뒤에 `async` 키워드를 추가한다.
```swift
//예) 함수 선언 뒤에 async 선언 시 비동기 함수 선언
//비동기 실행 중 오류를 던질 수 도 있으므로 throws도 함께 명시
func fetchData(url: String) async throws -> String {

}
```

`await`
- `await` 키워드는 비동기 함수를 호출 할 때 사용
- `await`를 함수 호출 앞에 붙이면 그 함수의 완료를 기다리는 동안 현재 작업을 일시 중단한다.
- `await`지점에서 해당 비동기 함수가 끝날 때 까지 기다리되, 현재 쓰레드를 블로킹하지 않고 다른 작업이 그 쓰레드에 진행될 수 있다. 즉, `await` 는 현재 함수의 실행을 일시 정지시키지만 쓰레드 자체는 다른 작업에 양보한다.
```swift
let result = try await fetchData(from: "https://google.example.com)
```

### GQ2. 기존의 클로저 기반 비동기 처리 방식과 비교하여 async/await의 장점을 이해해보자
기존 방식인 클로저 기반의 비동기 처리는 아래와 같은 단점이 존재:
- 중첩 구조가 복잡해짐 ➡ 콜백 지옥 발생
- `completion(nil)` 등으로 에러 흐름 분기 필요
- 비동기 흐름 추적과 예외 처리 가독성 떨어짐
```swift
func downloadImage(completion: @escaping (UIImage?) -> Void) {
    let url = URL(string: "https://server.com/image.png")!
    URLSession.shared.dataTask(with: url) { data, _, _ in
        if let data = data {
            completion(UIImage(data: data))
        } else {
            completion(nil)
        }
    }.resume()
}
```

반면, `async` `await`을 사용한다면?
아래 코드는 위의 클로저 기반 코드와 완전히 동일한 역할을 한다.
- 코드가 직관적이며 에러 처리를 `try/catch`로 일관되게 처리 가능
- 마치 동기 코드 처럼 한줄 씩 흐름을 따라가기 쉬움
- 코드 재사용성과 유지보수 용이
```swift
func downloadImage() async throws -> UIImage {
    let url = URL(string: "https://server.com/image.png")!
    let (data, _) = try await URLSession.shared.data(from: url)
    
    guard let image = UIImage(data: data) else {
        throw URLError(.badServerResponse)
    }
    
    return image
```

### GQ3. `async` `await` 을 사용한 네트워크 작업 예제
Swift의 async/await은 URLSession의 비동기 API와 함께 사용할 때 가장 실용적이며, 기존의 클로저 기반 비동기 요청보다 **코드의 가독성, 흐름 제어, 에러 처리**에서 확실한 이점을 보여준다.
```swift
struct UserProfile: Codable {
    let id: Int
    let name: String
    let email: String
}

func fetchUserProfile() async throws -> UserProfile {
    // URL 생성
    guard let url = URL(string: "https://api.server.com/user") else {
        throw URLError(.badURL)
    }

    // 비동기 네트워크 요청 (await 지점)
    let (data, response) = try await URLSession.shared.data(from: url)

    // 응답 상태코드 체크
    guard let httpResponse = response as? HTTPURLResponse,
          (200...299).contains(httpResponse.statusCode) else {
        throw URLError(.badServerResponse)
    }

    // JSON → 모델 디코딩
    let decoded = try JSONDecoder().decode(UserProfile.self, from: data)
    return decoded
}

func loadUser() {
    Task { //- Task는 비동기 함수를 호출할 수 있는 컨텍스트
        do {
            let profile = try await fetchUserProfile()
            print("사용자 정보:", profile)
        } catch {
            print("사용자 정보 오류 발생:", error.localizedDescription)
        }
    }
}
```


### GQ4. 그렇다면 `async` `await` 의 단점은 없을까?
- iOS 15 이상에서만 사용 가능한거 말고 없다.

## Keywords
[[Concurrency]]

## References
- [WWDC21- Meet async/await in Swift](https://developer.apple.com/kr/videos/play/wwdc2021/10132)
- [Use async/await with URLSession](https://developer.apple.com/kr/videos/play/wwdc2021/10095)

## 작성자
- Jun