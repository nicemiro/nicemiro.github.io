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
유튜브 유저들은 Observer 가 되어 유튜버를 구독하고 유튜버는 Subject 가 되어 이벤트가 발생할 때마다 구독자들에게 알람 (Notification) 을 발송한다.  

게임시스템의 업적달성 알람 또한 옵저버 패턴으로 볼 수 있는데, 유저가 특정한 업적에 도달하면 게임환경 객체 혹은 물리엔진 객체가 유저의 업적 달성을 확인하고 업적 객체에게 알람을 보내 메달획득 이벤트가 발생했음을 알려준다. 이 경우, 업적객체가 Observer, 유저객체가 Subject 가 되고 업적객체는 유저가 업적달성 상태가 되었는지 끊임없이 관찰하게 된다. (사실, 달성유무를 체크하는건 관찰자인 업적객체가 아니라 대상자인 유저와 소통하고 있는 게임환경, 물리엔진 객체다. 발생된 알람을 수신하는 객체를 관찰자로 생각하는 문학적인 표현으로 받아들이자)

방송국의 전파 브로드캐스트는 얼핏 옵저버 패턴으로 보일 수 있지만 방송국은 수신자에 대해 알지 못한다는 차이점이 있다. 옵저버 패턴의 Subject 는 자신을 관찰하고 있는 Observer 가 누구인지 모두 알고 있다는게 중요한 차이점.  

유튜버가 구독자 목록을 가지고 있듯이 Subject 는 Observer 를 목록에 등록해 두어야 한다. 구독자들은 서로를 알지 못하고 알 필요도 없다.   

#### Subject 의 주요 기능 두 가지
* 자신을 관찰하는 Observer 들의 목록을 들고 있을 것
* 이벤트 발생을 알리는 Notification 기능
<br>
<br>

### 구독과 알람 코드의 예시
뉴진스와 아이브의 유튜브 채널을 생각해보자. 유저는 선호하는 채널을 찾아가 구독버튼을 누른다. 그럼 각 채널은 자신의 구독자 목록에 이들을 추가하고 새로운 컨텐츠가 올라올 때마다 갖고 있는 목록을 순회하며 알람을 발송한다.  

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
<img src="/img/designPattern/2024-08-16-observerPattern_0.png" style="width:100%">
<br>
<br>

위의 예시에서는 구독자들이 스트링 타입의 구독채널 ID 만 갖고 있지만 이를 사용해 DB 조회를 하는 상황이 아니라면 각 유저들이 구독한 유튜버의 객체(포인터) 를 갖고 있는 것이 더 현실적일 것이다. 두 객체 사이에 강한 참조가 발생하지 않도록 Wrapper Class 를 이용해 둘 사이에 레이어를 하나 만들어주자.  

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

### 주의할 점

* 강한 참조로 인한 메모리 Leak 에 대한 걱정보다는 오히려 목록상의 관찰자들을 순환하며 알람을 날리는 시점의 동기화(Sync) 로 인해 발생할 수 있는 딜레이를 더 신경 써야한다. Subject 의 이벤트 알람 기능에는 과도한 오버로드가 발생하지 않도록 주의하자. 또한 이미 구독취소한 Observer 가 아직 목록에 남아 있다면 필요없는 알람으로 리소스가 낭비되거나 비어있는 옵저버의 포인터 공간을 건드리게 되는 치명적인 오류가 발생할 수 있다. (deinit 의 사용을 고려하자)  

* 멀티스레드를 사용한 비동기 처리와 같은 복잡한 시스템에 사용하기 어렵다. 하나의 스레드가 Notification을 처리하는 과정에서 락이 헤제되지 않는 상황, 대상자와 수많은 관찰자들 간에 발생할 수 있는 문제 해결을 위해서는 결국 옵저버 패턴을 확장한 이벤트 큐 패턴을 사용하게 된다.  

* 대상자가 관찰자와 정적으로 묶이지 않고 낮은 커플링으로 동작하며 상호작용 할 수 있다는 옵저버 패턴의 장점은, 관찰자가 많아질 경우 데이터의 흐름을 확인하기 불편할 수 있다는 단점이 된다.

* 요청과 처리 방식이 제한되어 있고 대상자를 구독한 모든 옵저버에 브로드캐스트 되기 때문에 UI 변경 이벤트와 같은 복잡도가 낮은 작업에 사용하기 좋다.  

* 같은 기능을 구현하는 코드들은 결합도가 낮다고 하더라도 결국 하나의 모듈로 응집시켜 사용하는 것이 편리하다. 옵저버 패턴은 그보다 서로 연관없는 코드들 사이의 상호작용이 필요할 때 더 쓰임세가 많다. (물리엔진 모듈에 업적시스템의 코드가 지저분하게 섞이지 않을 수 있음)  


<br>
