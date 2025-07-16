---
tags:
  - swift
  - Framework
---

제출일: 2025-06-04

>[!question]
>GQ1. CloudKit을 사용할 때 추가적인 비용은 없을까?
>GQ2. 간단한 앱에서 데이터를 외부에서 불러와야할때 반드시 서버를 써야 될까?
# iCloud (CloudKit) 구현 준비 단계
---
## 1. 내 프로젝트에 iCloud 추가하기

![[2025-06-03_20-25-11.png]]![[2025-06-03_20-25-40.png]]

## 2. iCloud를 추가한 Provisioning Profile 만들어서 적용하기
![[2025-06-03_21-40-27.png]]![[2025-06-03_21-41-20.png]]
⚠️ 한 번 iCloud Container를 만들면 삭제가 불가능하니 이름 짓는 데에 신중을 가하기!

![[2025-06-03_21-49-15.png]]![[2025-06-03_22-52-57.png]]

# CloudKit을 사용해서 앱 설계하기
---

## 1. CloudKit 개요

- CloudKit은 애플이 제공하는 iCloud 기반 데이터 저장 및 동기화 프레임워크이다.
- 서버를 직접 구축하지 않아도, 앱 데이터를 안전하게 클라우드에 저장하고 여러 기기에서 동기화할 수 있다.
- Swift, SwiftUI, Objective-C, 웹 등 다양한 플랫폼에서 사용할 수 있다.

## 2. CloudKit의 핵심 구조

### 2.1 컨테이너(Container)
![[2025-06-03_22-13-10.png]]

- 컨테이너는 CloudKit에서 데이터의 최상위 저장 단위이다.
- 하나의 앱은 하나 이상의 컨테이너를 가질 수 있다.
- 컨테이너마다 데이터가 완전히 분리되어 있다.
- 일반적으로 하나의 앱은 하나의 컨테이너를 사용한다.

### 2.2 데이터베이스(Database)
![[2025-06-03_22-13-29.png]]![[2025-06-03_22-56-14.png]]

- CloudKit에는 세 가지 데이터베이스가 존재한다.
    - 퍼블릭 데이터베이스(Public Database): 모든 사용자가 읽을 수 있는 데이터베이스이다.
    - 프라이빗 데이터베이스(Private Database): 각 사용자만 접근할 수 있는 데이터베이스이다.
    - 쉐어드 데이터베이스(Shared Database): 특정 사용자와 데이터를 공유할 수 있는 데이터베이스이다.

### 2.3 존(Zone)
![[2025-06-03_22-14-34.png]]![[2025-06-03_22-14-48.png]]![[2025-06-03_22-57-17.png]]

- 데이터베이스 내에서 데이터를 논리적으로 구분하는 단위가 존(Zone)이다.
- 기본 존(Default Zone)과 커스텀 존(Custom Zone)이 존재한다.
- 커스텀 존을 사용하면 동기화 전략을 세밀하게 제어할 수 있다.
- 존 단위로 데이터를 동기화 할 수 있다. 훨씬 효율적임.

### 2.4 Schema (스키마)

**Record Types > Metadata**
![[2025-06-04_00-36-06.png]]

- **createdTimestamp**: 생성한 시각 (DATE/TIME)
- **createdUserRecordName**: 레코드 생성한 식별자. (REFERENCE)
- **___etag**: 레코드의 변경을 추적하기 위한 태그
- **modifiedTimestamp**: 수정한 시각 (DATE/TIME)
- **modifiedUserRecordName**: 마지막 수정한 사용자 (REFERENCE)
- **recordName**: 레코드 이름 (REFERENCE)

**Single Field Indexes**는 특정 레코드 타입의 필드를 인덱싱하여 검색 성능을 향상시키는 기능

- 인덱스는 데이터베이스에서 검색할 때 빠른 조회를 가능

**Index(인덱스)**
효율적인 데이터베이스 검색을 위해 사용하는 구조

**Add Index > Type**

**1. Queryable**
**설명**: Queryable 속성은 필드가 쿼리에서 조건으로 사용될 수 있도록 인덱싱하는 설정.
- **용도**: 특정 필드 값을 기준으로 레코드를 필터링하는 쿼리에서 사용.
- **예시**: 사용자의 나이를 기준으로 30세 이상인 사용자를 검색하는 쿼리.
```swift
let predicate = NSPredicate(format: "age >= %d", 30)
let query = CKQuery(recordType: "User", predicate: predicate)
```

