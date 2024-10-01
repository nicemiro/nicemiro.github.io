---
title: Event Queue Pattern (이벤트 큐 패턴), Semaphore (세마포어)
titleEn:
author: BabyK
date: 2024-08-29
category: Design_Pattern
tags: [DesignPattern, OS, Swift]
layout: post
published: true
---
<br>

### 이벤트 큐 패턴
단순히 입력과 처리를 분리하고 싶다면 옵저버 패턴을 사용해도 되지만 보다 복잡한 **비동기화** 를 구현해야 한다면 이벤트 큐 패턴을 사용할 수 있다. 옵저버 패턴과의 차이점은 이벤트 발송과 처리의 **시점** 을 분리해서 **비동기** 로 작동한다는 점으로 중앙처리 방식을 비롯한 광역 시스템에 유용하게 사용될 수 있다. OS상의 인터페이스 작동 방식으로도 흔히 사용되는데 예를 들면 유저의 키입력과 화면 출력 사이 아주 짧은 시간에도 워드프로그램은 멈춰있지 않고 동시에 키입력을 감지하거나 중간저장을 수행할 수 있다.

이벤트 큐 패턴을 사용하면 옵저버 패턴 보다 더 폭넓고 다양한 이벤트의 수신과 처리 구현이 가능하다. 이는 큐와 멀티프로세스를 활용한 작업의 비동기화가 가능하기 때문인데 발송하는 쪽에서는 큐에 이벤트 요청을 넣는 작업까지만 관여하고 그 이후의 작업은 큐와 수신자가 담당한다. 양쪽은 이 작업을 계속해서 반복하는데 요청자와 실행자 (큐 + 수신자) 는 서로의 상태를 신경쓸 필요가 없다. 단지 요청자와 수신자 양쪽이 동시에 큐에 접근하는 상황 (Data corruption by race condition) 만 신경쓰면 될뿐.  
<br>
<br>

### 세마포어
유저가 실행한 프로그램은 프로세스(Process) 라는 단위로 메모리 공간을 점유하게 되는데 프로세스는 단순히 하나의 프로젝트, 패키지, 혹은 그냥 실행 중인 프로그램이라고 생각하면 된다. 멀티코어를 활용해 다수의 프로세스를 생성 할 수도 있지만 메모리 가용성과 퍼포먼스의 측면에서 페널티가 크기 때문에 하나의 프로세스 공간을 공유하는 다수의 스레드를 활용하여 **멀티 스레드** 환경을 구축하는 것이 일반적이다. (스레드와 프로세스는 운영체제의 영역이며 이들의 CPU 코어 점유는 OS의 스케줄러가 담당함. 엄밀히 말하자면 하나의 스레드가 하나의 코어를 점유하는 상황일 수도, 아닐 수도 있음)
하나의 프로그램이 멀티 스레드를 사용하여 처리한 데이터들은 어느 시점에서는 취합되고 조직되어야 하는데 이때, 다수의 프로세스 또는 스레드가 같은 자원 (큐와 같은) 에 접근해 동시에 데이터를 조작하려 하는 **Race condition** 이 발생할 수 있다. 이를 방지하는 대표적인 툴 중에 하나인 세마포어에 대해 알아보자.  

```c
// C 와 Swift 를 섞은 그냥 psudo 코드라고 생각하자...

// 세마포어 구조체
 typedef struct {
    int value;
    struct process *list;
 } semaphore;


// wait. 세마포어 요청
wait(semaphore *S) {
    S->value--;
    if (S->value < 0 )
    {   // value가 0 이하 일때. 이미 먼저 wait() 을 콜하고 실행 중인 스레드가 존재함.
        // 요청한 스레드 객체를 세마포어 리스트에 추가하고 대기 상태로 바꿔 비활성화.
        S->list.append(this)
        sleep();
    }
    else
    {   // 0 일때. 스레드 간의 커뮤니케이션 시작. 다른 스레드 접근을 차단
    }
}

// signal. 세마포어 잠금해제
signal(semaphore *S) {
    S->value++;
    if (S->value <= 0)
    {   // 스레드를 리스트에서 꺼내어
        P = S->list.removeFirst()
        // 활성화
        wakeup(P);
    }
    else
    {   // value가 1일때. 펜딩되어 있는 스레드 없음.
    }
}

```
-  스레드와 스레드가 (혹은 프로세스끼리) 일대일로 커뮤니케이션하는 동안 다른 스레드의 접근을 금지하는 Mutual exclusion (상호배제) 원칙으로 작동.
-  세마포어 값은 생성시 1로 초기화되고 스레드가 접근해 wait() 을 콜하면 세마포어 값이 0으로 바뀌며 다른 스레드의 접근을 금지함.
-  세마포어 값이 0 일때 wait() 을 콜하며 접근하는 스레드는 세마포어 리스트에 적재되고 sleep() 상태로 들어가며 본인들이 점유하고 있던 코어의 제어권을 OS 스케줄러에게 넘겨 CPU 는 다른 작업을 수행할 수 있게함.
-  처리가 끝난 스레드는 세마포어의 signal() 을 콜, 세마포어 값을 1로 바꾸고 세마포어 리스트 상에 적재되어 있던 다음 스레드를 꺼내 wakeup() 를 콜, 다시 활성화 상태로 바꿔줌.

