>[!question]
>GQ1. Metatype이 무엇이지?
>GQ2. self, .self, Self, .Type 이런게 다 무슨차이지?
>GQ3. 언제 쓰이는 거지?
>GQ4. 왜 Metatype 이라고 정의되는 거지?

## Description

###### Metatype이 무엇이지?

> A metatype type refers to the type of any type, including class types, structure types, enumeration types, and protocol types.

메타타입은 타입 그 자체를 나타내는 타입이다. 
이 타입에는 class, structure, enumeration, protocol 타입을 포함한다.

**타입 그 자체를 나타내는 타입이 무슨 말이지?**

예를 들어, Int는 5와 같은 정수 값을 담는 **타입**이다
그런데 이 Int라는 타입 자체를 값처럼 변수에 저장하려면, 그걸 감싸는 또 하나의 타입
즉, Int라는 "타입"을 나타내는 "타입"인 **Int.Type이 이와 같은 Metatype이다.**

아래 코드를 예시로 보자

```swift
struct Dog {
    static var species = "Canis familiaris"
    var name: String
}

```

먼저 Dog라는 **타입(Type)** 을 정의하고, name이라는 저장 프로퍼티를 선언했다.

```swift
let bori = Dog(name: "보리")
```

여기서 bori는 Dog 타입의 인스턴스이다

```swift
let boriType = type(of: bori)
```

인스턴스로 다룬 bori와 다르게, **boriType**은 bori의 타입, 즉 Dog라는 타입 그자체가 들어있다.
그럼 **boriType**의 타입은 Dog인가?
-> 아니다, Dog 타입은 인스턴스(bori)의 타입이지, 타입 자체의 타입이 아니기 때문이다.

**즉, 여기서 나오는 개념이 메타타입(Metatype)이다.**

```swift
let boriType: Dog.Type = Dog.self
```

여기서.self는 말 그대로 "타입 자체를 값으로 쓰겠다"는 뜻이고,
**Dog.Type**이 바로 **메타타입**이다. 즉, **boriType**은 Dog라는 타입을 저장하고 있고,
그 Dog라는 타입 자체의 타입은 Dog.Type
바로 타입의 타입 = **메타타입(Metatype)이다.**

1. Dog.self
	 .self를 붙이면 "이 타입 자체를 값으로" 쓰겠다라는 의미이다.
	 즉, Dog라는 타입이라는 것을 표현하는 하나의 값이다.
	 그냥 Dog 타입 그 자체를 지징할 뿐이다.

2. Dog.Type
	 .Type은 Dog라는 타입의 타입(Metatype)이다.
	 즉 Dog라는 타입 자체의 타입, 메타타입인 것이다.

**그렇다면 boriType을 어디에 사용하지?**

위에 작성된 코드 예시를 다시 한번 보자,

```swift
struct Dog {
    static var species = "Canis familiaris"
    var name: String
}
```
- speices는 종을 나타내는 타입 프로퍼티이고
- name은 저장 프로퍼티이다.

즉, 타입 프로퍼티에 접근하기 위해선 Dog 타입 자체를 가리키는 타입(메타타입)이 필요하다.

**Dog 타입 -> Dog로 생성한 인스턴스의 타입(인스턴스 멤버, name 사용 가능)**
**Dog 타입의 타입(메타 타입) -> Dog 타입 자체를 가리키는 메타 타입(타입 멤버 사용 가능)**

따라서, boriType은 타입 자체를 가리키기에, 인스턴스 프로퍼티인 name에는 접근이 불가능하다.

###### self, .self, Self, .Type 이런게 다 무슨차이지?
> 이제, 위에서 설명했던 메타타입의 개념을 되새기며 아래 차이를 보자

- self: 현재 인스턴스를 가리킨다. self.name, return self 등
```swift
struct Dog {
    var name: String

    func printName() {
        print(self.name) // self는 현재 인스턴스를 의미
    }
}
```

- .self: 타입 그 자체를 값으로 만들기 위해 사용한다.(Int.self, Dog.self)
```swift
let type: Int.Type = Int.self  // Int 타입 자체를 값으로 저장
```

- Self: 현재 타입 자체를 의미한다(주로 프로토콜, 제네릭에서 사용됨)
```swift
protocol Nameable {
    static func create() -> Self
}

struct Cat: Nameable {
    static func create() -> Self {
        return Cat()
    }
}
```
	 여기서 Self는 Cat 자체를 의미한다. 반환 타입이 Cat이다.

- .Type: 타입의 메타타입을 의미한다.
```swift
let t: String.Type = String.self
```
	Stirng.Type은 String이라는 타입의 메타타입을 의미한다.


###### 왜 Metatype이라고 정의되는 거지?

Meta란, **자기 자신을 설명하는 것**이라는 의미
즉, Metatype은 **타입을 설명하는 타입**을 의미하게 된다.

예를 들어서, 5라는 Int 타입이 있다고 가정해보자.
Int는 하나의 타입이다.
그런데, Int.self는 Int라는 타입을 값으로 표현한 것이고
그것의 타입은 Int.Type, 즉 Metatype이 되는 것이다.


## Keywords
+ 타입 프로퍼티
+ 저장 프로퍼티
+ [[Grammar]]

## References
- [개발자 소들이](https://babbab2.tistory.com/151)
- [Swift.org - Open Source Project](https://swift.org/)
- https://woozzang.tistory.com/160