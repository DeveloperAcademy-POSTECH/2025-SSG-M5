제출일: 2025-06-24

>[!question]
>GQ1. singleton 패턴이란 무엇일까?
>GQ2. Swift에서 singleton 패턴을 사용하는 경우에는 어떤 상황이 있을까?
>GQ3. Swift에서 singleton 패턴을 사용하는 이상적인 형태가 있을까?

## Description
- singleton 패턴이란 클래스의 인스턴스를 생성할 때 하나의 인스턴스만 생성하게 강제하는 구조를 말한다.
- 생성자를 private으로 지정하여 새로운 인스턴스 생성을 막는다.
- 인스턴스 제공 메소드를 제공해서 인스턴스가 생성되지 않은 최초의 경우에만 인스턴스 생성을 허용하고 나머지는 이전에 생성된 인스턴스를 반환하여 사용하도록 한다.
- 강제하고자 하는 인스턴스는 static으로 선언하여 사용하도록 한다.

## 주요 기능
- 생성자를 private으로 지정하여 새로운 인스턴스 생성을 막는다.
- 강제하고자 하는 인스턴스는 static으로 선언하여 사용하도록 한다.

## 코드 예시
~~~swift

class Circle{
	static let instance = Circle()//맨 처음에만 인스턴스의 생성이 일어남

	private init(){} //인스턴스 생성 막음

	static func getInstance(){
		return self.instance
	}
}

~~~

이런식으로 인스턴스의 생성을 한번만 하도록 강제하는 코드 형태를 말한다.

### 장점
- 인스턴스 생성의 무분별한 생성을 제한하여 **메모리 사용량을 줄여 성능향상**에 도움을 준다.
- 인스턴스 생성의 흐름을 제한하여 **코드의 가독성을 상승**시킬 수 있음
- 인스턴스 생성횟수를 줄이다 보니 **인스턴스 접근 시간이 줄어든다**
### 단점
- Singleton Instance가 너무 많은 일을 하거나, 많은 데이터를 공유시킬 경우 다른 클래스의 Instance들 간 **결합도가 높아지는  Open-Closed 원칙을 위배**할 수 있다. (의존성)
- 테스트가 어려워진다, 인스턴스 생성을 제한해버리면 **테스트용 Mock 객체를 만들어내기 어려워진다**. 이러면 관련된 로직을 테스트하기 어려워지고, 테스트 코드가 중요한 경우에는 꽤 곤란한 존재가 될 수 있다.

### Swift에서의 Singleton패턴
- obj-c나 java등의 언어에서 멀티스레드 환경에서 클래스에 동시에 접근할 시 객체가 2개 생성되는 위험이 발생할 수 있으나 Swift에서 static let으로 인스턴스를 강제해 놓으면 lazy하게 인스턴스가 생성되기에 swift에서 빛을 발하는 패턴이라고도 할 수 있다.
- 무슨 말이냐면 Swift에서 static let으로 선언하면 자동으로 해당 인스턴스가 사용되는 시점에서 생성되는 특성을 가지기에 다른언어에서처럼 단일성을 보장하기 위한 처리가 필요없다고 한다.

### Singleton이 사용되는 곳
~~~swift
let screen = UIScreen.main
let userDefault = UserDefaults.standard
let application = UIApplication.shared
let fileManager = FileManager.default
let notification = NotificationCenter.default
~~~
이러한 시스템안에서도 싱글톤을 이용하고 있다고 한다.
## Keywords
+ [[DesignPattern]]

## References
- Java 언어로 배우는 디자인 패턴 입문 - 유키 히로시 저
- https://babbab2.tistory.com/66

## 작성자
- steve
