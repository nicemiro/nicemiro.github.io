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
미리 약속된 행위를 트리거 하게 한다.  

<br>
>Although the notification center delivers a notification to its observers synchronously, you can post notifications asynchronously using a notification queue (NSNotificationQueue). 

- Posting Object to NotificationCenter (Async방식 가능)
- Broadcasting NotificationCenter to observer  (**Sync**방식)


<br>
>Before the posting object can resume its thread of execution, it must wait until the notification center dispatches the notification to all observers and return

세로운 Post를 수행하기 전에 NotificationCenter에서 브로트캐스트를 보낸 옵저버들의 행위가 끝날 때 까지 기다려야 한다.  
즉 NotificationCenter는 사용하기 편리하지만 **옵저버들이 늘어나면 속도를 저하 시킬 수 있다**는 것.  
대부분의 경우 default notification center를 사용하지만 사용하는 옵저버 수가 많아진다면 default가 아닌 새로운 notification center를 생성하여 응답속도를 빠르게 할 수 있다.  

Notification Center의 사용예시를 보자.

[1]: https://developer.apple.com/documentation/foundation/notificationcenter