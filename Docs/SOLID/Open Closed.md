제출일: 2025-06-15

>[!question]
>GQ1. Swift의 Protocol은 SOLID 원칙과 어떻게 연결될까

## Description
>S ' O ' LID 
>object should be open for extension but closed for modification
>***Abstract method*** 를 추구하자.

- OOP 그 자체를 설명하는 수준의 basic concept
- 코드 다형성과 확장가능성 향상

## 코드 예시
```swift
// 예시 - 결제수단
protocol PaymentMethod {
    func pay(amount: Int)
}

class CardPayment: PaymentMethod {
    func pay(amount: Int) {
        // 어떻게든 신용카드 결제하는 코드
    }
}

class PaymentService {
    let method: PaymentMethod

    init(method: PaymentMethod) {
        self.method = method
    }

    func processPayment(amount: Int) {
        method.pay(amount: amount)
    }
}

// 애플페이를 추가한다면? ApplePay라는 타입을 추가해주면 반영끗
class ApplePay: PaymentMethod {
    func pay(amount: Int) {
        // 애플페이 연결해서 결제하는 코드
    }
}

```

## GQ1. Swift의 Protocol은 OCP와 어떻게 연결될까
*Abstraction에 의존한 코드* 를 짜기 정말 편한 수단 protocol 
1. function/method를 protocol로 define
2. class/struct로 구현(+확장) : 냅다 새로운 type를 추가하면 끝

## Keywords
- [[SOLID]]

## References
- https://inpa.tistory.com/entry/OOP-%F0%9F%92%A0-%EC%95%84%EC%A3%BC-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EB%8A%94-OCP-%EA%B0%9C%EB%B0%A9-%ED%8F%90%EC%87%84-%EC%9B%90%EC%B9%99

## 작성자
- Sana

