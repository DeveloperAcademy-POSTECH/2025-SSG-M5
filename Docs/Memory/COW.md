제출일: 2025-05-24

>[!question]
>GQ1. Copy-on-Write는 언제 발생하며, 어떻게 최적화에 기여하는가

## Description
> Swift의 Value Semantics가 실제 효용이 있기 위해서 중요한 기법.
> **Copy-on-Write**, value type에 수정(write)이 발생하는 순간 data가 copy된다.

struct를 모든 순간에 copy를 만들어두는거는 메모리효율, 연산코스트 등 면에서 **비효율**이다
그래서 
```
var a = [1, 2, 3, 4, 5]
var b = a  
```
이 때 말고
```
b[0] = 999 
```
이 순간에 변수 a는 b라는 copy가 생기는겁니다

 Swift의 standard container에서 *복사를 해야하는지 아닌지 판단하는 로직* 을 가지고있다
 
`isKnownUniquelyReferenced()`을 통해 ARC를 추적하고 reference count를 알아온다
reference count == 1
	이 객체는 나만 쓰고 있어 : 복사하지 않고 수정
reference count != 1
	다른 변수가 이 객체를 참조중 : 복사발생, 수정 저장

이게 Swift 언어 자체가 내장하고 있는 로직이라니! So cool..
+standard container와 compiler에서만 쓰는 함수는 아니고, 원한다면 class를 value type처럼 쓰기 위해 COW를 직접 구현해버릴 때 쓸 수 있다 
## Keywords
+ 파생된 키워드들을 작성
+ [[Memory]]

## References
- 참고한 레퍼런스를 작성 (예 : Apple의 공식 문서)
- https://developer.apple.com/documentation/swift/isknownuniquelyreferenced(_:)-98zpp

## 작성자
- Sana 