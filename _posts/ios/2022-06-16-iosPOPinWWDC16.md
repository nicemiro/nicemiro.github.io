---
title: 프로토콜 오리엔티드 프로그래밍(POP) from WWDC16
titleEn: Protocol and Value Oriented Programming in UIKit Apps from WWDC16
author: BabyK
date: 2022-06-16
category: iOS
layout: post
---

### Protocol Oriented Programming (POP)
기존의 자바로 대표되는 OOP의 클래스 상속을 통한 구현이 아닌 Swift의 Protocol채택을 이용한 기능구현.  
부모클래스에 대한 의존도와 복잡성을 줄인 유연하고 확장가능하며 가독성있는 코드 구현이 가능함.

애플 WWDC16 세션의 'POP in UIkit Apps' 내용중.
- Customization through composition (for complex behaviors)
- Protocols for generic, reusable code (fast and safety polymorphism)
- Taking advantage of value semantics
- Local reasoning with value types(small and focused)

<br>
### 1. Taking advantage of value semantics
#### <strong>'Inheritance is another place where you really sacrifice the ability to use local reasoning'</strong>  
Class 객체의 사용시 <span style="color:red"><strong>상속 복잡도</strong></span>가 증가하며, 객체 복사시 <span style="color:red"><strong>동일 메모리주소 참조</strong></span>로 인해 값이 변경될 수 있음.  
OOP에서와 같이 Simple model type에 대해서만 Value형인 Structure를 사용할 것이 아니라  
적극적으로 structure를 사용하자.

```swift
class Dream {
 var description: String
 var creature: Creature
 var effects: Set<Effect>
 ...
}
var dream1 = Dream(...)
var dream2 = dream1

// 클래스 사용시 dream1,2 의 description이 모두 변경됨.
dream2.description = "Unicorns all over" 
```
<br>
### 2. Protocols for generic, reusable code

UITableViewCell 에서 사용될 레이아웃에 UIView와 스프라이트킷(SKNode) 겍체를 함께 사용하고자 할 때.

<img src="/img/2022-06-16-iosPOPinWWDC16_1.png
" style="width:90%;height:70%">
<img src="/img/2022-06-16-iosPOPinWWDC16_2.png" style="width:90%;height:80%">

<br>

```swift
    // class DecoratingLayoutCell : UITableViewCell { 
    //  var content: Layout            // UIVIew or SKNode
    //  var decoration: Layout         // UIVIew or SKNode
    //  ....
    //  }
    // 클래스 안에 두 레이아웃을 구현시 한 cell에 종속됨
    
// content, decoration 두 레이아웃과 로직 layout() 으로 structure를 구성하여
// UITableViewCell로부터 DecoratingLayout 레이아웃을 분리함
struct DecoratingLayout {
    var content: Layout            // UIVIew or SKNode
    var decoration: Layout         // UIVIew or SKNode
    mutating func layout(in rect: CGRect) {
        content.frame = ...
        decoration.frame = ...
    }
}
protocol Layout {
    var frame: CGRect { get set }
}

// Layout프로토콜을 채택한 UIView나 SKNode객체 둘다 선택적으로 적용할 수 있음.
// UIView용, SKNode용 두 개의 DecoratingLayout 스트럭쳐를 따로 구성할 필요가 없음.
extension UIView : Layout {}
extension SKNode : Layout {}
```

<br>

위의 *DecoratingLayout* 스트럭쳐를 사용한 뷰의 구성
```swift
// View Layout
// DecoratingLayout을 모든 UIView에 (물론 UIView를 상속받은 UITableViewCell 역시) 사용가능하게 변경함.
class DreamCell : UITableViewCell {
    ...
    override func layoutSubviews() {
    var decoratingLayout = DecoratingLayout(content: content, decoration: decoration)
    decoratingLayout.layout(in: bounds)
    }
 }
class DreamDetailView : UIView {
    ...
    override func layoutSubviews() {
    var decoratingLayout = DecoratingLayout(content: content, decoration: decoration)
    decoratingLayout.layout(in: bounds)
    }
}
```

<br>

Generic을 사용해 *DecoratingLayout* 을 수정한 버젼.  

```swift
struct DecoratingLayout<Child : Layout> {
    var content: Child
    var decoration: Child
    mutating func layout(in rect: CGRect) {
    content.frame = ...
    decoration.frame = ...
    }
}
protocol Layout {
    var frame: CGRect { get set }
}
extension UIView : Layout {}
extension SKNode : Layout {}
```

<strong>Class의 상속을 통한 polymorphism 대신<span style="color:red"> Protocol</span>을 사용해서<strong>  
- 관계가 없는 두 타입의 객체( UIView, SKNode ) 에 layout() 펑션을 추가했고  
- UIKit과도 분리되어 <span style="color:red">Reusable</span>해졌으며
- Generic을 사용하여 코드의 양이 줄었음.  

<br>
### 3. Customization through composition with associated type

아래 이미지와 같이 컨텐츠를 중첩 (cascading)가능하도록 기능을 추가하려 한다.  

<img src="/img/2022-06-16-iosPOPinWWDC16_3.png" style="width:80%;height:70%">

<br>

```swift
protocol Layout {
    mutating func layout(in rect: CGRect)

    // content를 프로토콜로 분리, array로 변경하여 중첩가능하게 하고 associatedtype을 사용해서 contents 배열에서 사용할 타입을 강제함. (struct나 class에서의 generic 사용처럼)
    associatedtype Content
    var contents: [Content] { get}

    // content를 프로토콜 자신을 리턴하게 만들면 UIView와 SKNode를 mix해서 사용할 수 있음.
    // var contents: [Layout] { get }   // UIView and SKNode
}

struct DecoratingLayout<Child : Layout, Decorating : Layout where Child.Content == Decoration.Content> : Layout { // where절로 content, decoration 에서 같은 타입 사용을 강제함
    var content: Child
    var decoration: Decoration

    mutating func layout(in rect: CGRect)
    typealias Content = Child.Content   // associatedtype사용으로 추가됨
    var contents: [Content] { get}  
}


func testLayout() {

    extension UIView : Layout {}

    let child1 = UIView()
    let child2 = UIView()

    var layout = DecoratingLayout(content: child1, decoration: child2)
    layout.layout(in: CGRect(x: 0, y: 0, width: 120, height: 40))
    ...
}
```

길게 설명해 놨지만 Class 대신 Struct 사용하라는 얘기다.  
- Classes instances are expensive. When you make another object, you have an extra heap allocation.
- Structs are cheap
- Composition is better with value semantics - With value types you have much better encapsulation without
worrying about someone else modifying the copy that you're using.

[Associatedtype][2]{:target="_blank"} 사용에 대한 예시는 Swift Doc을 봐도 정확히 감이오지 않는다. 컴파일러가 해당 객체안에서 사용될 타입을 추론가능하게 하고 특정 타입을 강제할 수 있다는 정도. 즉 프로토콜에서 사용 할 수 있는 Generic의 용도라는 것.

<br>
#### Reference
- [https://developer.apple.com/videos/play/wwdc2016/419/][1]{:target="_blank"}  
- [https://docs.swift.org/swift-book/LanguageGuide/Generics.html#ID189][2]{:target="_blank"}  

[1]: https://developer.apple.com/videos/play/wwdc2016/419/
[2]: https://docs.swift.org/swift-book/LanguageGuide/Generics.html#ID189
