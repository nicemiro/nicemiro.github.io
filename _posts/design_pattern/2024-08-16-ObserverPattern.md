---
title: Observer Pattern (옵저버 패턴)
titleEn:
author: BabyK
date: 2024-08-16
category: Design_Pattern
tags: [DesignPattern, iOS]
layout: post
published: true
---
<br>
### 구독과 알람 기능을 생각해보자
유튜브의 구독 기능이 바로 옵저버 패턴 이다. 구독자들은 Observer 가 되어 유튜버를 구독하고 유튜버는 Subject 가 되어 이벤트가 생길때마다 구독자들에게 알람 (Notification) 을 발송한다.  

업적당성 메달을 획득하는 게임 이벤트 또한 예시로 들 수 있는데 유저가 특정한 업적에 도달하면 업적객체는 유저의 업정달성을 확인하고 유저에게 메달획득 알람을 보내준다. 이 경우, 업적객체가 Observer, 유저객체가 Subject 가 되고 업적객체는 유저가 업적달성 상태가 되었는지 끊임없이 관찰하게 된다.  
* 실제로는 유저객체가 아닌 게임환경객체 혹은 물리엔진객체를 Subject 로 사용하고 이 객체들을 통해 유저정보를 받아 알람을 발송하는 것이 더 자연스럽다.    

방송국의 전파 브로드캐스트는 얼핏 보기에는 옵저버 패턴으로볼 수 있지만 방송국은 수신자에 대해 알지 못한다는 차이점이 있다 옵저버패턴의 Subject 는 자신을 관찰하고 있는 Observer 가 누구인지 모두 알고 있다는게 중요한 차이점.  

유튜버가 구독자목록을 가지고 있듯이 Subject 는 Observer 를 목록에 등록해 두어야 한다. 구독자들은 서로를 알지 못하고 알 필요도 없다.   

#### Subject 의 주요 기능 두 가지
* 자신을 관찰하는 Observer 들의 목록을 들고 있을 것
* 이벤트 발생을 알리는 Notification 기능
<br>
<br>

### 구독과 알람 코드의 예시
뉴진스와 아이브의 유튜브 채널을 생각해보자. 팬들은 선호하는 채널을 찾아가 구독버튼을 누른다. 그럼 각 채널은 자신의 구독자목록에 이들을 추가하고 새로운 영상을 올릴 때마다 가지고 있는 목록을 순회하며 알람을 발송한다.  

```swift
import Foundation

protocol Subject {
    var id : String {get set}
    var observers : [Observer] {get set}
    func notify(_ message: String)
    func setSubscriber(_ observer: Observer)
    func removeSubscriber(_ observer:Observer)
    func printObservers()
}

protocol Observer {
    var id: String { get set }
    var subjectIds: [String] {get set}
    func onNotify(_ message: Message)
    func subscribe(_ subject: Subject)
    func unsubscribe(_ subject: Subject)
    func printSubscribes()
}

class User: Observer {
    var id: String
    var subjectIds: [String] = []
    
    init(_ id:String) {
        self.id = id
    }
    
    func subscribe(_ subject: Subject) {
        if subjectIds.contains(where: { $0 == subject.id} )
        {
            print("You are already subscribing to \(subject.id)")
        }
        else
        {
            subject.setSubscriber(self)
            subjectIds.append(subject.id)
        }
    }
    
    func unsubscribe(_ subject: Subject) {
        if let index = subjectIds.firstIndex(where: {$0 == subject.id})
        {
            subject.removeSubscriber(self)
            subjectIds.remove(at: index)
        }
        else
        {
            print("You are not subscribing to \(subject.id)")
        }
    }
    
    func onNotify(_ message: Message) {
        print("(User \(id)) Message from \(message.subject) : \"\(message.message)\"")
    }
    
    func printSubscribes() {
        print("\(self.id) is subscring to \(subjectIds.count) subjects")
    }
}   // END User

class Youtuber: Subject {
    var id: String
    var observers: [Observer] = []
    
    init(_ id:String) {
        self.id = id
    }

    // 알람기능. 관찰자에 정의된 onNotify() 를 실행
    func notify(_ message: String) {
        let msg = Message(self.id, message)
        for i in observers {
            i.onNotify(msg)
        }
    }
    
    func setSubscriber(_ observer: Observer) {
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
    
    func removeSubscriber(_ observer: Observer) {
        if let index = observers.firstIndex(where: {$0.id == observer.id}) {
            observers.remove(at: index)
            let msg = Message(self.id, "Your subscription has been canceled")
            observer.onNotify(msg)
        }
    }
    
    func printObservers() {
        print("\(self.id) has \(observers.count) subscribers")
        for i in observers {
            print(i.id)
        }
    }
}   // END Youtuber

struct Message {
    let subject : String
    let message : String
    init (_ subject: String, _ message: String) {
        self.subject = subject
        self.message = message
    }
}
```

