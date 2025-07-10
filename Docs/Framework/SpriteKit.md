제출일: 2025-07-02

>[!question]
>GQ1. SpriteKit은 무엇일까?
>GQ2. SpriteKit은 어떻게 사용할 수 있을까?
>GQ3. SwiftUI로도 SpriteKit이 잘 작동할까?
## Description
### #SpriteKit

> Add high-performance 2D content with smooth animations to your app, or create a game with a high-level set of 2D game-based tools.
https://developer.apple.com/documentation/spritekit/ 

-> 앱에 고성능 2D 콘텐츠와 부드러운 애니메이션을 추가하거나, 고수준의 2D 게임 개발 도구를 사용해 게임을 만들어보세요.

결국 2D 디자인이랑 2D 게임 개발을 위해 만들어진 애플 공식 프레임워크다~

##### 특징
- **2D 도형, 파티클, 텍스트, 이미지, 비디오**를 그릴 수 있음
- **Metal**을 활용한 고성능 렌더링 지원
- 간단한 프로그래밍 인터페이스 제공
- 다양한 **애니메이션과 물리 효과** 내장
- 시각적 요소에 생동감과 부드러운 화면 전환 가능
- iOS, macOS, tvOS, watchOS에서 지원
- GameplayKit, SceneKit 등 다른 프레임워크와의 통합 용이
- visionOS에서 호환 앱에서는 사용 가능 (단, **visionOS 전용 앱에서는 권장하지 않음**)
https://developer.apple.com/documentation/spritekit/ 

## 주요 기능
#### 1. 캐릭터 이동 & 상호작용
- **`SKSpriteNode`**: 화면에 표시될 캐릭터나 객체를 나타냅니다.
- **`touchesBegan`, `touchesMoved`, `touchesEnded`**: `SKScene`에 구현하여 사용자의 터치 입력을 감지합니다.
- **`SKAction.move(to:duration:)`**: 현재 위치에서 목표 위치까지 부드럽게 이동하는 애니메이션을 만듭니다.
```swift
import SpriteKit

class GameScene: SKScene {
    // 1. 플레이어 스프라이트를 클래스의 프로퍼티로 선언한다.
    //    'private'으로 선언하여 이 클래스 내부에서만 접근 가능하도록 캡슐화하는 것이 좋다.
    //    이 코드가 정상 동작하려면 'player'라는 이름의 이미지가
    //    프로젝트의 Assets.xcassets에 추가되어 있어야 한다.
    private let player = SKSpriteNode(imageNamed: "player")
    ...
    // 씬(Scene)이 뷰(View)에 처음 표시될 때 호출되는 메서드이다.
    override func didMove(to _: SKView) {
        // 2. 씬의 배경색을 흰색으로 설정한다.
        backgroundColor = SKColor.white

        // 3. 플레이어의 위치를 설정한다.
        //    화면 너비의 10% 지점, 높이는 정중앙에 위치시킨다.
        player.position = CGPoint(x: size.width * 0.1, y: size.height * 0.5)

        ...

		// 9. 투사체가 이동할 액션을 생성한다.

        let actionMove = SKAction.move(to: realDest, duration: 2.0)
        // 10. 투사체가 이동 완료 후 씬에서 제거되는 액션을 생성한다.
        let actionMoveDone = SKAction.removeFromParent()

        // 11. 두 액션을 순서대로 실행하도록 시퀀스를 생성한다.
        let sequence = SKAction.sequence([actionMove, actionMoveDone])

        // 12. 투사체에 액션을 실행한다.
        projectile.run(sequence)
    }

    override func touchesEnded(_ touches: Set<UITouch>, with _: UIEvent?) {
        // 1. 여러 터치 중 첫 번째 터치만 처리한다.
        guard let touch = touches.first else { return }

        run(SKAction.playSoundFileNamed("pew-pew-lei.caf", waitForCompletion: false)

        ...
    }

	...
```
###### 출처
https://developer.apple.com/documentation/spritekit/skscene
https://developer.apple.com/documentation/spritekit/skaction

