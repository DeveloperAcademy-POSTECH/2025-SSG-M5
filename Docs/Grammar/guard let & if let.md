
> [!NOTE] Question
> GQ1. guard let과 if let은 어떤 차이를 가지고 있을까?
>GQ2. guard let을 알아보자
>GQ3. if let을 알아보자

## Description
- ==guard let과 if let 공통점==
	- guard let과 if let 모두 Optional Binding 이라는 특징을 가지고 있음
	- Optional Binding은 Swift에서 옵셔널(Optional) 값을 안전하게 꺼내서 사용하는 방법
	- Optional은 값이 있을 수도 있고, nill일 수도 있고, 없을 수도 있는 타입으로, 직접 접근시 위험
	- 안전하게, 값이 있는지 확인한 후 꺼내서 사용하는 방식이 Optional Binding
	- Optional Binding을 통해 Unwrapping해야 안전함


- ==guard let과 if let 차이점==
  **guard let**
	- 사용 목적 : 조건이 안 맞으면 조기 탈출(early exit)하고 싶을때 사용
	- 바인딩 범위 : guard 이후의 전체 범위에서 사용 가능
	- 가독성 : 흐름을 직관적으로 유지 (실패 먼저 처리)
	- else 필요 여부 : else는 필수 (guard는 조건이 실패하면 무조건 탈출해야 함)
	
	**if let**
	- 사용 목적 : 조건이 맞을때만 내부에서 실행하고 싶을 때
	- 바인딩 범위 : if 블록 안에서만 사용 가능
	- 가독성 : 조건적 흐름에 적합
	- else 필요 여부 : else는 선택


- ==guard let을 알아보자==
	- guard 안의 조건문이 참(true)이 아니면 else문이 실행되는 문법
	- 조건이 true일때 코드가 계속 실행
	- 전역변수로 사용 가능
	- 조건이 false일 때, else문이 실행되며 상위 코드 블럭을 종료하는 함수 반드시 필요
	  (contiue, break return, throw 등)


- ==if let을 알아보자==
	- if let은 성공과 실패 2가지 경우로 나뉘며, 2분기로 모두에 원하는 코드 작성 가능
	- 지역변수(scope 안에서만 유효한 변수)로만 사용 가능
	- 값이 있을때 특정 코드를 실행하고, 값이 없는 경우(nil)에 다른 처리 가능



## 주요 기능
==guard let이 좋은 코드가 되는 조건
1. 실패 조건을 먼저 처리하고 싶은 경우 : 조기 탈출(early exit)을 통해 코드 흐름을 간결하게 만듬
2. 중첩을 피하고 싶은 경우 : if let으로 여러 단계를 중첩하는 대신 평평한 구조로 유지
3. 바인딩한 변수를 이후에도 계속 써야 할 경우 : guard let으로 바인딩하면 그 이후 범위에서도 사용 가능
4. 에러 처리, 유효성 검사, 비동기 로직의 첫 단계일 때 : 실패한 입력을 초기에 거르고, 성공 흐름을 아래에 깔끔히 유지
5. Swift의 선언적 스타일에 맞게, 흐름을 위헤서 아래로 읽히게 할 때 : 예외 먼저 처리 → 정상 흐름은 들여쓰기 없이 아래로 전개


==guard let 사용시 좋은 코드 1 : 사용자 입력 검정(Form validation)==
- guard 로 인해 잘못된 값일 땐 바로 return
- 유효한 값일 때만 아래 줄에서 name, ageInt 를 깔끔하게 사용
```swift
func validateForm(name: String?, age: String?) {
    guard let name = name, !name.isEmpty else {
        print("이름을 입력해주세요.")
        return
    }

    guard let age = age, let ageInt = Int(age), ageInt > 0 else {
        print("유효한 나이를 입력해주세요.")
        return
    }

    print("사용자 정보: \(name), \(ageInt)세")
}
```

==guard let 사용시 좋은 코드 2 : onAppear 내에서 데이터 안전하게 처리==
- guard let 으로 nil일 때 바로 탈출
- 이후 코드는 들여쓰기 없이 깔끔하게 처리 가능
```swift
struct ProfileView: View {
    var user: User?

    @State private var nickname: String = ""

    var body: some View {
        Text("닉네임: \(nickname)")
            .onAppear {
                guard let user = user else {
                    print("유저 정보 없음")
                    return
                }

                // 이후 user 안전하게 사용 가능
                nickname = user.nickname
            }
    }
}
```


==if let이 좋은 코드가 되는 조건==
1. 단일 또는 소수의 Optional을 간단하게 바인딩할 때
2. Optional이 nil일 경우 별다른 처리 없이 무시할 수 있을 때 : else가 선택적인 경우
3. 조건 만족시에만 간단한 작업을 하고 싶을 때 :  UI 조건 처리에서 간단한 뷰 분기 로직에 사용할 때


==if let 사용시 좋은 코드 1 : 사용자 이름이 있을 때만 인사==
- 이름이 없으면 아무 일도 하지 않음
- 이름이 있을 때만 특정 동작 → 깔끔하게 적절한 if let 사용
```swift
var userName: String? = "수민"

if let name = userName {
    print("안녕하세요, \(name)님!")  // name은 이 블록 안에서만 사용됨
}
```


==if let 사용시 좋은 코드 2 : 뷰 조건적으로 표시할 때 (SwiftUI)==
- 뷰 로직에서 조건적으로 처리할 때 if let은 연스러운 선택
```swift
struct ContentView: View {
    var optionalImageName: String?

    var body: some View {
        VStack {
            if let imageName = optionalImageName {
                Image(imageName)
            } else {
                Text("이미지가 없습니다.")
            }
        }
    }
}
```


==if let 사용시 좋은 코드 3 : 단일 옵셔널 바인딩 후 사용==
- 두 개의 옵셔널 바인딩을 한줄로 간결하게 처리
- 조건 충족 시 블록 실행, 실패 시 무시 → guard보다 적절한 상황
```swift
let userInput: String? = "42"

if let value = userInput, let number = Int(value) {
    print("변환된 숫자: \(number)")
}
```



## 코드 예시
 ==guard let 예제==
 - name이 nill이면 바로 탈출(return)
 - 값이 있으면 이후 코드를 간결하게 작성 가능
 - 값이 없을때 탈출하고 이후 깔끔하게 처리하고 싶을때 guard let 사용
 - 중첩을 줄이고, 흐름을 위에서 아래로 직관적으로 유지할 수 있는 장점 존재
 - ViewController, async, form validation, network handling에서 자주 사용
``` swift
fun greet(name: String?) {
	guard let unwrappedName = name else {
		print("이름이 없습니다.")
		return
	}

	print("안녕하세요, \(unwrappedName)님!")
}
```

==if let 예제==
- name이 nil이 아닐 때만 unwrappedName 사용 가능 (if 블록 내부에서만 가능)
- 조건을 만족할 때만 잠깐 쓰고 끝낼 때 사용
```swift
func greet(name: String?) {
	if let unerappedName = name {
		print("안녕하세요, \(unwrappedName)님!")
	} else {
		print("이름이 없습니다.")
	}
}
```



## Keywords
+ optional
+ nil
+ let
+ guard
+ else
+ [[Grammar]]

## References
- https://jud00.tistory.com/entry/%EC%98%A4%EB%8A%98%EC%9D%98-Swift-%EC%A7%80%EC%8B%9D-if-let-%EA%B3%BC-guard-let%EC%9D%98-%EC%B0%A8%EC%9D%B4%EB%8A%94
- https://dvlpr-chan.tistory.com/12
- 코딩하는 체대생 : https://mini-min-dev.tistory.com/21