value 값이 0과 1로 마치 플래그처럼 작동하는 세마포어를 **Binary 세마포어** 라고 하며 접근 가능한 스레드는 1개로 제한된다. 반면에 **Counting 세마포어**는 다수의 스레드 접근을 허용할 수 있고 wait() 을 콜할 때마다 세마포어 value 값을 --1 처리하여 이때 음수가 되는 value 값은 리스트에 적재된 스레드의 개수를 의미하게 된다.

- Binary Semaphore 는 클래식한 사용방식으로 다른 스레드의 동시접근을 방지하여 일대일의 커뮤니케이션 상태로 만들어줌.
- Counting Semaphore 는 접근 가능한 스레드의 개수를 제한할 수 있으며 역시 raceCondition 은 피할 수 없기 때문에 데이터의 확인, 상수값의 Read, 데이터 조작 직전까지의 실행 상황에 쓰일 수 있다. 대표적인 예시로는 프린터의 풀링시스템, 스레드 풀을 사용하는 웹서버환경이 있음.
- 접근권한을 얻어 실행 중인 ( semaphore.value < 1 ) 스레드가 있는 상태에서 또다른 스레드가 signal() 을 콜하면 value 가 ++1 되고 세마포어 리스트에 있는 프로세스 하나가 활성화되어 대부분의 경우 의도치 않은 오류가 발생하게 됨.
<br>
<br>

### 메시지 출력 시스템의 예시
메시지전송 시스템에 연결된 여러 디바이스 (프린터, 모니터, 사운드시스템 등등) 에서 유저의 입력 메시지가 출력되는 상황을 가정해보자.  

```swift

import Foundation

struct Message {
    let description : String
    init(description: String) {
        self.description = description
    }
}

class PrintDevice : Thread {
    var message : Message?
    var no : Int
    
    init(no: Int) {
        self.no = no
    }
    
    override func main() {
        while(true) {
            showMessage()
            Thread.sleep(forTimeInterval: 0.5)
        }
    }
    
    func showMessage() {
        if let msg = message {
            Thread.sleep(forTimeInterval: 1)    // 디바이스가 사용 중인 상태를 표현하기 위해 추가
            print("[Device : PRINT] no : \(no) ,description : \(msg.description)")
            message = nil
            MessageSystem.semaphore.wait()
            MessageSystem.channels.append(self)
            MessageSystem.semaphore.signal()
        }
    }
}

class MessageSystem : Thread {
    
    static var msgQueue : [Message] = []
    static var channels : [PrintDevice] = []
    static let semaphore = DispatchSemaphore(value: 1)
    
    override func main() {
        MessageSystem.update()
    }
    
    static func update() {
        while(true) {
            // 확인, 입력, 제거 등 큐의 상태를 확인하거나 조작하는 경우 세마포어 값을 1 -> 0 변경
            semaphore.wait()
            if !msgQueue.isEmpty
            {
                if let channel = findChannel() 
                {
                    sendMessage(message: msgQueue.removeFirst(), channel: channel)
                }
                else
                {
                    print("[MessageSystem : UPDATE] All channels are full")
                    Thread.sleep(forTimeInterval: 2)    // 채널이 모두 사용중 일때 위의 print 문이 여러번 출력되는 상황을 방지
                }
            }
            // 실행 완료. 세마포어 값을 0 -> 1 변경. 큐에서 다음 실행할 스레드를 꺼냄
            semaphore.signal()
        }
    }
    
    // 사용가능한 채널 찾기
    static func findChannel() -> PrintDevice? {
        if channels.isEmpty
        {
            return nil
        }
        else
        {
            return channels.removeFirst()
        }
    }
    
    // 유저 입력.
    static func loadMessage(_ message: String) {
        semaphore.wait()
        msgQueue.append(Message(description: message))
        semaphore.signal()
    }
    
    static func sendMessage(message:Message, channel:PrintDevice ) {
        channel.message = message
        print("[MessageSystem : SEND]  queue_count : \(channels.count), channel : \(channel.no) , message: \(message.description)")
    }
}
 
let messageSystem = MessageSystem()
messageSystem.start()

for i in 1...3 {
    let device = PrintDevice(no: i)
    device.start()
    MessageSystem.channels.append(device)
}


// Main thread, 유저입력
// 만약 이 입력 부분을 멀티스레드화 해서 입력 채널을 늘리고 수신자를 지금의 PrintDevice 에서 전체 입력 채널로 바꾸어 주면 그룹 채팅시스템이 만들어진다.
while(true) {
    if let str = readLine() {
        if str == "`"
        {
            break
        }
        else
        {
            if str != "" {
                MessageSystem.loadMessage(str)
            }
        }
    }
}

