---
title: NotificationCenter 사용하기
titleEn: NotificationCenter in Swift
author: BabyK
date: 2022-03-26
category: iOS
layout: post
---

### NotificationCenter 사용하기.

[NotificationCenter][1] API는 등록된 옵저버에게 데이터를 브로드 캐스트 해주는 클래스이다.  
Delegation과 비교하면 이벤트등록의 간편성 및 특정 이벤트를 옵저빙하고 있는 모든 객체에게  
이벤트의 발생을 **브로드 캐스팅** 한다는 특성상 보다 편리하게 사용할 수 있고 객체들간의 의존성을 줄여줄 수 있다.


<img src="/img/2022-03-25-iosNotificationCenter1.png" >

어떤 객체에 의해 특정 이벤트의 Post가 들어오면 [NotificationCenter][1]는 해당 이벤트와 함께 등록된 옵저버들을 스캔하고 검색된 옵저버들에게
미리 약속된 행위를 실행하게 한다.  

<br>
>Although the notification center delivers a notification to its observers synchronously, you can post notifications asynchronously using a notification queue (NSNotificationQueue). 

- Posting Object -> NotificationCenter (Async방식 가능)
- Broadcasting NotificationCenter -> observer  (**Sync**방식)

<br>
>Before the posting object can resume its thread of execution, it must wait until the notification center dispatches the notification to all observers and return

세로운 Post를 수행하기 전에 NotificationCenter에서 브로트캐스트를 보낸 옵저버들의 행위가  
끝날 때 까지 기다려야 한다.  
  
즉 NotificationCenter는 사용하기 편리하지만 **옵저버들이 늘어나면 속도를 저하 시킬 수 있다**는 것.  
대부분의 경우 디폴트 Notification center를 사용하지만  
사용하는 옵저버 수가 많아진다면 디폴트가 아닌 신규 Notification center를 생성하여  
응답속도를 빠르게 할 수 있다.  

옵저버 등록에는 아래의 2가지 방법이 있고 옵저버 등록시 수신할 Notification을 명시해준다.  
- addObserver(_:selector:name:object:)
Adds an entry to the notification center to call the provided selector with the notification.
수신할 노티피케이션과 호출할 셀렉터를 등록하는 방법.  

- addObserver(forName:object:queue:using:)
Adds an entry to the notification center to receive notifications that passed to the provided block.
노티피케이션이 수신되는 블록을 노티피케이션과 함께 등록하는 방법.  

옵저버는 하나의 프로그램 내에서만 Notification을 수신할 수 있으며 프로그램간 Notification을 수신하려면  
[DistributedNotificationCenter][2] 를 사용해야 한다.  



[1]: https://developer.apple.com/documentation/foundation/notificationcenter
[2]: https://developer.apple.com/documentation/foundation/distributednotificationcenter