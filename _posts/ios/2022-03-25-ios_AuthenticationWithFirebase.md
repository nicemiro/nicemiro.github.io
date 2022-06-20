---
title: Firebase를 이용한 카카오톡 만들기
titleEn: Chat App using Firebase
author: BabyK
date: 2022-05-20
category: iOS
layout: post
published: false
---

### 채팅앱 만들기

카카오톡과 같은 채팅앱을 만들 때는 사용자 인증 부터 데이터베이스구축, 테스트 및 리모트 설정까지 다양한 서비스를 포함하고 있는
서버가 필요하지만 [Firebase][1]를 사용하면 백엔드 서버의 구축없이 간단하게 위의 기능들을 구현하여 다양한 앱을 만들어 볼 수 있다.
카카오톡을 카피한 채팅앱을 만들어 보자.
<br>
#### 화면별 사용할 수 있는 Firebase의 기능들.
- 로그인 화면의 사용자인증  
    * Firebase Authentication
- 사용자 목록, 채팅룸  
    * Cloud Firestore
    * Realtime Database
- 세팅창 및 공지사항  
    * RemoteConfig


로그인 중일 때 메세지가 발생하면 push message 생성

* programatic하게 코드로 뷰를 구성할 수도 있지만 UI Kit을 사용하면서 굳이 그럴 필요가 있나 싶다.
조만간 SwiftUI로 모두 전화해볼 예정.

[1]: https://firebase.google.com/