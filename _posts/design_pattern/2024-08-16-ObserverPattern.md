---
title: Observer Pattern + notification Center
titleEn:
author: BabyK
date: 2024-08-16
category: Design_Pattern
tags: [Pattern, ios]
layout: post
published: false
---

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