---
title: Observer Pattern + notification Center
titleEn:
author: BabyK
date: 2024-08-16
category: Design_Pattern
tags: [Pattern, ios]
layout: post
published: true
---

### 구독과 알람 기능을 생각해보자
유튜브의 Subscribe 기능을 생각해보면 옵저버패턴의 정체를 알 수 있다.  
구독자들은 Observer 가 되어 유튜버를 구독하고  
유튜버는 이벤트가 생길때마다 구독자들에게 알람으로 Notification을 날린다.  

업적당성 메달을 획득하는 게임 이벤트 또한 예시로 들 수 있다.  
유저가 특정한 업적에 도달하면 업적객체는 유저의 업정달성을 확인하고  
유저에게 메달획득 확인알람을 보내준다.  
이 경우 업적객체가 Observer, 유저객체가 Subject가 되고  
업적객체는 유저가 업적달성 상태가 되었는지 끊임없이 관찰하게 된다.  
* 실제로는 유저객체가 아닌 게임환경객체 혹은 물리엔진객체를 Subject 로   
  사용하고 이 객체들을 통해 유저정보를 받아 알람을 브로드캐스팅   
  하는 것이 더 자연스럽다.    

방송국의 전파 브로드캐스트는 옵저버 패턴이라고 보기 애메하다.  
방송국은 해당 컨텐츠를 무작위로 브로드캐스트하고 시청자들은 주파수를  
맞춰 컨텐츠를 시청하는데 방송국은 시청자가 누구인지 모르기 때문이다.  
옵저버패턴의 Subject는 자신을 관찰하고 있는 Observer 가 누구인지  
모두 알고 있다는게 중요한 차이점임.  

즉 방송국을 Subject라고 본다면 Observer들은 Subject 가 가진  
목록에 등록되어 있어야 한다. 마치 유튜버가 구독자목록을 가지고 있듯이.  
구독자들은 서로를 알지 못하고 알 필요도 없다.   

요약하자면 Subject 의 주요 기능 두 가지는 
* 자신을 관찰하는 Observer 들의 목록을 들고 있을 것
* Notification - 알람기능

### 유튜브 구독,알람 코드의 예시
뉴진스 채널과 아이브 채널이 존재하고 팬들은 선호하는 채널을 찾아가  
구독버튼을 누른다. 그럼 각 채널은 자신의 구독자목록에 이들을 추가하고 새로운  
영상을 올릴때마다 가지고 있는 목록을 순회하며 알람을 발송한다.  

```swift
import Foundation

protocol Subject {
    var id : String {get set}
    var observers : [Observer] {get set}
    func notify(_ message: String)
    func subscribe(_ observer: Observer)
    func unsubscribe(_ observer:Observer)
}

protocol Observer {
    var id: String { get set }
    func onNotify(_ message: Message)
}

class User: Observer {
    var id: String
    init(_ id:String) {
        self.id = id
    }
    func onNotify(_ message: Message) {
        print("(User \(id)) Message from \(message.subject) : \"\(message.message)\"")
    }
}

class Youtuber: Subject {
    var id: String
    var observers: [Observer] = []
    
    init(_ id:String) {
        self.id = id
    }
    
    func notify(_ message: String) {
        let msg = Message(self.id, message)
        for i in observers {
            i.onNotify(msg)
        }
    }
    
    func subscribe(_ observer: Observer) {
        if observers.contains(where: {$0.id == observer.id} ) == true {
            let msg = Message(self.id, "You are already subscribed")
            observer.onNotify(msg)
        }
        else
        {
            self.observers.append(observer)
            print("User '\(observer.id)' has subscribed to '\(self.id)' channel.")
        }
    }
    
    func unsubscribe(_ observer: Observer) {
        if let index = observers.firstIndex(where: {$0.id == observer.id}) {
            observers.remove(at: index)
            let msg = Message(self.id, "Your subscription has been canceled")
            observer.onNotify(msg)
        }
    }
}   // END class Youtuber

// notification info
struct Message {
    let subject : String
    let message : String
    init (_ subject: String, _ message: String) {
        self.subject = subject
        self.message = message
    }
}

// init Youtuber
let youtuber_NewJeans = Youtuber("NewJeans")
let youtuber_Ive = Youtuber("Ive")

// init subscriber
let user1 = User("Kim")
let user2 = User("Lee")
let user3 = User("Choi")
let user4 = User("Park")

// user clicks subscribe button (구독)
youtuber_NewJeans.subscribe(user1)
youtuber_NewJeans.subscribe(user3)
youtuber_NewJeans.subscribe(user4)
print("\n")
youtuber_Ive.subscribe(user2)
youtuber_Ive.subscribe(user4)
print("\n")

// you tubers send notifications of their new content to their subscribers (알람)
youtuber_NewJeans.notify("Our New content just has been uploaded. Don't miss it!")
youtuber_Ive.notify("Our New content just has been uploaded. Thank you!")
print("\n")

// unsubscribe (구독취소)
youtuber_NewJeans.unsubscribe(user1)
youtuber_Ive.unsubscribe(user2)
```

<img src="/img/designPattern/2024-08-16-ObserverPattern_0.png" style="width:100%">

위의 코드에서 구독자는 유튜버에 대한 정보를 갖고 있지 않기 때문에 둘 사이에 강한 커플링은 발생하지 않는다.  
강한 참조로 인한 메모리 Leak 에 대한 걱정보다는 모든 옵저버들을 순환하며 Noti 를 날릴때    
동기화(Sync) 방식으로 인해 발생할 수 있는 딜레이 문제를 더 신경 써야한다.  
Subject의 알람 기능에는 과도한 오버로드가 쏠리지 않도록 주의하자.    

멀티스레드를 사용한 비동기 방식을 생각할 수도 있지만  
Subject가 하나의 Observer 에 정의된 onNotifiaction 을 실행하는 상황에서 Lock 이 해제되지 않는다면  
다른 Observer 들이 실행되지 못해 전체 시스템이 꼬여버릴수 있고   
뒤이어 설명할 iOS의 Notification Center의 코드를 보면 알 수 있겠지만  
옵저버 패턴자체가 둘 사이에 발생하는 Notification의 코드 흐름이 런타임 환경에서 확인하는 것이  
더 용이하기 때문에 디버깅시에 코드 추적이 불편할 수도 있다.   
비동기 방식을 사용하고 싶다면 이벤트 큐 패턴을 사용하는 것을 고려해보자.  
 

### NotificationCenter in iOS

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