#### 2. 충돌 감지와 물리 시뮬레이션 (Collision Detection & Physics)
- **`SKPhysicsBody`**: `SKSpriteNode`에 물리적인 형태, 질량, 마찰력 등의 속성을 부여합니다.
- **`SKPhysicsContactDelegate`**: 두 `SKPhysicsBody`가 서로 접촉했을 때 이벤트를 받을 수 있는 프로토콜입니다.
- **`categoryBitMask`, `contactTestBitMask`**: 어떤 물리 객체끼리 충돌하고, 어떤 객체끼리 접촉 이벤트를 발생시킬지 정의하는 비트마스크입니다.
```swift

// 2.1 투사체의 물리체(Physics Body)를 설정한다.
	// 원형 물리체 생성
	projectile.physicsBody = SKPhysicsBody(circleOfRadius: projectile.size.width / 2) 
	// 투사체가 물리 엔진에 의해 영향을 받도록 설정
	projectile.physicsBody?.isDynamic = true 
	// 투사체의 카테고리 설정
	projectile.physicsBody?.categoryBitMask = PhysicsCategory.projectile
	// 몬스터와의 충돌 테스트 설정 
	projectile.physicsBody?.contactTestBitMask = PhysicsCategory.monster
	// 충돌하지 않도록 설정 
	projectile.physicsBody?.collisionBitMask = PhysicsCategory.none 
	// 정밀 충돌 감지 사용
	projectile.physicsBody?.usesPreciseCollisionDetection = true 

```

```swift
extension GameScene: SKPhysicsContactDelegate {
    func didBegin(_ contact: SKPhysicsContact) {
        // 1. 두 물체의 카테고리 비트 마스크를 가져온다.
        var firstBody: SKPhysicsBody
        var secondBody: SKPhysicsBody
        if contact.bodyA.categoryBitMask < contact.bodyB.categoryBitMask {
            firstBody = contact.bodyA
            secondBody = contact.bodyB
        } else {
            firstBody = contact.bodyB
            secondBody = contact.bodyA
        }

        // 2. 투사체와 몬스터가 충돌했는지 확인한다.
        if (firstBody.categoryBitMask & PhysicsCategory.monster) != 0 &&
            (secondBody.categoryBitMask & PhysicsCategory.projectile) != 0
        {
            // 투사체가 첫 번째 몬스터, 투사체가 두 번째 물체인 경우
            if let monster = firstBody.node as? SKSpriteNode,
               let projectile = secondBody.node as? SKSpriteNode
            {
                // 3. 충돌 처리 메서드를 호출한다.
                projectileDidColiideWithMonster(projectile: projectile, monster: monster)
            }
        }
    }
}
```
###### 출처
https://developer.apple.com/documentation/spritekit/skphysicsbody
https://developer.apple.com/documentation/spritekit/skphysicscontactdelegate

#### 3. 파티클과 셰이더를 이용한 특수 효과 (Special Effects)
- **`SKEmitterNode`**: 파티클(작은 입자)을 방출하여 불, 연기, 비, 눈과 같은 효과를 쉽게 만들 수 있습니다. Xcode 내의 시각적 에디터로 효과를 미리 디자인할 수 있습니다.
- **`SKShader`**: GPU에서 직접 실행되는 코드로, 픽셀 단위의 정교한 시각 효과를 구현합니다. 화면 왜곡, 색상 변환, 물결 효과 등에 사용됩니다.
```swift
import SpriteKit

class ParticleScene: SKScene {
    override func didMove(to view: SKView) {
        // 배경색을 어둡게 설정하여 파티클이 잘 보이게 합니다.
        backgroundColor = .darkGray
        
        // 1. 방금 만든 .sks 파일로부터 SKEmitterNode 인스턴스를 생성합니다.
        //    파일 이름이 일치해야 합니다.
        if let emitter = SKEmitterNode(fileNamed: "SnowParticle.sks") {
            // 2. 파티클이 화면 상단 중앙에서부터 시작되도록 위치를 잡습니다.
            emitter.position = CGPoint(x: frame.midX, y: frame.maxY)
            
            // 3. 씬에 파티클 노드를 추가합니다.
            addChild(emitter)
        }
    }
}
```
###### 출처
https://developer.apple.com/documentation/spritekit/skemitternode
https://developer.apple.com/documentation/spritekit/skshader

