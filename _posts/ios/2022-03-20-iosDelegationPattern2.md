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
실행 했을 때 첫 화면인 FirstView가 떠맡은 일을 해주는 Delegate이다. NextView가 일을 주는 Delegator.

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
 
<span style="font-size:200%">왜 Delegation을 사용할까</span>

Objective-C에 관한 내용이지만 애플의 [공식 문서][1]를 인용하자.
>Delegation is a simple and powerful pattern in which one object in a program acts on behalf of, or in coordination with, another object. The delegating object keeps a reference to the other object—the delegate—and at the appropriate time sends a message to it. The message informs the delegate of an event that the delegating object is about to handle or has just handled. The delegate may respond to the message by updating the appearance or state of itself or other objects in the application, and in some cases it can return a value that affects how an impending event is handled. The main value of delegation is that it allows you to easily customize the behavior of several objects in one central object.

<br>
요약하자면 위임패턴의 장점으로  
- Delegator는 Delegate이 작업시 옵저버의 역할을 수행가능. Notification을 받거나 이후 행위 컨트롤이 가능함.
- 여러 오브젝트의 행위를 한 곳에서 쉽게 수정 가능  

한 객체안의 기능들을 여러 오브젝트에서 사용하게 하고 싶을 때 상속을 통해 구현하게 된다면 부모 클래스의 모든 기능들을 일일이 신경써야 하고 리소스 또한 낭비가 생길 수 있다.  

더 중요한 것은 문서상에서도 강조하듯이 여러 오브젝트(Delegate)들이 공유하는 기능들을 Delegate의 (프로토콜에 명시된) 펑션들만 수정하는 것으로 한방에 해결 할 수 있고 필요한 경우 Delegator의 소스 수정없이 Delegate를 교체 가능하다는 점.
<br>
여러가지 뷰에서 사용되는 같은 기능들을 delegate의 소스만 수정하면 모두 교체 가능하고 delegate에서 리턴되는 값(notification)을 이용해 delegator객체는 이벤트의 관찰 및 제어가 가능하다는 점을 기억하자.  
<br>

[1]: https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/Delegation.html
