제출일: 2025-06-24

>[!question]
>GQ1. Value Semantics가 뭔데 Swift에서 중요한가
>GQ2. - Swift에서 struct가 효율적인 이유가 뭘까

## Description
> **Value Semantics**s는 데이터에 variable을 할당하거나 parameter로 전달할 때 **데이터값 자체를 복사**하는 방식

원본 instance와 복사된 instance가 독립적으로 동작하고, 메모리 효율저하의 원인이 되니까 swift에서는 Copy On Write로 프로세스를 최적화하고있다.

Swift는 modern language중 유일하게 Value Sementics를 디폴트로 삼은 언어이다.
(Java, Python등은 reference semantics. C++은 확장된 사용으로서의 struct)
## 주요 기능
`struct`, `enum`, `Array`, `String` 등이 Value Sementics 기반 type다.

1. Safety - 복사된/전달된 값이 변경되어도 호출자 변수는 변하지 않는다. 원본 보존!
2. Predictability - perfect 함수형 프로그래밍. 어느 값이 어떻게 쓰이는지 쉽게 추론가능함.
3. Concurrency Safety - 값을 여러 threads가 공유하지 않으니까 race condition이 없음
## 코드 예시
+ 실제 코드 예시를 작성

## Keywords
+ [[COW]]
+ [[Memory]]

## References
- 참고한 레퍼런스를 작성 (예 : Apple의 공식 문서)
- https://alexisgallagher.com/talks/valuesemantics/
- https://www.swiftbysundell.com/articles/utilizing-value-semantics-in-swift/
- GPTGPT

## 작성자
- Sana