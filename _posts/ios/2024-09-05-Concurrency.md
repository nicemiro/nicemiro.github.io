---
title: Concurrency
titleEn: Using thread in swift
author: BabyK
date: 2023-03-25
category: iOS
layout: post
tags: [iOS, OS, Swift]
published: false
---
<br>

### 멀티코어 CPU의 사용
동시성 작업을 말할 때 사용되는 'Process' 는 사전적 의미가 아니라 컴푸터의 메모리상에 올려져 실행되는 프로그램의 단위를 뜻한다.  
CPU의 코어와 프로세스는 1대1이다. 2개의 프로세스(프로그램)를 1개의 코어를 가진 CPU에서 실행시키고 있다면 CPU는 프로세스들을 빠르게 스위칭하며 두 프로그램이 멈추지 않고 동시에 실행되고 있는 것처럼 보이게 해준다.  
워드프로그램을 사용해 문서를 작성하고 있는 동안에도 문서저장은 자동으로 이루어지며 과거 싱글코어 CPU를 사용하던 시절에는 유저가 수시로 저장버튼을 눌러 줘야 했고 (Ctrl + s의 기억...) 문서의 크기가 커질수록 저장되는 동안 유저의 입력은 먹통이 되기 일쑤였다.  

일반적으로 한 프로그램은 헌 프로세스로 이루어진다. (물론 프로세스를 나누어 멀티 프로세스화 할 수도 있다). 개발 IDE상에서 하나의 프로젝트 (혹은 패키지)를 프로세스로 이해하면 된다. 이 프로세스 안에서 생성되는 스레드들이 있고 이것을 또다시 코루틴으로 나눌 수도 있지만 동시성 프로그램을 작성할 때 사용하는 프로그램 단위는 기본적으로 스레드다.

메모리 > 프로세스 > 스레드 > 코루틴

멀티 스레드를 사용하는 이유는 하나의 프로세스 내에서 이루어지는 스레드의 생성과 스레드들 간의 커뮤니케이션이 멀티 프로세스에 비해 매우 빠르고 리소스 소모도 적기 때문이다.

스위프트의 비동기 코드는 현재의 스레드가 아닌 새로운 스레드에서 실행되며 작성하기 편리하지만 스레드와 직접적으로 커뮤니케이션 하는 방식은 아니기 때문에 어떤 스레드에서 코드가 작동하게 될지는 알수 없다. C 와 같은 언매니지드 언어를 사용한다면 스레드를 보다 직관적이고 명시적으로 다룰 수 있지만 유저가 스레드의 생성과 분배, 소멸과 같은 사이클을 모두 관리해야 한다는 불편함이 있다. ( 불편함 or 복잡함 == 잘 사용한다면 성능적으로 더 좋다는 뜻 )

Parellel 과 Asynchronous 는 다른의미로 구분지어 사용해야 한다. 패러럴 코드는 여러개의 코드가 동시에 시작될 수 있음을 뜻하지만 (시점), 비동기 코드는 동시에 시작되지 않고 다만 여러개의 코드가 멈추지 않고 실행되고 있음을 뜻한다 (상태). 패러럴은 비동기가 될 수 있지만 비동기가 패러럴을 의미 할 수는 없다. 비동기 코드는 1코어에서도 작동할 수 있지만 패러럴 코드는 2개 이상의 코어가 필요함.