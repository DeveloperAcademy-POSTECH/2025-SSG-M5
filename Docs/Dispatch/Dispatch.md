>[!question]
>GQ1. Dispatch는 무엇일까?
>GQ2. Static Dispatch란 무엇일까?
>GQ3. Dynamic Dispatch란 무엇일까?
>GQ4. Static과 Dynamic의 차이는 무엇일까?

## Description

###### Dispatch는 무엇일까?

어떤 함수/메서드 구현체를  **"컴파일 타임"** or "**런타임**"에 선택할지 결정하는 방식이다.

Swift에서는 **비동기 작업 처리(GCD)와 메서드 호출 방식**모두에서 ‘Dispatch’라는 용어가 사용되기 때문에
두 용어가 혼동되지 않도록 알아두자.

- DispatchQueue의 Dispatch는 "**작업을 실행할 위치(큐/스레드)를 결정하는 것**"
- Swift 문법에서 Method Dispatch는 "**어떤 함수/메서드를 호출할 때 어떻게 실행될지 결정하는 과정**"

###### Static Dispatch는 무엇일까?

"**컴파일 타임**"에 호출될 함수를 결정하여, 런타임 때 그대로 실행한다. -> Direct Call이라고도 한다.
정적으로 결정되기 때문에, 빠르다는 장점이 있다.

주로 **final, private, struct, enum** 등에서 사용된다.

위 특징은 주로 Value Type에서의 Dispatch는 Static Dispatch로 호출된다는 것이다.
예시처럼, struct, enum 등은 상속을 할 수 없다는 특징 때문에 오버라이딩 될 가능성이 없기 때문이다.

```swift
struct Animal {
    func sound() {
        print("Animal sound")
    }
}
```

sound란 함수가 어디서 불려도, **Animal 구조체 안의 함수가 불릴 것이 보장되기에**
런타임에 따로 추적할 필요가 없어서 컴파일 타임에 결정되는 것이다.

###### Dynamic Dispatch는 무엇일까?

어떤 함수/메서드를 호출할지 런타임에 결정한다.
class 타입, 특히 오버라이딩이 가능한 메서드에서 사용한다.
따라서, 클래스마다 함수 포인터들의 배열, 즉, **vTable(Virtual Method Table)이라는 것을 유지**한다.
하위 클래스가 메서드를 호출할 때, vTable을 참조하여 실제 호출할 함수를 결정한다.
이 과정들이 "런타임"에 결정 되기 때문에 성능상 손해가 일어난다.

```swift
class Animal {
    func speak() {
        print("Animal sound")
    }
}

class Dog: Animal {
}
```

위처럼, Animal이란 클래스에서 speak라는 함수를 만들었다. 
Dog라는 클래스에서 Animal을 상속받고 있지만, speak 메서드를 오버라이딩 하지 않았다.

```swift
let puppy: Animal = Dog()
puppy.speak()
```

이 경우 Animal이란 타입으로, Dog 인스턴스를 가리켰을땐 speak가 오버라이딩 되지 않아, Animal 내의
speak 메서드가 반드시 호출될 것이다.

그렇다면 아래와 같은 경우는 어떨까?


```swift
class Animal {
    func speak() {
        print("Animal sound")
    }
}

class Dog: Animal {
    override func speak() {
        print("Bark")
    }
}

let myPet: Animal = Dog()
myPet.speak() // "Bark"
```

myPet은 Animal 타입이지만, Dog 타입을 업캐스팅 했기 때문에, Animal의 speak가 아니라 Dog 클래스의 speak()를 참조해야 한다.

이처럼 컴파일러는 메서드가
하위 클래스에서 오버라이딩 될 경우를 대비하여, 상위 클래스의 speak일지, 하위 클래스 Dog의 speak일지 확인해야 "**런타임**"때 확인하는 작업이 필요하다.

따라서, speak이라는 함수는 각 클래스마다 가지고 있는 vTable 안에 함수 포인터로 두고
실제 런타임 시점에 해당 포인터를 찾아 실행될 함수를 선택한다.

정리하자면, 런타임 시점에 Dog라는 클래의 vTable을 탐색하여 
실제 불릴 speak 함수를 찾아 실행하는 것이다.


#### Summary

정리하자면 Static Dispatch와 Dynamic Dispatch는 오버라이딩 여부와 관련된 것이다.
Value Type에서는 컴파일 시점에 즉각 실행되기에 성능상 이점이 있고
Reference Type에서는 런타임 시점에 vTable을 통해 해당 주소를 찾고, 점프해야 하는 작업이 필요하기에
성능상 손해가 발생한다는 것이다.

또한, DispatchQueue와 분명히 다른 점을 알아두자.

## 


## References
- https://babbab2.tistory.com/143
- ChatGPT
