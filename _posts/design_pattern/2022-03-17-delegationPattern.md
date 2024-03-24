---
title: Delegation Pattern (위임패턴)
titleEn:
author: BabyK
date: 2022-03-17
category: Design_Pattern
layout: post
---
한 객체가 자기자신을 다른 객체에게 넘겨준 후 자신을 위해 일하게 만드는 패턴.  
현재 화면에서 이전화면의 상태를 바꾸는 가장 손쉬운 방법으로  
뷰를 사용한 화면 전환이라던지 버튼 클릭과 같은 이벤트 처리에서 주로 사용됨.  
<br>

```swift
protocol DiceGame {
    func play()
}

protocol DiceGameDelegate: AnyObject {
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

GameBoard와 GameBoardDetail을 두 개의 화면으로 보면 GameBoard 에서 다음 화면인  
Detail 로 넘겨준 self 객체를 가지고 Detail 화면에서 주사위를 굴리는 로직을 실행함.  
GameBoard 화면에서는 본인 (주사위)을 넘겨주며 실행요청만 하고 정작 주사위 Roll 로직은  
GameBoardDetail 에서 구현됨.  

두 객체의 강한연결로 인한 메모리 누수를 막기위해 weak 으로 선언, 약하게 결합하여  
GameBoard가 객체가 사라졌을 때 GameBoardDetail 객체가 메모리에 남아있는 상황을 방지.  

현재 화면 (지금 사용중인 객체)이 아닌 다른 객체에 있는 기능을 사용하려 하거나 (주로 버튼입력)  
이전 혹은 다음 화면의 상태를 현재 화면에서 조작 하려 할 때 주로 사용된다.  

일종의 Composition 패턴으로, 상속보다는 느슨하게 다른 객체를 자신의 프로퍼티로 선언하여 사용함.  


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