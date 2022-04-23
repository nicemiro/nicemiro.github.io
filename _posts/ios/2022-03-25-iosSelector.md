---
title: Selector란
titleEn: Using Selector in iOS
author: BabyK
date: 2022-04-22
category: iOS
layout: post
---

### Selector 란?

공식 문서를 보자.
> In Objective-C, a selector is a type that refers to the name of an Objective-C method. In Swift, Objective-C selectors are represented by the Selector structure, and you create them using the #selector expression.

대충 Selector란 Objective C에서 쓰이는 함수 포인터 인듯하다.  
함수에 파라메터로 함수를 넘겨줘야 할 경우 *Selector* 를 사용하는 듯 한데  
아마도 포인터의 사용을 없애기 위함이 아니었을까 일단 추측하고 넘어간다.

<br>
>You use a selector created from a string when you want your code to send a message whose name you may not know until runtime.

런타임시에 동적으로 펑션지정을 가능하게 한다. C의 동적메모리 할당과 비슷한 용도로 사용되는 유용한 기능이다.  

사용예시  
```swift
class ViewController: UIViewController {
    @IBOutlet weak var selectorBtn: UIButton!

    @IBAction func testSelector(_ sender: UIButton) {
        let action = #selector(self.tappedButton)
        self.selectorBtn.addTarget(self, action: action , for: .touchUpInside)
    }

    @objc func tappedButton(_ sender: UIButton?) {
        print("tapped button")
    }
}
```
-  **@objc** : Objective C의 런타임과 인터렉트 하기 위해 사용. (Swift를 제대로 사용하기 위해서는  
 Objective-C 에 대한 학습이 필요해 보인다.)

<br>
*Reference*  
- [https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/Selector.html][1]{:target="_blank"}  
- [https://developer.apple.com/documentation/swift/using_objective-c_runtime_features_in_swift][2]{:target="_blank"}  
- [https://developer.apple.com/documentation/objectivec/sel][3]{:target="_blank"}  

[1]: https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/Selector.html
[2]: https://developer.apple.com/documentation/swift/using_objective-c_runtime_features_in_swift
[3]: https://developer.apple.com/documentation/objectivec/sel
