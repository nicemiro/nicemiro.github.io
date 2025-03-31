---
title: Delegation Pattern (위임패턴)
titleEn:
author: BabyK
date: 2022-03-17
category: Design_Pattern
tags: [DesignPattern]
layout: post
---

OOP가 등장하고 상속의 편리함을 누리던 개발자들은 슬슬 상속된 클래스들의 문제점을 발견하기 시작했다.  

* 상위 클래스에서 상속되는 불필요한 코드들
* 기능이 추가된에 따라 점점 비대해지는 상위 클래스
* 높은 커플링으로 인한 리팩토링의 어려움

현재 대세는 상속대신 컴포지션과 위임패턴을 사용하는 방향으로 바뀌었고 추가되는 클래스를 프로퍼티로 저장하거나 자바의 인터페이스와 같은 추상화 방식으로 커플링을 줄여나고 있다.  

### 대리자에게 Self 객체를 넘겨준다
현재 화면에서 이전화면의 상태를 바꾸는 가장 손쉬운 방법으로 뷰를 사용한 화면 전환, 버튼 클릭과 같은 이벤트 처리에서 주로 사용된다. 단순하게 표현하면 현재 화면객체가 이전 화면객체의 주소값을 포인터로 들고 있다고 보면 되는데 양쪽을 각각 delegator(위임자), delegate(대리인)로 구분지어 다음과 같이 기술한다.  
##### '위임자 객체가 자기자신을 대리자 객체에게 넘겨준 후 자신을 위해 일하게 만드는 패턴'

<br>
#### 예시
```swift
protocol DiceGame {
    // Sending Object의 시작지점
    func play()
}

protocol DiceGameDelegate: AnyObject {
    // Receiving Object(Delegate) 가 실행할 로직
    func gameDidStart(_ game: DiceGame)
    func rollDice(_ game: DiceGame, _ dice: [Int])
    func gameDidEnd(_ game: DiceGame)
}

// SENDING OBJECT
class GameBoard: DiceGame {
    weak var delegate: DiceGameDelegate?
    let dice = [1,2,3,4,5,6]
    
    func play() {
        delegate?.gameDidStart(self)
        delegate?.gameDidEnd(self)
        delegate?.rollDice(self, dice)
    }
}

// RECEIVING OBJECT
class GameBoardDetail: DiceGameDelegate {
    private var numberOfTurns = 0
    
    func gameDidStart(_ game: any DiceGame) {
        print("gameDidStart, currentTurn : \(numberOfTurns)")
    }
    func gameDidEnd(_ game: any DiceGame) {
        print("gameDidEnd.")
        numberOfTurns += 1
    }
    func rollDice(_ game: any DiceGame, _ dice: [Int]) {
        print("game logic is running. currentTurn : \(numberOfTurns)")
        let randomNum = dice.randomElement() ?? 1
        print("Dice num in this turn is \(randomNum.description)")
    }
}
```
<br>

'GameBoard' 와 'GameBoardDetail' 을 두 개의 화면으로 보자면  
첫 화면인 'GameBoard' 가 넘겨준 self 객체를 가지고 'GmaeBoardDetail' 화면에서 주사위 로직을 실행.  
'GameBoard' 화면에서는 본인을 넘겨주며 실행요청만 하고 정작 주사위 Roll 로직은 'GameBoardDetail' 에서 구현됨.  

두 객체의 강한연결로 인한 메모리 누수를 막기위해 `weak` 키워드로 'GameBoard' 가 메모리 해제되는 시점에 'GameBoardDetail' 객체가 메모리에 남아있는 상황을 방지함. 현재 객체상에서 다른 객체를 생성 할때 self(포인터)를 넘겨준다는 것이 중요하다.  

더 단순하게 표현된 코틀린 예시.  

```kotlin
interface ClosedShape {
    fun area(): Int
}

class Rectangle(val width: Int, val height: Int) : ClosedShape {
    override fun area() = width * height
}

// The ClosedShape implementation of Window delegates to that of the Rectangle that is bounds
class Window(private val bounds: Rectangle) : ClosedShape by bounds
```