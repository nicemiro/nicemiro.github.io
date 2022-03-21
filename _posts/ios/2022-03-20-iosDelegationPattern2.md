---
title: Delegation Pattern (위임패턴)(2)
titleEn: Delegation Pattern in iOS(2)
author: BabyK
date: 2022-03-20
category: iOS
layout: post
---

### Delegation in practice

<img src="/img/2022-03-20-iosDelegationPattern2_1.png" >

버튼 클릭으로 화면을 이동하면서 라벨의 문구를 바꿔주는 예시이다.  
실행 했을 때 첫 화면인 FirstView가 떠맡은 일을 해주는 Delegator이다. NextView가 일을 주는 Delegate.

```swift
protocol SendDataDelegate : AnyObject{
    func sendData (message: String)
}
class NextViewController: UIViewController {
    @IBOutlet weak var viewLabel: UILabel!
    var message : String?
    weak var delegate :SendDataDelegate?

    override func viewDidLoad() {
        super.viewDidLoad()
        if let from = message {
            self.viewLabel.text = from
        }
    }
    @IBAction func PreviousViewButton(_ sender: UIButton) {
        self.delegate?.sendData(message: "SecondView에서 보냄")
        self.presentingViewController?.dismiss(animated: true, completion: nil)
    }
}
```

NextView 컨트롤러를 보면 프로토콜 변수 `delegate` 을 선언했다.    
Optional Type으로 선언하여 Delegate객체가 없어도 NextView 컨트롤러가 작동할 수 있게 했다.
```swift
weak var delegate :SendDataDelegate?
```
`PreviousViewButton` 펑션에서 `delegate.sendData`를 실행했다.  
```swift
self.delegate?.sendData(message: "SecondView에서 보냄")
```
실행 로직은 일을 받아 처리할 첫 화면 컨트롤러인 ViewController에 정의되어 있다.

<br>
```swift
class ViewController: UIViewController , SendDataDelegate{
    @IBOutlet weak var viewLabel: UILabel!
    override func viewDidLoad() {
        super.viewDidLoad()
    }
    @IBAction func nextViewButton(_ sender: UIButton) {
        guard let viewController = self.storyboard?.instantiateViewController(withIdentifier: "NextViewController") as? NextViewController else {return}
        viewController.name = "FistView에서 보냄"
        viewController.delegate = self
        self.present(viewController, animated: true, completion: nil)
    }
}
extension ViewController {
    func sendData(name: String) {
        self.viewLabel?.text = name
    }
}
```
<br>
첫 번째 화면 컨트롤러인 ViewController를 보면 `sendDataDelegate` 프로토콜을 채택하여 구현했고  

```swift
class ViewController: UIViewController , SendDataDelegate{
```
NextViewController 객체를 만들고 `NextViewController.delegate` 변수안에 self 객체를 넣었다.
```swift
self.storyboard?.instantiateViewController(withIdentifier: "NextViewController") ...
viewController.delegate = self
```

이렇게 하면`self.present(viewController, animated: true, completion: nil)` 에서  
두 번째 화면으로 전환된 이후에도 함께 넘긴 self객체를 이용해 첫 번째 화면을 변경 할 수 있게 된다.  

<img src="/img/2022-03-20-iosDelegationPattern2_2.png" >

<img src="/img/2022-03-20-iosDelegationPattern2_3.png" >

<br>
그렇기 때문에 NextView 화면에서 'PreviousViewButton' 버튼을 클릭시  
FirstView 화면의 라벨 문구가 수정될 수 있다.

```swift
// 두 번째 화면의 버튼기능 정의
@IBAction func PreviousViewButton(_ sender: UIButton) {
        self.delegate?.sendData(message: "SecondView에서 보냄") // self는 첫번째 화면 객체
        self.presentingViewController?.dismiss(animated: true, completion: nil)
    }
```
<br>
<br>
 
<span style="font-size:200%">Why?</span>

애플의 [공식 문서][1]를 인용하자.
>Delegation is a simple and powerful pattern in which one object in a program acts on behalf of, or in coordination with, another object. The delegating object keeps a reference to the other object—the delegate—and at the appropriate time sends a message to it. The message informs the delegate of an event that the delegating object is about to handle or has just handled. The delegate may respond to the message by updating the appearance or state of itself or other objects in the application, and in some cases it can return a value that affects how an impending event is handled. The main value of delegation is that it allows you to easily customize the behavior of several objects in one central object.

<br>
요약하자면 위임패턴의 장점은 
- Delegate는 Delegator가 작업시 옵저버의 역할을 수행가능. Notification을 받거나 이후 행위 컨트롤이 가능함.
- 여러 오브젝트의 행위를 한 곳에서 쉽게 수정 가능  

공식문서의 (Objective-C 문서이지만) [memory management][2] 관련 내용을 살펴보면 retain cycle에 따른 문제 (예를들면 화면 전환된 이후 이전 화면의 메모리 해제)를 다루고 있는데  
Swift에도 메모리 관리시의 유의사항이 있는 것 같다. 아마도 한 컨트롤러 내에서 다른 컨트롤러 객체를 띄우고 이것을 Delegation 패턴을 사용해 연결하는 것이 여기서 말하는 'Use Weak References to Avoid Retain Cycles' 의 한 종류가 아닐까 싶은데 이 부분은 따로 정리해 봐야 할 것 같다.

<br>



[1]: https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/Delegation.html
[2]: https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmPractical.html#//apple_ref/doc/uid/TP40004447-1000810