##### 실행코드
```swift
// init Youtuber
let youtuber_NewJeans = Youtuber("NewJeans")
let youtuber_Ive = Youtuber("Ive")

// init subscriber
let user1 = User("Kim")
let user2 = User("Lee")
let user3 = User("Choi")
let user4 = User("Park")

// user click subscribe button
print("구독")
user1.subscribe(youtuber_NewJeans)
user3.subscribe(youtuber_NewJeans)
user4.subscribe(youtuber_NewJeans)
user2.subscribe(youtuber_Ive)
user4.subscribe(youtuber_Ive)
print("\n")

// you tubers send notifications of their new content to their subscribers
print("알람")
youtuber_NewJeans.notify("Our New content just has been uploaded. Don't miss it!")
youtuber_Ive.notify("Our New content just has been uploaded. Thank you!")
print("\n")

print("구독자 목록")
youtuber_NewJeans.printObservers()
youtuber_Ive.printObservers()
print("\n")

print("구독중")
user1.printSubscribes()
user2.printSubscribes()
user3.printSubscribes()
user4.printSubscribes()
print("\n")

// unsubscribe
print("구독취소")
user1.unsubscribe(youtuber_NewJeans)
user2.unsubscribe(youtuber_Ive)
youtuber_NewJeans.printObservers()
youtuber_Ive.printObservers()
print("\n")

print("구독중")
user1.printSubscribes()
user2.printSubscribes()
user3.printSubscribes()
user4.printSubscribes()
```

##### 결과
<img src="/img/designPattern/2024-08-16-ObserverPattern_0.png" style="width:100%">
<br>
<br>

실제 운영환경에서는 유튜버의 ID 값만 가져가서 DB 처리를 할 것이기 때문에 유저가 구독한 채널들을 String 배열에 저장했지만, Wrapper Class 를 사용해서 유저가 유튜버 객체를 들고 있도록 설계할 수도 있다. 이 경우, 두 객체 사이에 불필요한 강한참조가 발생하지 않도록 'weak' 키워드를 사용한다.  

##### Wapper Class 를 사용한 예시
```swift

// weak은 클래스 타입에만 적용할 수 있기때문에 AnyObject를 상속해서 
// class 타입에만 적용할 수 있도록 제한함
protocol Subject: AnyObject {
    var id : String {get set}
    var observers : [Observer] {get set}
    func notify(_ message: String)
    func setSubscriber(_ observer: Observer)
    func removeSubscriber(_ observer:Observer)
    func printObservers()
}

// Subject 타입의 weak 프로퍼티를 가진 레퍼클래스 선언
class WeakSubjecctWrapper {
    weak var value: Subject?
    init(value: Subject) {
        self.value = value
    }
}

// 유튜버 - 구독자 객체 사이의 강한 참조를 피해  
// 유튜버레퍼클래스 - 구독자 객체 사이의 약한 참조를 만들어 메모리 누수를 방지
class User: Observer {
    var id: String
    // var subscribingIds: [String] = []
    var subscribingSubjects: [WeakSubjecctWrapper] = []
    
    // ...
}

```

강한 참조로 인한 메모리 Leak 에 대한 걱정보다는 오히려 목록상의 관찰자들을 순환하며 알람을 날리는 시점의 동기화(Sync) 로 인해 발생할 수 있는 딜레이를 더 신경 써야한다. Subject 의 이벤트 알람 기능에는 과도한 오버로드가 발생하지 않도록 주의하자. 또한 이미 구독취소한 Observer 가 아직 목록에 남아 있다면 필요없는 알람으로 리소스가 낭비되거나 비어있는 옵저버의 포인터 공간을 건드리게 되는 치명적인 오류가 발생할 수 있다.  

멀티스레드를 사용한 비동기 방식을 생각할 수도 있지만 Subject 가 특정 Observer 에 정의된 onNotify() 를 실행하는 상황에서 쓰레드의 Lock 이 해제되지 않는다면 다른 Observer 가 알람을 받지못해 전체 시스템이 꼬여버릴수 있고, 대상자와 수많은 관찰자 사이에서 발생하는 오류를 파악하기 위해서는 결국 런타임 환경에서 동적인 실행과정을 확인해야만 한다. 비동기 방식을 사용하고 싶다면 이벤트 큐 패턴을 사용하는 것을 고려해보자.  

위의 코드에서는 Message 객체에 Subject.id 만 저장했지만 관찰자들에게 보내는 Notifiaciton 의 파라미터로 이벤트 Sender 인 self 를 추가하는 것이 일반적이다.  

##### 옵저버 패턴의 장점은 Subject 가 Observer 와 정적으로 묶이지 않고 낮은 커플링으로 동작하며 상호작용 할 수 있다는 것, 때문에 옵저버가 많아질 경우  데이터의 흐름을 확인하기 불편할 수 있다는 단점 또한 여기서 기인한다.   

##### 같은 기능을 구현하는 코드들은 결합도가 낮다고 하더라도 결국 하나의 모듈로 응집되어 있을 수밖에 없다. 옵저버 패턴은 서로 연관없는 코드간의 상호작용이 필요할 때 사용하면 좋다.  


<br>

<!---
### NotificationCenter in iOS (on going)
iOS 환경에서 옵저버 패턴으로 사용되는 NotificationCenter를 살펴보자.     

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
-->
<br>