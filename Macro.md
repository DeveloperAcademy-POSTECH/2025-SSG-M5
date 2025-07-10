제출일: 2025-07-03
>[!question]
>GQ1. Swift Macro가 무엇인가?
>GQ2.Swift에서 기본으로 제공해주는 매크로는 무엇이 있을까?
>GQ3.매크로를 내가 직접 만들 수 있는가?
>GQ4. 매크로를 사용하면 어떤 장점이 있으며, 주의할 점은?

## Description
- Macro는 Swift 5.9부터 도입되었고, Xcode 15부터 사용 가능하다. (macro: 거시적인, 크거나 대규모임을 나타냄)
- 매크로는 특정 입력이 어떤 방식으로 대치되어 출력되어야 하는지를 정한 규칙, 혹은 패턴을 의미한다 = "큰" 블럭의 코드를 "작은" 일련의 문자열로부터 만들어 낼 수 있기 때문에 "매크로"라고 불린다.

## 주요 기능
### GQ1. Swift Macro가 무엇인가?
Swift Macro는 "컴파일 시점에 코드를 생성하고 수정하는 코드"이다. 
개발자가 작성한 코드를 분석하여, 정해진 규칙에 따라 반복적인 코드를 대신 작성해주거나, 코드에 새로운 기능을 추가하고, 오류를 미리 검사해주는 자동화 도구이다.

### GQ2. Swift에서 기본으로 제공해주는 매크로는 무엇이 있을까?
Swift 개발 편의성을 위해 매크로들이 내장되어 있다. 대표적인 매크로는 다음과 같다:
1. `@Observable`
	- 클래스를 '관찰 가능'하게 만들어, 프로퍼티 값이 변할 때 SwiftUI뷰가 자동으로 업데이트 되도록한다
	- 기존의 `ObservableObject`와 `@Publihsed`를 대체하는 더 간결하고 효율적인 상태 관리 도구이다. 
```swift
//기존의 방식인 ObservableObject를 사용하는 방식
class CounterViewModel: ObservableObject{
	@Published var count = 0

	func increment(){
		count += 1
	}
}

struct ContentView: View{
	@ObservedObject private var viewModel = CounterViewModel()

	var body: some View{
		Text("Count: \(viewModel.count)")
		Button("Increment){
			viewModel.increment
		}
	}
}
```

```swift
import Observation

@Observable
class CounterViewModel{
	var count = 0
}
```

2. `@Model`
	- `@Model`은 SwiftData에서 데이터베이스 모델을 정의하는 방식이다. 
	- 코드가 간편해 CoreData를 대체하기 좋다

```swift
import CoreData 

public class Note: NSManagedObject { 
	@NSManaged public var content: String? 
	@NSManaged public var creationDate: Date? 
} 

let fetchRequest: NSFetchRequest<Note> = Note.fetchRequest()
```

```swift
import SwiftData

@Model
class Note{
	var content: String
	var creationDate: Date
	
	init(content: String, creationDate: Date){
		self.content = content
		self.creationDate = creationDate
	}
}

@Query var notes : [Note]
```
Swift 클래스 처럼 직관적으로 데이터 모델을 정의할 수 있어 개꿀

3. `#URL`
	- `#URL`은 문자열로 URL을 생성할 때 발생할 수 있는 런타임 오류를 컴파일 시점에 방지한다

```swift
func createURL() -> URL?{
	let urlString = "https://swift.org"
	guard let url = URL(string: urlString) else {
		print("유효하지 않는 URL임")
		return nil
	}
	return url
}
```

```swift
func createURL() -> URL{
	// #URL 매크로는 컴파일 시점에 URL 유효성을 검사
	// 유효하지 않으면 컴파일 에러가 발생하므로, 런타임에 nil이 될 가능성이 없음
	// 따라서 반환 타입은 non-optional(URL)이며, 옵셔널 처리 코드가 불필요
	return #URL("https://swift.org")
}
```

4. `#Predicate`
	- `#Predicate`는 데이터를 필터링 하는 조건을 만들 때, 오타 및 타입 불일치 문제를 해결한다

```swift
import SwiftData

@Model
class Note {
	var content: String
	var isUrgent: Bool
	var creationDate: Date

	init(content: String, isUrgent: Bool, creationDate: Date){
		self.content = content
		self.isUrgent = isUrgent
		self.creationDate = creationDate
	}
}

let searchKeyword = "Swift"
let newPredicate = #Predicate<Note> { note in
	note.isUrgent == true && note.content.contains(searchKeyword)
}

//만약 note.isUrgnent라고 오타를 내면, Xcode가 즉시 "value of type 'Note' has no member 'isUrgnent'" 라는 컴파일 에러를 보여준다
```


### GQ3. 매크로를 내가 직접 만들 수 있는가?
있다. 스크린샷이 너무 많아지는 관계로 블로그 보고 따라하도록... ㅈㅅ
[https://ios-development.tistory.com/1490]

### GQ4. 매크로를 사용하면 어떤 장점이 있으며, 주의할 점은?
**장점**
1. 코드 중복 감소: `@Observable`처럼 반복적으로 작성해야 하는 상용구 코드를 단 한줄의 매크로로 대체할 수 있다. 이를 통해 코드의 전체적인 양이 줄어듬
2. 가독성 향상: 매크로로 의도를 명확하게 드러낼 수 있다. 예를 들어, 클래스 위에 `@Model`한줄로만으로도 '이 클래스는 데이터베이스에 저장되는 모델이다' 라는 사실을 누구나 쉽게 파악할 수 있다.
3. 타입 안정성 강화: 매크로는 컴파일 시점에 코드를 생성하고 즉시 유효성 검사를 한다. `#URL` 및 `#Predicate`처럼 문자열 기반의 작업에서 발생할 수 있는 오타나 타입 불일치 오류를 런타임이 아닌 컴파일 시점에 잡아내여 앱의 안정성을 높여준다

**단점**
1. 빌드 시간 증가: 매크로는 컴파일 과정에서 추가적인 코드 생성 및 분석 단계를 거친다. 따라서 복잡한 매크로를 프로젝트 전반에 과도하게 사용하면 전체적인 빌드 시간이 길어질 수 있다
2. 과용 주의: 매크로가 편리하다고 해서 모든 것을 매크로로 해결하는 것은 좋은 습관이 아님. 간단한 함수나 제네릭으로 해결하는 것이 코드를 이해하고 테스트하기에 더 간단하고 명확해질 수 있음.

## Keywords
+ Macro

## 작성자
- Jun