**2. Sortable**
**설명**: Sortable 속성은 필드가 쿼리 결과를 정렬하는 데 사용될 수 있도록 인덱싱하는 설정.
- **용도**: 쿼리 결과를 특정 필드 값에 따라 오름차순 또는 내림차순으로 정렬하는 경우에 사용됩니다.
- **예시**: 사용자의 이름을 알파벳 순서로 정렬하는 쿼리.
```swift
let query = CKQuery(recordType: "User", predicate: NSPredicate(value: true))
query.sortDescriptors = [NSSortDescriptor(key: "name", ascending: true)]
```

**3. Searchable**
**설명**: Searchable 속성은 필드가 텍스트 검색에서 사용될 수 있도록 인덱싱하는 설정.
- **용도**: 텍스트 필드에서 키워드 검색을 수행하는 경우에 사용됩니다.
- **예시**: 사용자의 소개 글에서 특정 키워드를 검색하는 쿼리.
```swift
let predicate = NSPredicate(format: "self contains %@", "developer")
let query = CKQuery(recordType: "User", predicate: predicate)
```

**Security Roles**
![[2025-06-04_01-14-01.png]]

기본 제공되는 `_world`, `_icloud`, `_creator` 역할은 CloudKit의 공개 데이터베이스(Public Database) 접근 제어를 관리하는 핵심 메커니즘이다.

1. _world (모든 사용자)
    - 대상: iCloud 계정 유무와 관계없이 앱을 사용하는 모든 사용자.
    - 기본 권한:
        - 읽기(Read): 모든 레코드 타입에 대한 조회 권한이 기본적으로 부여됨.
        - 쓰기/생성(Write/Create): 기본적으로 비활성화되며, 보안상 절대 추가할 수 없음.
    - 사용 사례: 공개적으로 노출해야 하는 콘텐츠(예: 앱의 공지사항, 커뮤니티 게시물)에 적용.
2. _icloud (인증된 사용자)
    - 대상: 앱 사용 시 iCloud 계정으로 로그인한 사용자.
    - 기본 권한:
        - 생성(Create): 새 레코드 추가 가능.
        - 읽기/쓰기(Read/Write): 기본적으로 비활성화되며, 레코드 타입별로 명시적 허용 필요.
    - 특징: 생성 권한만 있을 경우, 사용자는 자신이 만든 레코드조차 조회/수정할 수 없으므로 `_creator` 역할과 조합해야 함.
3. _creator (레코드 생성자)
    - 대상: 특정 레코드를 직접 생성한 사용자.
    - 기본 권한:
        - 읽기/쓰기(Read/Write): 자신이 생성한 레코드에 대한 완전한 제어권.
        - 삭제(Delete): 쓰기 권한에 포함됨.
    - 동작 원리: `_icloud` 역할에서 생성 권한이 허용된 경우, 사용자가 레코드를 생성하면 자동으로 `_creator` 권한이 부여됨.

### 2.5. Subscriptions
![[2025-06-04_01-26-30.png]]

Subscriptions는 특정 조건에 맞는 레코드가 생성, 수정, 삭제될 때 사용자에게 알림을 보내는 기능. 이 기능을 통해 실시간으로 데이터 변경 사항을 감지하고, 사용자에게 알림을 제공할 수 있음.

Subscriptions는 다양한 유형이 있으며, 이를 통해 다양한 시나리오에 맞게 알림을 설정할 수 있음.

## 3. CloudKit 개발 시 고려사항
---
### 3.1 에러 처리

- CloudKit은 네트워크 기반이므로 다양한 에러가 발생할 수 있다.
- 한 번에 너무 많은 데이터를 전송하면 limitExceeded 에러가 발생한다.
- 데이터를 여러 번에 나누어 전송하는 것이 좋다.
- 데이터 충돌이 발생할 수 있으며, 일반적으로 "last writer wins" 전략을 사용한다.

### 3.2 권한과 보안

- CloudKit에는 World, iCloud, Creator 역할이 있다.
    - World: iCloud 미로그인 사용자도 포함하며, 기본적으로 읽기만 가능하다.
    - iCloud: iCloud 로그인 사용자로, 쓰기 권한을 부여할 수 있다.
    - Creator: 해당 레코드를 생성한 사용자만 수정할 수 있다.
