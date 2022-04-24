---
title: Selector란
titleEn: Using Selector in iOS
author: BabyK
date: 2022-04-22
category: iOS
layout: post
---

### Selector

>Some Objective-C APIs—like target-action—accept method or property names as parameters, then use those names to dynamically call or access the methods or properties. In Swift, you use the #selector and #keyPath expressions to represent those method or property names as selectors or key paths, respectively.

Objective-C에서 함수명을 파라메터로 사용하는 API를 위해 사용되던 것이 selector라고 한다.  
한마디로 함수에 함수를 인자로 넘겨줄때 함수 포인터 정도로 쓰이던 기능인듯 하다.  
셀렉터의 파라메터가 스트링타입이니 동적으로 선택이 가능하다는 점이 조금 다른듯 보인다.  
(Swift에서는 스트링타입이 아니므로 dynamic call이 불가능 할듯)

>In Swift, you create a selector for an Objective-C method by placing the name of the method within the #selector expression: #selector(MyViewController.tappedButton(_:)).

Swift에서 Objective-C 펑션을 호출할때 Selector를 사용한다.  
iOS의 Objective-C API 레거시 때문이 아닐까 추측해본다.  
(UIKit이 아닌 SwiftUI에서는 변경되었을까? iOS를 딥하게 사용하기 위해서는 ObjC에 대한 학습이 필요해 보인다.)

<br>
#### 사용예시  
```swift
class ViewController: UIViewController {
    @IBOutlet weak var selectorBtn: UIButton!

    @IBAction func testSelector(_ sender: UIButton) {
        let action = #selector(self.tappedButton)
        self.selectorBtn.addTarget(self, action: action , for: .touchUpInside)
    }

    // @objc Objective C의 런타임과 인터렉트 하기 위해 사용한다고 한다. (레거시코드 때문이 맞는듯)
    @objc func tappedButton(_ sender: UIButton?) {
        print("tapped button")
    }
}
```

<br>
#### Reference
- [https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/Selector.html][1]{:target="_blank"}  
- [https://developer.apple.com/documentation/swift/using_objective-c_runtime_features_in_swift][2]{:target="_blank"}  

[1]: https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/Selector.html
[2]: https://developer.apple.com/documentation/swift/using_objective-c_runtime_features_in_swift
