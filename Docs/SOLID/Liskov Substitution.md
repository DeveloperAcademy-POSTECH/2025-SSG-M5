제출일: 2025-06-29

>[!question]
>GQ1. Swift의 Protocol은 SOLID 원칙과 어떻게 연결될까

## Description
>SO ' L ' ID 
>every **subtype** should be **replaceable** for their **parent.**
>**Consistency** between parent and child types

LSP ~= 다형성
*" parenttype의 행동 규약을 subtype가 위반하면 안된다 "*

근데 이걸 위배하기도 쉽지가 않다

### subtype가 잘못된 overriding을 하는 경우
1. **parent의 method signature을 임의로 변경**
``` swift
class Animal {
    func makeSound() -> String {
        return "뭔가 string"
    }
}

class Dog: Animal {
    // return type 을 임의로 변경 → LSP 위배
    override func makeSound() -> Int {
        return 123
    }
}```
2. parent type의 method를 다른 의도로 overriding 
	``` swift
class Bird {
    func fly() {
        print("I can fly!")
    }
}

class Penguin: Bird {
    override func fly() {
        // 논리적으로 잘못된 상속을 받는 케이스
        // error는 아닌데 틀린거지 펭귄이 어케납니까
        print("I can fly!") 
    }
}```

## GQ1. Swift의 Protocol은 LSP와 어떻게 연결될까
- `protocol`은 **행동의 규약**을 정의하고, 다양한 타입이 이를 준수하도록 강제한다
- 다형성을 `class` 상속 없이 **struct, enum, class 모두에 적용**할 수 있는 방식 제공
- `associatedtype`, `Self`, `where` 등을 활용해 **타입 안정성과 일관성** 확보 가능

 그러나 잘못 overriding하는 2번 케이스는 잡아낼 방도가 없다. (runtime error?)
--> 논리적 오류가 없는 protocol을 잘 선택하는 외에 방법은 없다


## References
- 참고한 레퍼런스를 작성 (예 : Apple의 공식 문서)

## 작성자
- Sana