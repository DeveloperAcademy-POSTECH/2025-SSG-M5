#Combine
Combine에서는 subscriber를 직접 구현하지 않아도, Publisher의 Output 및 Failure 유형과 자동으로 일치하는 두 개의 기본 제공 구독자를 제공한다!

#### **sink**
: Publisher가 보내주는 값을 직접 작성한 코드로 처리

1. `sink(receiveCompletion:receiveValue:)`
2. `sink(receiveValue:)`
```swift
// 이런 식으로 쓰면 됩니다
$namePublisher
	.sink { name in
		print("입력된 이름은 \(name)입니다")  // 값을 직접 받아서 처리
	}
```

##### sink subscriber를 사용해서 publish된 텍스트값으로 검색 수행하기
```swift
class SearchViewModel: ObservableObject {
	@Published var searchText = ""
	@Published var searchResults: [String] = []
	@Published var isLoading = false
	
	private var cancellables = Set<AnyCancellable>()
	private let searchService = SearchService()
	
	init() {
		$searchText
			.debounce(for: .milliseconds(300), scheduler: RunLoop.main)
			.removeDuplicates()
			.sink { [weak self] searchText in
				self?.performSearch(searchText)
			}
			.store(in: &cancellables)
	}
	
	private func performSearch(_ text: String) {
		// 검색 로직
	}
}
```

#### assign
값을 자동으로 특정 객체의 속성에 대입(assign) 해주는 방식

1. `assign(to:on:)`
2. `assign(to:)`
```swift
// 이런 식으로 쓰면 됩니다
$isLoginEnabledPublisher
	.assign(to: \.isEnabled, on: loginButton)  // 값이 바뀔 때마다 버튼 상태 업데이트
```

##### assign subscriber를 활용해 두 가지 값이 모두 입력되었을 때 뷰 상에서 버튼 활성화시키기 예제
```swift
class LoginViewModel: ObservableObject {
	@Published var username = ""
	@Published var password = ""
	@Published var isLoginButtonEnabled = false

	private var cancellables = Set<AnyCancellable>()

	init() {
		Publishers.CombineLatest($username, $password) 
			.map { !$0.0.isEmpty && !$0.1.isEmpty } 
			.assign(to: &$isLoginButtonEnabled)
	}
}
```