---
title: Observer Pattern (옵저버패턴)
titleEn: Observer Pattern
author: BabyK
date: 2024-08-16
category: Design_Pattern
layout: post
---
한 객체가 자기자신을 다른 객체에게 넘겨준 후 자신을 위해 일하게 만드는 패턴.  
현재 화면에서 이전화면의 상태를 바꾸는 가장 손쉬운 방법으로  
뷰를 사용한 화면 전환이라던지 버튼 클릭과 같은 이벤트 처리에서 주로 사용됨.  
<br>

Observer Pattern의 설명 (subscribe + broadcast)


weak 변수를 사용한 delegate 를 서로 만들 경우에 weak 으로 인해
두 객체가 가비지 컬렉터에의해 메모리해제될 위험이 있음.
그래서 iOS 에서는 NotificationCenter를 사용함

아래의 예시를 보여줌
Observer Pattern의 사용예시인 Notification

GameViewController.swift
```swift
 
// GameScene 에서 delegate 으로 호출됨. 성공시 notifyRewarded() 실행
func showRewardedAd() {
    guard let rewardedAdObj = rewardedAd else {
        return print("Ad wasn't ready.")
    }
    // call back after user is rewarded
    rewardedAdObj.present(fromRootViewController: nil) {
        self.notifyRewarded()
    }
}

// subscriber (gameScene) 에게 notify 
@objc func notifyRewarded() {
    NotificationCenter.default.post(name: Notification.Name("CustomNotification"), object: nil, userInfo: ["message": "Hello from MainViewController!"])
}
```
GameScene.swift
```swift
init {
    NotificationCenter.default.addObserver(self, selector:  #selector(handleNotification(_:)), name: Notification.Name
    ("CustomNotification"), object: nil)
}

deinit {
    NotificationCenter.default.removeObserver(self)
}

@objc func handleNotification(_ notification: Notification) {
    if let userInfo = notification.userInfo, let message = userInfo["message"] as? String {
        // 알림 수신 시 처리할 로직
        print("Received notification with message: \(message)")
    }
}



```





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
첫 화면인 'GameBoard' 가 넘겨준 self 객체를 가지고  
'GmaeBoardDetail' 화면에서 주사위 로직을 실행함.  
'GameBoard' 화면에서는 본인을 넘겨주며 실행요청만 하고 정작 주사위 Roll 로직은  
'GameBoardDetail' 에서 구현됨.  

두 객체의 강한연결로 인한 메모리 누수를 막기위해 `weak` 사용,  
'GameBoard' 메모리 해제되었을 때 'GameBoardDetail' 객체가 남아있는 상황을 방지함.  

현재 화면 (지금 사용중인 객체)이 아닌 다른 객체에 있는 기능을 사용하려 하거나 (주로 버튼입력)  
이전 혹은 다음 화면의 상태를 현재 화면에서 조작 하려 할 때 주로 사용된다.  
self를 넘겨주어서 자기자신의 객체와 함께 추가 로직을 실행할 수 있다는 점이 포인트.  

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