- 퍼블릭 데이터베이스는 기본적으로 읽기 전용이다.

### 3.3 동기화 전략

- 오프라인 상태에서는 로컬에 데이터를 저장하고, 온라인이 되면 한 번에 동기화하는 전략이 필요하다.
- remote notification(원격 알림)을 활용하면 서버에서 데이터 변경 시 앱에 알릴 수 있다.
- 동기화 요청은 너무 자주 보내지 않도록 throttle(예: 2초에 한 번) 처리가 필요하다.

### 3.4 컨테이너 이름

- 컨테이너 이름은 보통 iCloud.com.회사이름.앱이름 형식이다.
- 직접 컨테이너를 생성한 경우 Apple Developer 사이트에서 등록해야 한다.

### 3.5 SwiftUI와 CloudKit

- SwiftUI에서는 CloudKit 데이터를 ObservableObject로 감싸서 상태 변화를 감지하게 할 수 있다.
- 네트워크 지연에 대비해 로딩 상태 표시와 실패 시 대체 UI를 준비해야 한다.

---

## 4. 추가로 알아두면 좋은 내용

- CloudKit은 퍼블릭 데이터베이스에 용량 제한이 있으므로 대용량 데이터 저장 시 주의가 필요하다.
- CloudKit JS나 Web API를 통해 웹앱에서도 동일한 데이터를 사용할 수 있다.
- 레코드 타입, 쿼리, 인덱스 등 데이터베이스 설계에 신경 써야 성능이 보장된다.

# 내 상황에 적용하기
---

내 상황: 공통적인 단어 데이터를 Public Database에 넣어서 모든 앱의 유저들이 해당 DB에 접근하여 모든 단어 데이터들을 불러올 수 있게 해야함.

## 세팅 순서

- Schema > Record Types > Add
- Schema > Indexes > Add
- Records > Query Records 생성, 데이터 추가

\[2025.06.18자 추가]
### CloudKit을 내 앱에 적용하기
노션 링크: https://kimhyeongi.notion.site/CloudKit-2072ce0304bd80bda138cc4176a0c077?source=copy_link 

## Keywords
[[Framework]]

# Reference
---
[https://developer.apple.com/documentation/cloudkit/](https://developer.apple.com/documentation/cloudkit/)
[https://developer.apple.com/documentation/cloudkit/deciding-whether-cloudkit-is-right-for-your-app](https://developer.apple.com/documentation/cloudkit/deciding-whether-cloudkit-is-right-for-your-app)
[https://developer.apple.com/documentation/cloudkit/enabling-cloudkit-in-your-app](https://developer.apple.com/documentation/cloudkit/enabling-cloudkit-in-your-app)
[https://developer.apple.com/kr/icloud/cloudkit/designing/](https://developer.apple.com/kr/icloud/cloudkit/designing/)
[https://rriver2.tistory.com/entry/iCloudKit-나도-백엔드-있다-시리즈-1-iCloud-세팅과-개념-그리고-유저-연결하기](https://rriver2.tistory.com/entry/iCloudKit-%EB%82%98%EB%8F%84-%EB%B0%B1%EC%97%94%EB%93%9C-%EC%9E%88%EB%8B%A4-%EC%8B%9C%EB%A6%AC%EC%A6%88-1-iCloud-%EC%84%B8%ED%8C%85%EA%B3%BC-%EA%B0%9C%EB%85%90-%EA%B7%B8%EB%A6%AC%EA%B3%A0-%EC%9C%A0%EC%A0%80-%EC%97%B0%EA%B2%B0%ED%95%98%EA%B8%B0)
[https://rriver2.tistory.com/entry/iCloudKit-나도-백엔드-있다-시리즈-2-iCloud-CRUD-해보기?category=996957](https://rriver2.tistory.com/entry/iCloudKit-%EB%82%98%EB%8F%84-%EB%B0%B1%EC%97%94%EB%93%9C-%EC%9E%88%EB%8B%A4-%EC%8B%9C%EB%A6%AC%EC%A6%88-2-iCloud-CRUD-%ED%95%B4%EB%B3%B4%EA%B8%B0?category=996957)