>[!question]
>GQ1. Protocol oriented language로서 Swift의 성격은 어떠한가
>GQ2. Protocol Oreiented programming은 대체 뭔가
>GQ3. Swift는 왜 POP를 채택했는가

@Sana
## Description

> Protocol을 통해 프로그램의 **인터페이스를 우선 설계**하고, 다양한 type에서 실제로 구현하거나 확장하는 programming paradigm. 
> 기능의 묶음을 Protocol로 abstraction. 그리고 conform하는 방식 >> 유연한 대규모 시스템 설계가능

"**Swift is a Protocol-Oriented Programming language**"
 *- WWDC15*

Objective-C의 한계를 극복하기 위한 시도. << 안전성, 성능 중심
1. **다중 상속 불가** 
2. **Class(Rreference type)의 한계** - value type를 더 적극적으로 차용하고싶음
3. **의존성 이슈** - Protocol + Extension으로 여러 타입에 공통 기능을 재사용


## 주요 기능
우선 Swift는 Multi Paradigm Programming Language이다. POP뿐만 아닌.

1. 기능 중심의 설계 
2. abstraction + combination으로 기능 객체를 구성
3. 구조체, 클래스, 열거형안에 실제 기능을 구현

## 코드 예시
OOP랑 POP
``` swift
// Object Oriented Programming
class Animal {
    func speak() {
        print("Animal sound")
    }
}

class Dog: Animal {
    override func speak() {
        print("Woof!")
    }
}
class Cat: Animal {
    override func speak() {
        print("Meow!")
    }
}

// Protocol Oriented Programming
protocol Animal {
    func speak()
}

struct Dog: Animal {
    func speak() {
        print("Woof!")
    }
}
struct Cat: Animal {
    func speak() {
        print("Meow!")
    }
}
```
## Keywords
+ 파생된 키워드들을 작성
+ [[Protocol]]

## References
- https://wwdcnotes.com/documentation/wwdcnotes/wwdc15-408-protocoloriented-programming-in-swift/