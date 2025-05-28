>[!question]
>GQ1. GQ를 쓰세요
>GQ2. GQ를 쓰세요

# 1. ARC란 무엇일까?
---

Swift에서는 앱의 메모리 사용을 추척, 관리할 때 `Automatic Reference Counting (ARC)` 를 사용한다.
더 이상 메모리를 사용 안한다고 판단되는 객체는 자동으로 `ARC`를 통해서 메모리로부터 free 시킨다.

# 2. ARC의 메모리 사용 판단 기준은 무엇일까?
---

특정 객체에서 메모리를 더 이상 사용되지 않는다는 것을 어떻게 `ARC`는 판단할 수 있는걸까?
`ARC`가 판단하는 기준은 특정 객체를 참조하는 변수, 상수, 프로퍼티들의 개수이다. 해당 개수들을 카운팅한다.

→ 특정 객체에 적어도 하나의 참조변수라도 존재한다면 ARC에서 메모리를 Free 시키지 않는다.
→ 반대의 경우, 해당 객체를 메모리 Free 시킨다.

# 3. ARC 간단 예제
---

```swift
class Person {
    let name: String
    init(name: String) {
        self.name = name
        print("\\(name) is being initialized")
    }
    deinit {
        print("\\(name) is being deinitialized")
    }
}
var reference1: Person?
var reference2: Person?
var reference3: Person?

reference1 = Person(name: "무찬이")
reference2 = reference1
reference3 = reference1
```

위 예제 코드를 보았을 때 `reference1 & 2 & 3` 를 통해서 무찬이 라는 이름을 가진 Person 객체의 reference count는 3이 된다.

하지만 다음 코드가 이후에 추가된다면 어떻게 될까?

```swift
reference3 = nil
reference2 = nil 
reference1 = nil
```

무찬이 Person 객체의 reference count는 reference1이 nil이 됨에 따라 0이 된다.

따라서 reference1이 nil이 되는 시점에서 메모리가 free 된다. (deinit 실행됨)

`→ 누구도 해당 instance에 접근하지 못한다고 판단되면 메모리에서 deallocate 시킨다.`

그렇다면, 누구도 객체에 접근하지 못하고, 사용하지 않더라도 deallocate가 이루어지지 않는다면?

→ Memory Leak 발생.

그 케이스를 바로 `“Strong Reference Cycle”`이라고 부른다.

# 4. Strong Reference 란 무엇일까?
---

Strong Reference란 변수, 상수, 프로퍼티 등으로 클래스 객체에 할당 시에 아래와 같은 경우를 의미한다.

```swift
var strongReference = SomeClass()
```

해당 reference가 남아있는 한 메모리로부터 Free 될 수 없기 때문에 Strong Reference라는 이름이 붙혔다.

# 5. 그럼, Strong Reference Cycle이란 뭔데?
---

만약, 두 개의 클래스 객체가 서로를 강하게 참조하고 있고, 서로를 참조하는 관계를 없애지 못한다면?

→ `Strong Reference Cycle`.

아래 예시를 봐보겠다.

```swift
class Person {
    let name: String
    var apartment: Apartment? // Person이 Apartment를 강하게 참조

    init(name: String) {
        self.name = name
    }

    deinit {
        print("\\(name) is being deinitialized")
    }
}

class Apartment {
    let unit: String
    var tenant: Person? // Apartment가 Person을 강하게 참조

    init(unit: String) {
        self.unit = unit
    }

    deinit {
        print("Apartment \\(unit) is being deinitialized")
    }
}

// 강한 순환 참조 발생
var john: Person? = Person(name: "John")
var unit4A: Apartment? = Apartment(unit: "4A")

john?.apartment = unit4A
unit4A?.tenant = john

// 모든 참조를 nil로 설정해도 deinit이 호출되지 않음
john = nil
unit4A = nil
```

john과 unit4A를 nil로 처리해줬으므로 예상하기로는 둘다 참조가 풀리면서 deinit이 이루어져야 할 것 같다.

하지만 결과는..?

