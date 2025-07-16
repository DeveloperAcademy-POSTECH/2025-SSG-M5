>[!question]
>GQ1. Dependency injection은 어떤 상황에서 어떻게 적용할 수 있는가  
>GQ2. delegate pattern과 DI의 공통점과 차이점. 함께 사용할 수 있는가

@Sana
## Description
> SOLI ' D '
> 어떤 객체가 dependency를 직접 생성하지 않고 외부에서 주입받는 설계 방식.
> 하위 모듈이 상위 모듈로부터 독립되는 구조

왜 DI를 쓰는가
1. 종속성 감소 - 설계 유연성 증가
2. Mock 객체를 이용한 unit test 편해짐
3. 객체 모듈화 편해짐
![[Pasted image 20250603131156.png]]

## 주요 기능

| 방식             | 예시                                     |
| -------------- | -------------------------------------- |
| constructor 주입 | `init(service: ServiceProtocol)`       |
| property 주입    | `var service: ServiceProtocol?`        |
| method 주입      | `func setup(service: ServiceProtocol)` |


==GQ2. delegate pattern과 DI의 공통점과 차이점. 함께 사용할 수 있는가==

**공통점** - Protocol 기반 추상화, 객체간 의존성 낮음, 교체쉬움
--> 함께 사용 가능. delegate가 일종의 DI로 여길 수 있음

**차이점**

|     | Delegate Pattern   | Dependency Injection     |
| --- | ------------------ | ------------------------ |
| 목적  | 동작을 위임함 (callback) | dependency로 객체 구성 관계를 다룸 |
| 방식  | 단방향 이벤트 전달         | 객체를 제공, 양방향으로 사용 가능      |
| 역할  | 작업 대리 수행           | 객체 생성 통제                 |
| 대상  | delegate property  | 이것저것                     |



## 코드 예시

## Keywords
+ 파생된 키워드들을 작성
+ [[SOLID]]

## References
- https://www.youtube.com/watch?v=kF7rQmSRlq0
- https://www.swiftanytime.com/blog/dependency-injection-in-swift
- 