```
<br>
##### 결과
<img src="/img/designPattern/2024-08-29-eventQueuePattern_0.png" style="width:100%">

채널이 모두 사용 중일 때에도 입력은 작동되고 사용이 끝난 채널이 확인되면 큐에 적재되어 있던 메시지 'd' 가 발송된다. 만약 동기화된 패턴을 적용한다고 가정하면 'a,b,c' 의 입력이후 이들의 출력이 끝날 때까지 입력이 먹통이 되거나 3개의 출력 채널을 모두 사용하지 못하고 하나의 채널만 점유하게 되는 상황이 발생할 것이다. 
<br>

총 5개의 스레드가 실행 중인데
* Main 스레드 : 유저의 키 입력을 감지하고 MessageSystem 으로 전달
* MessageSystem : 큐의 쌓인 메시지를 지속적으로 확인하고 사용가능한 디바이스 채널을 찾아 메시지를 전송
* PrintDevice 스레드 3개 : 메시지 수신을 확인하고 출력

MessageSystem 은 과도한 입력이 들어오면 이를 큐에 모아두었다가 사용 가능한 채널이 생길 때마다 바로바로 메시지를 전달해주기 때문에 입력과 출력에서 서로의 오버로드를 체크할 필요가 없다. 메시지의 입력과 출력의 시점을 큐를 사용하여 분리해주기 때문에 동기화를 신경쓸 필요도 없다. 또한, 입력 받은 메시지들의 필터링이 필요하다거나 출력할 디바이스를 제한하거나 늘려야 할 경우, 즉 입력된 메시지나 수신채널의 조작이 필요한 경우 큐와 메시지전달 로직을 적절히 수정하면 다양한 형태의 이벤트 처리를 커스터마이즈 할 수 있다. 큐와 수신자에 제어권을 넘겨줄 수 있게 되는 것이다.  

### 주의할 점
* 중앙 이벤트 큐는 전역변수와 비슷하다. 잘 사용해야 한다.
* 입력과 처리의 **시점** 이 분리되어 있기 때문에 입력 시점의 특정한 상태가 필요한 경우 (타임 스태프와 같이) 이를 기록해야 할 수도 있다. 시점의 분리 (비동기) 가 필요하지 않다면 옵저버패턴의 사용을 먼저 고려해 보자. 
* 큐는 요청을 받는 쪽에 제어권을 제공하기 때문에 보내는 쪽에서 처리 응답을 받아야 한다면 큐의 사용은 적합하지 않을 수도 있고 (수신자의 응답이 없을 수도 있으므로), 비동기 작동으로 인한 슨환루프에 빠지면 동기 시스템에 비해 오류를 찾아내기 힘들 수도 있다. 이 문제는 이벤트를 처리하는 수신측 코드에서는 이벤트를 보내지 않는 방법으로 회피할 수 있음.
<br>