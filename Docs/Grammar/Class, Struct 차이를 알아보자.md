>[!question]
>GQ1. Swift에서는 class 대신 struct를 더 빈번하게 사용하는데 struct를 자주 사용하는 이유가 뭘까?
>GQ2. class와 struct를 각각 사용하는 순간들은 언제일까?
>GQ3. class와 struct의 공통점과 차이점은 뭘까?
>GQ4. 어떤 점이 class와 struct의 공통점과 차이점을 만들까?

# 배경 지식
---

**기초적인 쓰레드 & 메모리 지식을 알고 있으면 좋습니다.**

[[Threads & Memories]]

**기초적인 ARC 지식을 알아두면 좋습니다.**

[[ARC(Automatic Reference Counting)]]


# Struct는 Stack에, Class는 Heap에 저장된다.

Struct → 할당/해제가 상대적으로 빠름 + Thread Safe

Class → Heap에 할당되어 오버헤드 발생 + Thread Safe

- 오버헤드(Overhead)
    
    “실제 작업에 **직접적으로 기여하지 않지만**, 작업을 지원하기 위해 **불가피하게 발생하는 추가 비용(시간, 메모리 등)**“입니다.
    
    Swift를 포함한 모든 프로그래밍에서 오버헤드를 최소화하는 것이 성능 최적화의 핵심입니다.
    
    **Swift 실무에서 오버헤드의 실제 사례**
    
    - 클래스의 힙 할당: 클래스 인스턴스는 힙에 할당되며, 이 과정에서 메모리 할당/해제, 참조 카운팅, 동기화 등 다양한 오버헤드가 발생합니다.
    - 참조 카운팅(ARC) 오버헤드: 클래스 인스턴스의 참조가 변경될 때마다 카운트 증가/감소, 원자적 연산 등 추가 비용이 발생합니다.

# Class와 Struct의 Method Dispatch 방식 차이

Class, Struct의 Method Dispatch 방식 차이 → 성능 최적화와 직접적인 연결

1. 기본 동작 차이

Class

- 상속 가능성 → Dynamic Dispatch(동적 디스패치) 사용
    
- 가상 메소드 테이블(vtable-virtual method table)을 통해 런타임에 메소드 구현체를 탐색한다
    
- Example Code:
    ```swift
    class Animal {
      func sound() { print("울음소리") } // vtable[0]
    }
    
    class Dog: Animal {
      override func sound() { print("멍멍") } // vtable[0]
    }
    
    let myPet: Animal = Dog()
    myPet.sound() // 런타임에 Dog의 sound() 호출 결정
    ```
    

Struct

- 상속 불가 → Static Dispatch(정적 디스패치) 사용
    
- 컴파일 타임에 메소드 직접 연결
    
- Example Code:
    ```swift
    struct Point {
      var x, y: Double
      func draw() { /* 구현 */ } // 컴파일 시점에 주소 고정
    }
    
    let p = Point(x: 0, y: 0)
    p.draw() // 바로 Point의 draw() 호출
    ```
    

**Dispatch 참고하기!**
[[Dispatch 비교]]

# 메모리 할당 방식 차이로 인한 성능 차이

Struct - 값 타입 → Stack에 저장

Stack → Static Memory Allocation 된다.

- 스택 자체 크기가 컴파일 타임에 결정됨.
    
    ```swift
    // 정적 메모리 할당 (스택)
    // 값 타입: struct
    
    struct Point {
        var x: Double
        var y: Double
    }
    
    func example() {
        let point1 = Point(x: 0, y: 0) // 스택에 메모리 할당
        var point2 = point1            // 값 복사(별도 인스턴스)
        point2.x = 5                   // point1.x는 여전히 0
        // point1, point2는 독립적
    }
    ```
    
- 매우 빠름. → LIFO(Last In First Out) 방식으로 작동해서 O(1)로 할당, 해제 가능
    
- Stack에 쓰이기 때문에 Thread Free하다.
    
    - 각 쓰레드마다 별도의 스택 메모리 영역이 할당됩니다.
    - 함수 호출 시 지역 변수, 매개변수, 반환 주소 등이 이 스택에 저장됩니다.
    - 스택은 해당 쓰레드만 접근할 수 있으므로, 다른 쓰레드와 메모리를 공유하지 않습니다.

Class - 참조(Reference) 타입 → Heap에 저장

참조 타입은 Heap에 값을 저장하고 이를 가리키는 주소값을 Stack에 저장한다.

## 왜 참조 타입에서 오버헤드가 더 크게 발생할까?

### <할당 해제 시>

Heap 영역 메모리는 물리 메모리에 연속적으로 존재하는 것이 아님.

즉, Heap은 Stack처럼 연속적인 메모리 공간에 순차적으로 할당되는 것이 아니라 여러 페이지를 할당해 가상 주소 공간에 매핑하는데, 이때 페이징 작업을 통해 할당되게 된다.

또한, 추가적으로 페이지 테이블에서 주소값 변환에 대한 데이터를 저장하는 작업 역시 필요하다.

Heap에서 메모리를 해제한다면 MMU를 업데이트 해줘야 하기도 한다.

- **MMU(Memory Management Unit)**는 CPU와 물리 메모리 사이에서 가상 주소를 물리 주소로 변환하고, 각 프로세스가 자신만의 독립적인 메모리 공간(스택, 데이터, 힙 등)에만 접근하도록 보호하는 역할을 한다.

### <값 접근 시>

스택은 지정된 값에 바로 접근할 수 있다.

힙은 할당된 데이터에 바로 접근할 수 없고, Class의 인스턴스가 저장되어 있는 힙 메모리의 주소를 통해 접근해야 해서(역참조) 오버헤드가 스택보다 더 걸리게 된다.

주소값을 통해서 접근하는게 스택보다 더 오래 걸릴 이유는 뭐야?

- MMU로 메모리 범위 검사
- 가상 메모리 주소 → 물리 메모리 주소로 변환 (페이지 테이블 검사)
- 캐싱 작업

→ 이 모든 것이 오버헤드로 발생하게 된다!

# 참조 카운트 오버헤드 줄이기를 통한 성능 개선

엥? 그러면 그동안의 내용을 봤을 때 당연히 Class보다 Struct를 쓰는게 성능이 더 좋은거 아닌가? 라고 할 수 있다.

하지만 마냥 그렇진 않다.

> Class & Struct 비교 2편에서 계속..

[[Class, Struct 비교 분석 마무리]]

---

## Keywords
+ [[Grammar]]
# Reference

[https://developer.apple.com/videos/play/wwdc2016/416/](https://developer.apple.com/videos/play/wwdc2016/416/)
[https://developer.apple.com/documentation/swift/choosing-between-structures-and-classes](https://developer.apple.com/documentation/swift/choosing-between-structures-and-classes)
[https://hasensprung.tistory.com/181](https://hasensprung.tistory.com/181)