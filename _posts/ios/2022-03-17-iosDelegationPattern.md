---
title: Delegation Pattern (위임패턴)(1)
titleEn: Delegation Pattern in iOS(1)
author: BabyK
date: 2022-03-17
category: iOS
layout: post
---

### Delegation Pattern
Delegation (위임) 이란 단어는 iOS 개발시 매우 흔하게 마주치게 되는 디자인패턴 용어이다.  
생각해보니 아주 먼 그 옜날 자바 Swing 컴포넌트를 사용했던 학생시절 처음 용어를 접한 기억이 있다.  
Delegation 패턴에 대한 애플의 [Swift][1]언어 가이드를 확인해보자. (protocols 에 대한 내용중)

<br>
> 'Delegation is a design pattern that enables a class or structure to hand off (or delegate) some of its responsibilities to an instance of another type. This design pattern is implemented by defining a protocol that encapsulates the delegated responsibilities, such that a conforming type (known as a delegate) is guaranteed to provide the functionality that has been delegated. Delegation can be used to respond to a particular action, or to retrieve data from an external source without needing to know the underlying type of that source.'
<br>

한 클래스가 어떤 인스탄스에 대한 책임에서 손을 떼는 (다른 객체에게 위임하는) 디자인 패턴인데  
Swift에서는 protocol을 사용해 구현하며 특정 액션 (흔히 버튼클릭) 에 반응하거나  
외부소스에서 데이터를 편하게 가져오기 위해 사용된다고 한다.  
protocol은 자바의 interface와 같다고 보면 되는데 자바 Swing 컴포넌트 사용시에도 버튼 이벤트 리스너 관련하여 위임 패턴을 사용했었다.

<br>

[Swift][1] 가이드문서의 예시를 살펴보자.

```swift
protocol DiceGame {
    var dice: Dice { get }
    func play()
}
protocol DiceGameDelegate: AnyObject {
    func gameDidStart(_ game: DiceGame)
    func game(_ game: DiceGame, didStartNewTurnWithDiceRoll diceRoll: Int)
    func gameDidEnd(_ game: DiceGame)
}
```

두 개의 프로토콜을 정의했고 밑에 있는 DiceGameDelegate 프로토콜은 AnyObject를 상속함으로 [Class-Only 프로토콜][2]로 정의 되었다.
* [Class-Only 프로토콜][2]은 class 타입에서만 구현 가능하도록 제약하는것으로 protocol은 원래 class 이외에도 structure, emumeration 에서도 사용될 수 있는 것으로 보인다.

<br>
*SnakesAndLadders* 클래스에서 위의 두 프로토콜을 구현함.

```swift
class SnakesAndLadders: DiceGame {
    let finalSquare = 25
    let dice = Dice(sides: 6, generator: LinearCongruentialGenerator())
    var square = 0
    var board: [Int]
    init() {
        board = Array(repeating: 0, count: finalSquare + 1)
        board[03] = +08; board[06] = +11; board[09] = +09; board[10] = +02
        board[14] = -10; board[19] = -11; board[22] = -02; board[24] = -08
    }
    weak var delegate: DiceGameDelegate?
    func play() {
        square = 0
        delegate?.gameDidStart(self)
        gameLoop: while square != finalSquare {
            let diceRoll = dice.roll()
            delegate?.game(self, didStartNewTurnWithDiceRoll: diceRoll)
            switch square + diceRoll {
            case finalSquare:
                break gameLoop
            case let newSquare where newSquare > finalSquare:
                continue gameLoop
            default:
                square += diceRoll
                square += board[square]
            }
        }
        delegate?.gameDidEnd(self)
    }
}
```

그런데 두가지 프로토콜을 구현한 방법이 각각 조금 다르다.  
주사위게임의 구현내용은 중요하지 않고 위임에 관련된 부분만 보자.  
* *DiceGame* -  `play()` 안에서 모두 구현됨  
* *DiceGameDelegate* - 구현해야할 3가지 기능을 다른객체에게 위임함.  

정리하자면 *SnakesAndLadders* 클래스는 자기일을 떠넘기는 방법으로  
1.&nbsp;*DiceGame* 프로토콜을 **상속하여 구현**했고,

```swift
class SnakesAndLadders: DiceGame {
```

2.&nbsp;*DiceGameDelegate* 프로토콜의 **변수를 선언**했고,  

```swift
weak var delegate: DiceGameDelegate?
```
3.&nbsp;2에서 선언된 *delegate* 프로토콜 기능들을 직접구현하지 않고 파라메터 인자로 **자기자신(SnakesAndLadders)** 을 넘겨주었다. 이 부분이 포인트.

```swift
// 1번의 이유로 본인을 넘겨줄수 있음.
delegate?.gameDidStart(self)    // func gameDidStart(_ game: DiceGame)
delegate?.game(self, didStartNewTurnWithDiceRoll: diceRoll)
delegate?.gameDidEnd(self)
```

<br>

그럼 이제 Delegate 의 일을 대신해줄 Delegator 를 살펴보자.

```swift
class DiceGameTracker: DiceGameDelegate {
    var numberOfTurns = 0
    func gameDidStart(_ game: DiceGame) {
        numberOfTurns = 0
        if game is SnakesAndLadders {
            print("Started a new game of Snakes and Ladders")
        }
        print("The game is using a \(game.dice.sides)-sided dice")
    }
    func game(_ game: DiceGame, didStartNewTurnWithDiceRoll diceRoll: Int) {
        numberOfTurns += 1
        print("Rolled a \(diceRoll)")
    }
    func gameDidEnd(_ game: DiceGame) {
        print("The game lasted for \(numberOfTurns) turns")
    }
}
```
<br>
특별한 내용은 없다.  
*SnakesAndLadders* 클래스는 일을 하는척(프로토콜을 실행)하면서 self객체인 자기자신을 인자로 넘겨주고   
일을 받은 *DiceGameTracker* 클래스는 *SnakeAndLadders* 객체를 인자로 받는 *DiceGameDelegate* 프로토콜의 기능을 대신 구현해주는 것이다.  
정말 갑과을의 전형을 보여주는 눈물의 K 디자인패턴이라 볼 수 있다.  

<br>
두 클래스의 실행부분을 보면
```swift
let tracker = DiceGameTracker()
let game = SnakesAndLadders()
game.delegate = tracker
game.play()
```
<br>

위 로직은 일을 시킨 *SnakesAndLadders* 클래스에 정의된 주사위를 *DiceGameTracker* 클래스가 가져와서 사용하는 형태로 표현되어 있지만 우리에게 조금 더 흥미있는 방식은 일을 받은 *DiceGameTracker* 가 일을 시킨 *SnakeAndLadders* 의 속성값을 직접 바꿔주는 방식이다. (갑과을의 하극상이랄까...)  
이것을 그림으로 표현하자면

<img src="/img/delegationPattern1.png" >

Delegator가 Delegate 안에 있는 ID 값을 변경했다.  

가이드를 이해했다면 실제 iOS개발시의 예시를 살펴보자.  
다음 포스트에서.
<br>

[1]: https://docs.swift.org/swift-book/LanguageGuide/Protocols.html#ID276
[2]: https://docs.swift.org/swift-book/LanguageGuide/Protocols.html#ID281
