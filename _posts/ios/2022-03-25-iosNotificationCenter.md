---
title: NotificationCenter 사용하기
titleEn: NotificationCenter in Swift
author: BabyK
date: 2022-03-26
category: iOS
layout: post
---

### NotificationCenter

[NotificationCenter][1] API는 등록된 옵저버에게 데이터를 브로드 캐스트 해주는 클래스이다.  
Delegation과 비교하면 이벤트등록의 간편성 및 특정 이벤트를 옵저빙하고 있는 모든 객체에게  
이벤트의 발생을 **브로드 캐스팅** 한다는 특성상 보다 편리하게 사용할 수 있고 객체들간의 의존성을 줄여줄 수 있다.


<img src="/img/2022-03-25-iosNotificationCenter1.png" >

어떤 객체에 의해 특정 이벤트의 Post가 들어오면 [NotificationCenter][1]는 해당 이벤트와 함께 등록된 옵저버들을 스캔하고 검색된 옵저버들에게
미리 약속된 행위를 실행하게 한다.  
  
<!-- >Although the notification center delivers a notification to its observers synchronously, you can post notifications asynchronously using a notification queue (NSNotificationQueue). 
   
>Before the posting object can resume its thread of execution, it must wait until the notification center dispatches the notification to all observers and return -->

- Posting Object -> NotificationCenter (Post : Sync/Async방식 둘다 가능)  
  NotificationCenter -> observer  (Notification : **Sync**방식)  

- 세로운 Post를 수행하기 전에 NotificationCenter에서 브로트캐스트를 보낸 옵저버들의 행위가  
끝날 때 까지 기다려야 한다.  
  
- 즉 NotificationCenter는 사용하기 편리하지만 **옵저버들이 늘어나면 속도를 저하 시킬 수 있다**는 것.  
대부분의 경우 디폴트 Notification center를 사용하지만 사용하는 옵저버 수가 많아진다면 디폴트가 아닌 신규 Notification center를 생성하여  
응답속도를 빠르게 할 수 있다.  

- Notification은 싱글 프로그램 상에서만 사용가능하다. 프로세스간에 Notification이 필요하다면
[DistributedNotificationCenter][2] 를 사용해야 한다.

<br>
#### 옵저버 등록
`addObserver(_:selector:name:object:)`  
수신할 노티피케이션과 호출할 셀렉터를 등록하는 방법.    
- &nbsp; *_ observer* : 노티피케이션 발생을 관찰할 옵저버 객체  
- &nbsp; *selector* : 노티피케이션 수신시 실행할 펑션의 셀렉터   
- &nbsp; *name* : 수신할 노티피케이션명. nil 일경우 sender는 노티피케이션 이름을 기준으로 post하지 않음.  
- &nbsp; *object* : 노티피케이션을 send하는 객체. nil 설정가능 (*name* 파라메터가 nil일때 *object* 파라메터가 sender의 기준이 될수도 있지 않을까)

`addObserver(forName:object:queue:using:)`  
노티피케이션 수신시 실행할 블록을 명시하는 방법.  
오퍼레이션 큐 ( 순차적 펑션실행 == 쓰레드관련 == 왠만하면 사용하지 말자 )를 사용함.

<br>
#### 노티피케이션 포스트  
`post(name:object:userInfo:)`  
Notification name, sender, userinfo를 파라메터로 사용함.
혹은 userinfo 를 생략할 수 있음.


<br>
#### 사용예시
<strong>addObserver

```swift
class ViewController: UIViewController {
    ...
    override func viewDidLoad() {
        super.viewDidLoad()
        // 옵저버생성
        let notiTempt = NSNotification.Name("sendNotification")
        NotificationCenter.default.addObserver(self, selector: #selector(self.tappedButton), name: notiTempt, object: nil) 
    }
    // selector 생성
    @objc func tappedButton() {
        debugPrint("tapped button")
    }
    ...
}
```

<strong>post
```swift
class ViewController2: UIViewController {
    ...
    @IBAction func sendNotification(_ sender: UIButton) {
        let notiTemp = NSNotification.Name("sendNotification")
        NotificationCenter.default.post(name: notiTemp, object: nil)
    }
    ...
}
```


[1]: https://developer.apple.com/documentation/foundation/notificationcenter
[2]: https://developer.apple.com/documentation/foundation/distributednotificationcenter


<br>
<strong>Reference
- [https://developer.apple.com/documentation/foundation/notificationcenter] [1]{:target="_blank"}  
- [https://developer.apple.com/documentation/foundation/distributednotificationcenter] [2]{:target="_blank"}  
