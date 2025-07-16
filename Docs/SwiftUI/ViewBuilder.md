>[!question]
>GQ1. ViewBuilder는 무엇일까?
>GQ2. ViewBuilder는 언제 사용할까?
>GQ3. ViewBuilder는 언제 사용하면 안되는 걸까?

## Description

> A custom parameter attribute that constructs views from closures.

> 쉽게 말하면 ViewBuilder는 동적으로 다른 view를 만드는 클로저나 함수를 만들기 위해서 사용하는 도구이다...

- **ViewBuilder**는 클로저를 통해 여러 개의 자식 뷰를 구성할 수 있도록 해주는 custom parameter attribute 이다.
- 애플 도큐먼트 발췌...

# 예시코드

~~~swift
struct ContentView: View {
    @State var isOn = false

    var body: some View {
        VStack {
            Toggle("Toggle", isOn: $isOn)
            makeView(isOn: isOn)
        }
    }

    func makeView(isOn: Bool) -> some View {
        if isOn {
            return Text("Text")
        }else{
            return Button("Button") {}
        }
    }

	@ViewBuilder
	func fixedMakeiew(isOn: Bool) -> some View {
		if isOn {
			Text("Text")
		} else {
			Button("Button") {
				//some actions here...
			}
		}
	}
}
~~~

#### ⚠️ makeView 함수의 오류:
- Function declares an opaque return type 'some View', but the return statements in its body do not have matching underlying types
- (대충 함수가 불분명한 타입을 반환해서 안된다는 뜻)

#### 🔨 viewBuilder가 필요한 이유
~~~swift
@ViewBuilder
func fixedMakeiew(isOn: Bool) -> some View {
	if isOn {
		Text("Text")
	} else {
		Button("Button") {
			//some actions here...
		}
	}
}
~~~
- `some View`는 SwiftUI에서 불투명 타입(opaque type)으로, 구체적인 뷰 타입을 명시하지 않고 “어떤 뷰든 될 수 있다”는 의미를 가지지만 컴파일러는 함수 반환값으로 불투명한 타입을 허용하지 않음
- 즉 makeView()가 정상작동하려면 Text만 반환하거나 Button만 반환해야 함

- `@ViewBuilder`는 SwiftUI에서 뷰를 구성하는 클로저를 처리하는 특별한 속성으로 동작하며, 
  내부적으로 여러 타입의 뷰를 하나의 통합된 뷰로 묶어주는 역할을 함

#### 🫠 AnyView를 사용하는 방법도 이따...
~~~ swift
func fixedMakeiew(isOn: Bool) -> AnyView {
	if isOn {
		return Text("Text")
	} else {
		return Button("Button") {
			//some actions here...
		}
	}
}
~~~
- 이렇게 AnyView를 사용하는 방법도 있지만 이렇게 되면 반환되는 뷰의 타입이 지워지는 형태가 되기에 SwiftUI가 View를 관리하기 어려워 짐(성능에 문제가 생길 수 있음)
- 애플에서 권장하는 형태가 아님

## ✋ ViewBuilder를 사용하면 안되는 경우

1. 함수나 클로저가 단일 뷰를 반환하는 경우
	1. 코드 복잡도 증가(가독성, 유지보수성 저하)
	2. 성능에 문제가 생길수도 있음
2. 성능 최적화가 필요한 경우
	1. 업데이트가 많은 뷰를 ViewBuilder를 이용해서 개발하는 경우 성능 오버헤드가 발생할 수 있음

## Keywords
- [[SwiftUI]]

## References

https://developer.apple.com/documentation/swiftui/viewbuilder
https://phillip5094.tistory.com/222
 +ChatGPT's little help