`NOTHING..

![[2025-05-28_12-12-36.png]]
**nil로 설정해도 내부 참조가 해제되지 않는 이유**

`john = nil`과 `unit4A = nil`로 설정하면, 외부 변수(`john`과 `unit4A`)가 객체를 더 이상 참조하지 않게 됩니다. 하지만 객체 내부에서 서로를 강하게 참조하고 있기 때문에, ARC는 이를 "사용 중"이라고 간주하여 메모리에서 해제하지 않습니다.

이는 ARC가 객체 간의 **순환 참조를 자동으로 끊어주지 않기 때문**입니다. ARC는 단순히 참조 카운트를 기반으로 동작하며, 두 객체가 서로를 참조하고 있는지에 대한 관계를 이해하지 못합니다.

# 6. Memory Leak을 일으키는 Strong Reference Cycle을 어떻게 해결해줄 수 있을까?
---

결국 `ARC`가 Memory를 Free 시켜주기 위해서는 모든 Reference Count를 0으로 만들어줘야 ← FACTOS👀

**사례 1. 두 객체 간의 서로 참조 모두 해제**

```swift
var john: Person? = Person(name: "John")
var unit4A: Apartment? = Apartment(unit: "4A")

john?.apartment = unit4A
unit4A?.tenant = john

john?.apartment = nil
unit4A?.tenant = nil
john = nil // Person 해제
unit4A = nil // Apartment 해제
```

→ Result

**둘 다 deinit을 호출하게 된다!**

![[2025-05-28_12-13-11.png]]

근데.. 반드시 두 개 객체를 둘다 해제를 서로 시켜줘야만 메모리가 Free 될까?

**NOPE.**

**사례 2. 두 객체 간의 서로 참조 중 하나만 해제**

```swift
var john: Person? = Person(name: "John")
var unit4A: Apartment? = Apartment(unit: "4A")

john?.apartment = unit4A
unit4A?.tenant = john

john?.apartment = nil
// unit4A?.tenant = nil  하나는 해제해주지 않는다면?
john = nil
unit4A = nil
```

→ Result

**여전히 둘 다 deinit 호출!**

WHY?

![[2025-05-28_12-13-45.png]]

**여기까지의 결론:**

- 오! 결국 강한 참조 관계를 해제하려면 reference count를 0으로 만들어야 하는구나!
- 하나만 만들어도 연쇄적으로 count가 둘 다 0이 되기는 하네?
- 근데 개 귀찮네..?


# 7. 그렇다면, 아예 Reference Count를 증가시키지 않는 방법은 없을까?
---

있다. `weak`, `unowned`를 사용하는 것.

이 둘은 Strong Reference와는 달리 Reference Count를 증가시키지 않고 객체를 참조한다.

**7-1. `weak` 사용**

**특징**

- **참조 카운트를 증가시키지 않습니다.**
- 항상 **Optional**로 선언됩니다. (객체가 nil이 될 수 있기 때문)
- 객체가 메모리에서 해제될 경우, **자동으로 nil**로 설정됩니다.

```swift
class Person {
    let name: String
    var apartment: Apartment?

    init(name: String) {
        self.name = name
    }

    deinit {
        print("\\(name) is being deinitialized")
    }
}

class Apartment {
    let unit: String
    weak var tenant: Person? // 약한 참조 사용

    init(unit: String) {
        self.unit = unit
    }

    deinit {
        print("Apartment \\(unit) is being deinitialized")
    }
}

var john: Person? = Person(name: "John")
var unit4A: Apartment? = Apartment(unit: "4A")

john?.apartment = unit4A
unit4A?.tenant = john // 약한 참조로 순환 참조 방지

john = nil // Person 객체 해제
unit4A = nil // Apartment 객체 해제
```

**Result**

```swift
John is being deinitialized
Apartment 4A is being deinitialized
```

**장점**

- `weak` 참조는 객체가 해제되었을 때 자동으로 nil로 설정되므로 **안전성**이 높습니다.
- 강한 순환 참조를 방지하면서 Optional로 nil 검사를 통해 상태를 확인할 수 있습니다.

**단점**

- 항상 Optional로 선언해야 하므로, 값을 사용할 때마다 Optional Unwrapping이 필요합니다.

**7-2. `unowned` 사용**

**특징**

- **참조 카운트를 증가시키지 않습니다.**
- Optional이 아니며, **객체가 항상 유효하다고 가정**해야 합니다.
- 객체가 해제된 후에도 여전히 참조를 유지하므로, **객체가 해제되었는데 접근하면 런타임 에러**가 발생합니다.

```swift
class Customer {
    let name: String
    var card: CreditCard?

