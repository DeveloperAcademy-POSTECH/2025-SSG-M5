제출일: 2025-06-29

>[!question]
>GQ1. Swift의 Protocol은 SOLID 원칙과 어떻게 연결될까

## Description
>SOL ' I ' D 
>rather than large monolithic interfaces, **multiple small role-specific interfaces** better.
>Clients should not be depend on **methods they don't use.**

interface **유지보수성**과 **코드 가독성**을 위해 클래스에 적합한 메소드만을 상속시켜주자
'interface' 단위의 SRP(단일책임원칙)이라고 볼 수 있다. 

설계를 잘해야하는게 interface는 **추상화된** 개념이라 보유 메소드 **수량 제한**이 없다.
![[스크린샷 2025-06-30 오후 7.29.00.png]]

interface는 상황과 맥락에 따라 분리하는거라 방법론은 없고, 유관한 기능끼리 일정 규모 이하로 사이즈를 제한하라는 원칙이다.

## GQ1. Swift의 Protocol은 LSP와 어떻게 연결될까
**역할 분리**와 **작은 책임 단위 설계**가 중요하다. --> Protocol을 **작게 쪼개**고, 구체적인 역할에 따라 다양하게 **조합**하기. 애초에 이게 protocol의 목적임!
Swift는 다중 protocol 채택이 가능하니까 ISP설계? very ez.
``` swift
// 안좋은 예시: fat protocol
protocol Device {
    func call()
    func sendText()
    func playMusic()
}```
```swift
protocol Callable {
    func call()
}

protocol Textable {
    func sendText()
}

protocol MusicPlayable {
    func playMusic()
}
```
이렇게 ISP+POP로 잘 분리해두면
``` swift
struct iPhone: Callable, Textable, MusicPlayable { ... }
struct AppleWatch: Callable, MusicPlayable { ... }
```
이렇게 필요한거만 잘 조합하면된다구!

## Keywords
- protocol composition`(&)`

## References
- 참고한 레퍼런스를 작성 (예 : Apple의 공식 문서)
- https://inpa.tistory.com/entry/OOP-%F0%9F%92%A0-%EC%95%84%EC%A3%BC-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EB%8A%94-ISP-%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4-%EB%B6%84%EB%A6%AC-%EC%9B%90%EC%B9%99

## 작성자
- Sana