#### 4. 장면 전환과 게임 상태 관리 (Scene Transitions)
- **`SKScene`**: 게임의 각 화면(메뉴, 게임, 설정 등)을 별도의 `SKScene` 클래스로 구현합니다.
- **`SKTransition`**: 한 `SKScene`에서 다른 `SKScene`으로 전환될 때 사용되는 애니메이션 효과입니다. (예: `crossFade(withDuration:)`, `flipHorizontal(withDuration:)`)
- **`SKView.presentScene(_:transition:)`**: 현재 뷰에 새로운 씬을 지정된 트랜지션 효과와 함께 표시하도록 명령하는 메서드입니다.
```swift
...
func projectileDidColiideWithMonster(projectile: SKSpriteNode, monster: SKSpriteNode) {
        print("🏏 Hit!")
        // 1. 투사체와 몬스터를 씬에서 제거한다.
        projectile.removeFromParent()
        monster.removeFromParent()

        // 2. 파괴된 몬스터 수를 증가시킨다.
        monstersDestroyed += 1
        if monstersDestroyed >= 5 {
            let reveal = SKTransition.flipHorizontal(withDuration: 0.5)
            let gameOverScene = GameOverScene(size: size, won: true)
            view?.presentScene(gameOverScene, transition: reveal)
        }
}
...
```

```swift
let loseAction = SKAction.run { [weak self] in
            guard let self = self else { return }
            let reveal = SKTransition.flipHorizontal(withDuration: 0.5)
            let gameOverScene = GameOverScene(size: self.size, won: false)
            self.view?.presentScene(gameOverScene, transition: reveal)
        }
```
###### 출처
https://developer.apple.com/documentation/spritekit/sktransition
https://developer.apple.com/documentation/spritekit/skview/presentscene(_:transition:)

## Real 핵심 Keywords
+ **1. `SKScene` : 무대 (The Stage) 🎭**
	게임의 한 장면이나 레벨 전체를 담는 **최상위 컨테이너**입니다. 화면에 보이는 모든 것은 이 `SKScene` 안에 포함되어야 합니다
	
+ **2. `SKNode` : 객체 (The Actor)**
	씬을 구성하는 **가장 기본적인 빌딩 블록**입니다. 화면에 나타나는 캐릭터, 배경, 텍스트, 버튼 등 모든 요소는 `SKNode`입니다. 특히 이미지를 표시하는 **`SKSpriteNode`**가 가장 대표적이고 많이 사용됩니다.
	
- **3. `SKAction` : 동작 (The Script/Verb) 🏃**
	`SKNode`를 움직이게 하는 **'스크립트' 또는 '동사'**입니다. 노드를 이동시키고, 회전시키고, 크기를 바꾸거나, 사라지게 하는 등 모든 애니메이션과 시간 기반 로직을 담당합니다.
	
- **4. `SKPhysicsBody` : 물리 (The Laws of Physics) 🚀**
	노드에 질량, 중력, 마찰력, 탄성 등 **물리적인 특성을 부여**합니다. 이를 통해 노드 간의 충돌을 자동으로 감지하고, 현실적인 움직임을 시뮬레이션할 수 있습니다.
	
- **5. `SpriteView` : 캔버스 (The Window) 🖼️**
	`SKScene`을 화면에 그려주는 **'창문' 또는 '캔버스'** 역할을 하는 SwiftUI 뷰입니다. 
	(UIKit에서는 `SKView`가 이 역할을 합니다.)

#### 개인적으로 생각하는 아쉬운 점
1. 최신 업데이트가 거의 이제 확정적으로 중단되었음..
	그렇다고 Deprecate 될 느낌은 아님. 필수적인 프레임워크라서
2. 진짜 게임 만들거면 다른 엔진 쓰는게 무조건 이득임
3. 요즘 비전을 미는 추세라서 소외 받는 것 같아서 안타까움.

## 작성자
- #Wade