    init(name: String) {
        self.name = name
    }

    deinit {
        print("\\(name) is being deinitialized")
    }
}

class CreditCard {
    let number: Int
    unowned let customer: Customer // 비소유 참조 사용

    init(number: Int, customer: Customer) {
        self.number = number
        self.customer = customer
    }

    deinit {
        print("CreditCard \\(number) is being deinitialized")
    }
}

var john: Customer? = Customer(name: "John")
john?.card = CreditCard(number: 1234, customer: john!)

john = nil // 모든 객체 해제
```

**Result**

```swift
John is being deinitialized
CreditCard 1234 is being deinitialized
```

**장점**

- Optional Unwrapping이 필요하지 않으므로 코드가 더 간결합니다.
- 객체가 항상 유효하다고 가정할 수 있는 경우 유용합니다.

**단점**

- 객체가 해제된 상태에서 참조를 시도하면 **런타임 에러**가 발생합니다. (안전성이 낮음)
- 객체가 해제된 상태를 안전하게 처리하려면 개발자가 상태를 보장해야 합니다.

`unowned`는 `weak`와 달리, 다른 인스턴스와 수명이 같거나 더욱 긴 라이프타임을 가질때 사용됩니다.

weak와 달리 “**항상” 값이 있다는 전제**하에 사용됩니다.

그래서 결과적으로 optional을 만들어내지도 않고, ARC또한 자동적으로 `unowned`를 nil로 만들지 않습니다.

그리하여 unowned는 weak와 달리 system을 abort시킬수 있는 위험이 있기 때문에 주의해서 사용해야 합니다.

애플 문서에서도 이를 명확히 권고하고 있습니다

> 인스턴스가 메모리에서 해제 된 후, `unowned`를 통해서 값을 접근하면 runtime error를 발생한다. 그래서 **메모리에서 해제 되지 않은게 instance를 참조한다고 확신이 들 때에만 `unowned`를 사용해라!**


# 8. 왜 `unowned`라는 게 존재할까? 굳이?
---

`unowned`는 항상 메모리에 객체가 존재하는지 여부를 확인해줘야 해서 사용하기가 불편하다는 것을 알게됐다.

근데 Swift에서 `unowned`를 만든 이유는 무엇일까?

> Swift는 안전성이 떨어지는 `unowned`를 제공함으로써, **성능상의 이유 등으로 runtime에 안정성 체크를 disable 시키는 경우를 위해** 만들었다.

**결론:**

1. **퍼포먼스**

결국, ARC에서 nil처리를 직접 해줘야 하는 사실은 결국 런타임중 해야할 작업이 늘어나는 것입니다.

그리하여 안전하게 사용할 수 있다면, 퍼포먼스면에서 조금 더 우위에 있을수 있는 점이 이유인 것 같습니다.

1. **가독성**

unowned의 기본적인 전제는 “항상” 값이 있다입니다.

이는 클래스 내부의 값을 가져올때 옵셔널 처리를 하지 않아도 된다는 장점이 있습니다.

**하지만, 우리는 조금의 퍼포먼스 보다는 오래 동작해도 안전한 앱을 지향하기 때문에 weak를 더 많이 사용한다!**

# 9. weak, unowned로 객체 선언하는 건 어떨까?
---

```swift
class Job {
    weak var name: Name?

    deinit {
        print("deinit Job")
    }
}

class Name {
    var number = 5

    deinit {
        print("deinit Name")
    }
}

var a: Job? = Job()
a?.name = Name()
print(a?.name)
```

이 경우의 출력값은 어떻게 될까?

```swift
deinit Name
nil
```

참조하는 객체가 존재하지 않고, unowned는 “값이 있다”는 전제하에 해당 값을 참조하기 때문에, 프로그램은 에러를 발생시키며 종료됩니다.

이렇듯 unowned, weak는 해당 객체를 가지는게 불가능합니다. 따라서 Strong을 통해서 누군가가 해당 값을 참조하고 있을 경우에만 “유효”한 값을 얻을 수 있습니다.


# 앞으로 더 알아볼 부분..!
---
https://yudonlee.tistory.com/37
https://yudonlee.tistory.com/38

> **ARC 2탄에서 계속 됩니다...**


# Reference
---
https://developer.apple.com/videos/play/wwdc2021/10216/