---
title: '@State, @ObservedObject, @StateObject, @Published, @EnvironmentObject 정리'
titleEn:
author: BabyK
date: 2023-03-25
category: iOS
layout: post
published: true
---
<br>
#### @State  
값이 바뀔 때마다 바인딩된 뷰를 자동으로 업데이트 해주는 기본적인 프로퍼티 매크로.  
Simple Primitive 타입의 프로퍼티로 선언되며 현재 뷰에 속한 서브뷰에서 @State 값을 변경하기 위해서는  
@Binding 을 사용한 일대일 매핑이 필요함 ($name).  


```swift
struct PlayerView: View {
    @State private var isPlaying: Bool = false // Will update when 'isPlaying' changes.

    var body: some View {
        VStack {
            PlayButton(isPlaying: $isPlaying) // Pass a binding to a subView
            // ...
        }
    }
} 
struct PlayButton: View {
    @Binding var isPlaying: Bool // Play button now receives a binding.

    var body: some View {
        Button(isPlaying ? "Pause" : "Play") {
            isPlaying.toggle()
        }
    }
}
```
<br>

#### @ObservedObject 와 @StateObject  
단순 Primitive 값에 사용되는 @State와 다르게 복잡한 객체와 뷰 사이의 바인딩에는      
@ObservedObject 와  @StateObject 를 사용함.  
일대일 매핑이 아닌 겍체 하나로 다수의 뷰와 매핑이 가능하며 이때 이 객체의 프로퍼티에    
@Published 를 사용하여 프로퍼티 값이 변화 될때 마다 바인딩된 뷰들의 Reload 를 일으킴.  

@ObservedObject 나 @StateObject 는 SwiftUI 라이브러리에 속하고 @Published 는 Foundation 에 속함.  
그래서 @Published 는 뷰 없이도 사용이 가능하며 class-only Wrapper로 struct에는 사용이 불가하다.  

뷰를 사용하지 않는 일반적인 @Published 와 subscriber 의 예시.  
```swift
class Weather {
	@Published var temperature: Double
    init(temperature: Double) {
    	self.temperature = temperature
    }
}

let weather = Weather(temperature: 20)
cancellable = weather.$temperature
	.sink() {   // Publisher의 값을 Observing
    	print("Temperature now: \($0)")
}
weather.temperature = 25

// or

let integers = (0...3)
integers.publisher  // @Published
    .sink { print("Received \($0)") }

// Prints:
//  Received 0
//  Received 1
//  Received 2
//  Received 3
```
<br>

사용방법과 효과는 동일하지만 @ObservedObject 와 @StateObject 사이에는 아주 중요한 차이점이 있다.  
<span style="color:red"> @ObservedObject 프로퍼티를 갖고 있는 뷰는 리프레쉬 될 때마다 해당 프로퍼티 객체를 새로 생성한다는 것. </span>  
프로퍼티 값의 변화, 메모리할당 및 퍼포먼스 (유의미하지 않을 수 있지만) 와 관련해서 둘은 다르게 사용되어야 한다.  

```swift
import SwiftUI

final class CounterViewModel: ObservableObject {
    private(set) var count = 0

    func incrementCounter() {
        count += 1
        objectWillChange.send()
    }
}

struct RandomNumberView: View {
    @State var randomNumber = 0

    var body: some View {
        VStack {
            Text("Random number is: \(randomNumber)")
            Button("Randomize number") {
                randomNumber = (0..<1000).randomElement()!
            }
        }.padding(.bottom)
        
        CounterView()
    }
}

struct CounterView: View {
    @ObservedObject var viewModel = CounterViewModel()  // 랜덤 버튼을 누를때마다 count 값이 초기화 됨.
//    @StateObject var viewModel = CounterViewModel()   // 랜덤 버튼을 눌러도 count 값은 변화 없음.

    var body: some View {
        VStack {
            Text("Count is: \(viewModel.count)")
            Button("Increment Counter") {
                viewModel.incrementCounter()
            }
        }
    }
}

```
<br>

결론적으로  
- 바인딩되는 객체를 <span style="color:red">외부에서 생성 </span>해 주입해야 한다 -> @ObservedObject
- <span style="color:red"> 화면 내부에서 프로퍼티를 생성 </span>하여 현재 뷰 및 서브뷰의 Refresh 에도 안전하게 사용해야 한다 -> @StateObeject  

@ObservedObject는 아래와 같은 용도로 하나의 객체를 여러 뷰에서 공유 할때, 즉 외부에서 생성해 사용하는 것이 좋다.  
애플의 Doc 에서도 Default or initial value for the observed object 를 정의하지 말라고 함.  

```swift
struct CounterView: View {
    @ObservedObject var viewModel : CounterViewModel

    init(viewModel: CounterViewModel) {
        self.viewModel = viewModel
    }
    var body: some View {
        VStack {
            Text("Count is: \(viewModel.count)")
            Button("Increment Counter") {
                viewModel.incrementCounter()
            }
        }
    }
}
```
<br>

#### @EnvironmentObject
하나의 객체를 여러 뷰에서 공유 할때 사용하는 또다른 매크로 @EnvironmentObject가 있다.   
App, View, NavigationStack 등, @EnvironmentObject 가 사용된 하위 모든 뷰에 전역적으로 사용 가능하고  
마찬가지로 ObservableObject 프로토콜을 conform 해야하고 @Published로 뷰의 리로드를 트리거링 한다.  
```swift

@main
struct EnvironmentTestApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(DataModel())
//            RandomNumberView()
        }
    }
}

class DataModel: ObservableObject {
    @Published var count: Int = 0
    func increment() {
        count += 1
    }
}

struct ContentView: View {
    var body: some View  {
        
        View1()
        Spacer()
            .frame(height: 30)
        View2()
    }
}

struct View1: View {
    @EnvironmentObject var environmentObject: DataModel
    
    var body: some View {
        VStack {
            Text("@EnvironmentObject in view1: \(environmentObject.count)")
            Button("+ count") {
                environmentObject.increment()
            }
        }
    }
}

struct View2: View {
    @EnvironmentObject var environmentObject: DataModel // 뷰 내부에서 상태를 관리하는 뷰
    var body: some View {
        VStack {
            Text("@EnvironmentObject in view2: \(environmentObject.count)")
            Button("+ count") {
                environmentObject.count += 1
            }
        }
    }
}
```
<br>
#### Reference
<a href="https://www.avanderlee.com/swiftui/stateobject-observedobject-differences/" target="_blank">https://www.avanderlee.com/swiftui/stateobject-observedobject-differences/ </a>
<a href="https://www.hackingwithswift.com/quick-start/swiftui/how-to-use-environmentobject-to-share-data-between-views" target="_blank">https://www.hackingwithswift.com/quick-start/swiftui/how-to-use-environmentobject-to-share-data-between-views</a>