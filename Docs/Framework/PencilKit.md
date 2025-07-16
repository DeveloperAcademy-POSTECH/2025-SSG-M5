제출일: 2025-07-16

>[!question]
>GQ1. pencilKit으로 그린 그림은 vector type일까?
>GQ2. pencilKit으로 그린 그림을 공유하고 활용하기위해 어떻게 정제해야할까

## Description
PencilKit이란?
((조만간 추가될 예정))

PencilKit에서 사용되는 data type를 알아보자!
- Object: PKDrawing 
anyInput으로 Canvas에 그려진 선(PKStroke)의 집합. _immutable._ 하나의 그?림
- Struct: **PKStrok**e 
_point sequence._ PKInK: 사용된 툴(색깔, 펜종류)의 정보와 PKStrokePath를 저장한다.

==GQ1. pencilKit으로 그린 그림은 vector type일까?==
 - PKStrokePath
PKStrokePoint의 집합. 펜을 떼기 전까지 생성되는 모든 point를 **시간순서**대로 저장
*InterpolatedSlice << 포인트 간격 넓더라도(약한필압, 빠른드로잉) 데이터를 근사해서 부드럽게 표현

- **PKStrokePoint**
```swift
struct PKStrokePoint {
    var location: CGPoint      // 화면상 좌표
    var timeOffset: TimeInterval  // 스트로크 시작 후 시간(sec)
    var size: CGSize           // 획 두께
    var opacity: CGFloat       // 투명도
    var force: CGFloat         // Apple Pencil의 압력
    var azimuth: CGFloat       // 펜의 방향 
    var altitude: CGFloat      // 펜의 기울기 
}
```
stroke speed data는 저장되지 않음

> pencilKit으로 수집하는 데이터는  vector아니라 vector alike(PKStrokePoint의 집합이기에 확대해도 깨지지 않고, 선, 곡률, 좌표값등 수학적인 정보를 저장함) 독자 데이터타입이다

==GQ2. pencilKit으로 그린 그림을 공유하고 활용하기위해 어떻게 정제해야할까==
1. 표준 벡터 포맷(SVG, PDF 등)으로 직접 저장할 수 없다. << 호환되지 않는다
2. Apple 전용 바이너리 구조라서 Apple 생태계 내에서만 활용 및 내보내기가 가능하다(`dataRepresentation()`)

방법1. UIImage를 PDF로 렌더링 (`UIGraphicsBeginPDFContextToData()`)
방법2. UIImage를 SVG로 트레이싱 (Core Graphics, Core Animation, OpenCV)
방법3. `dataRepresentation()`으로 JSON 형식으로 변환
방법4. 애초에 PDF위에 Canvas를 얹는다 (`pdfRenderer`)
~~방법5. 그냥 Apple 환경에서만 사용한다~~

>PKDrawing은 벡터적인 품질을 가진 드로잉 데이터지만,실제 벡터 파일처럼 사용하거나 교환하는게 사실상 불가능하다

## Keywords
+ 펜슬킷 그 자체에 대하여. 투비컨티뉴

## References
- https://developer.apple.com/documentation/pencilkit/pkdrawing-swift.struct
- https://developer.apple.com/documentation/pencilkit/pkstroke-swift.struct

## 작성자
- Sana
- 