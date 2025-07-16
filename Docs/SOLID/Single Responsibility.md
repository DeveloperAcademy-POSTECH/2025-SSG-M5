제출일: 2025-06-15

>[!question]
>GQ1. Swift의 Protocol은 SOLID 원칙과 어떻게 연결될까

## Description
>' S ' OLID 
>Single Class Object owns **single functional responsibility** 
>Module need to modified in **only one reason**

![[스크린샷 2025-06-15 오후 2.36.20.png]]
- 프로젝트 전체의 유지보수성 상승
- 코드 자체의 가독성? 이해도 상승 
## GQ1. Swift의 Protocol은 SRP와 어떻게 연결될까
" 하나의 Class가 *한 가지 기능* 만 갖는다면 ***굳이 Abstraction*** 필요한가? "
" *재사용* 되지 않을건데, ***overengineering***이지 않을까 ""

Answer >> 맞는듯함. 단일클래스로 구현하는게 공수덜들고 합리적인듯함

**GQ2. 그럼 굳이굳이 프로토콜을 쓰려면?** 
- 인터페이스를 우선적으로 설계하는 방식 
	기능들만 우선 구현해두고 상위 class에서 조립하기
```swift
// 예시 - 회원가입 서비스
protocol UserSaving {
    func save(user: User)
}

protocol EmailSending {
    func sendWelcomeEmail(to user: User)
}

class LocalUserSaver: UserSaving {
    func save(user: User) {
        // 뭔가 유저정보 저장하는 코드
    }
}

class SimpleEmailSender: EmailSending {
    func sendWelcomeEmail(to user: User) {
        // 어떻게든 환영 메일 쓰고 보내는 코드
    }
}

class SignUpService {
    let saver: UserSaving
    let emailSender: EmailSending

    init(saver: UserSaving, emailSender: EmailSending) {
        self.saver = saver
        self.emailSender = emailSender
    }

    func signUp(user: User) {
        saver.save(user: user)
        emailSender.sendWelcomeEmail(to: user)
    }
}
```
- Unit test, Mocking 필요한 경우
	결국 Protocol의 외부의존성 격리라는 툴이 강력하기에 지속적인 테스트가 필요한 경우 데려갈만함

## Keywords
- [[SOLID]]

## References
- https://inpa.tistory.com/entry/OOP-%F0%9F%92%A0-%EA%B0%9D%EC%B2%B4-%EC%A7%80%ED%96%A5-%EC%84%A4%EA%B3%84%EC%9D%98-5%EA%B0%80%EC%A7%80-%EC%9B%90%EC%B9%99-SOLID#%EB%8B%A8%EC%9D%BC_%EC%B1%85%EC%9E%84_%EC%9B%90%EC%B9%99_-_srp_single_responsibility_principle

## 작성자
- Sana

