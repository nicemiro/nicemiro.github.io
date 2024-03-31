---
title: 2D 벡터의 이동
titleEn: 2D Vector Translation
author: BabyK
date: 2024-03-29
category: GameDev
layout: post
published: true
---
<br>
화면상에 두 비행체가 각각 두 지점을 향해 같은 탄환을 발사하는 상황을 생각해보자.  

<div class="screenShots" align="left">
<img src="/img/gameDev/2024-03-29-gameDev_rotationOfVector00.png" style="width:50%;height:50%">
</div>
<br>

4번에 걸쳐 이동하는 두 탄환의 좌표를 찍어주는 위의 화면을 정리해보면   

vx1 = (B.x - A.x) / 4  
vy1 = (B.y - A.y) / 4  

vx2 = (D.x - C.x) / 4  
vy2 = (D.y - C.y) / 4  

이 공식은 한가지 문제가 있는데   
두 탄환은 화면상의 좌표를 각각 4번씩 찍어주며 이동하기 때문에 거리에 상관없이 도착 시간이 같다.  
시작점과 끝점 사이의 거리가 길어질수록 탄환의 속도가 빨리지고 가까울수록 속도가 느려지고 있는 것이다.  

<div class="screenShots" align="left">
<img src="/img/gameDev/2024-03-29-gameDev_rotationOfVector02.png" style="width:50%;height:50%">
</div>
<br>

현실 세계에서 같은 두 탄환의 속도는 고정되어 있고 거리에 따라 도착 시간이 늘어나거나 줄어들어야 한다.  
게임에서 화면의 주사율은 대게 60 FPS,  
화면이 깜빡이며 좌표를 새로 찍어주는 시간은 초당 60번 (1/60) 으로 고정되어 있다.  
각각 현재 좌표에서 거리가 5, 10 인 지점으로 두 탄환을 발사한다면 1 프레임당 각각의 이동거리는 다음과 같다.  
<br>

<div class="ex" align="center">
0.08333 = 5 / (1/60)<br>
0.16666 = 10 / (1/60)
</div>

<br>

시간이 고정되어 있기 때문에 거리가 10인 탄환이 2배 더 빠른 셈인데  
현실 세계에서 두 탄환의 1프레임당 이동 거리인 속도는 같아야 한다.  

화면상에서 탄환의 시작점 A와 도착지점 B의 좌표를 정했다면  
피타고라스의 정리로 두 좌표지점의 x,y 값을 이용해 탄환의 총 이동거리인 r을 구할 수 있다.  
<div class="screenShots" align="left">
<img src="/img/gameDev/2024-03-29-gameDev_rotationOfVector01.png" style="width:50%;height:50%">
</div>
<br>

<div class="ex" align="center">
rr  = 4 X 4 + 3 X 3  <br>
r = 5
</div>
<br>

A와 B사이의 탄환 이동거리인 r이 5 이므로 1프레임당 움직이는 x, y 의 거리는  

<div class="ex" align="center">
x : (B.x - A.x) / 5  <br>
y : (B.y - A.y) / 5  <br>
(4 / 5 , 3 / 5)  <br>
=> (0.8, 0.6)
</div>
<br>

같은 방법으로 삼각형의 윗변 r이 10인, 좌표 (8,6)만큼 이동하는 탄환의 속도는   

<div class="ex" align="center">
(8 / 10 , 6 / 10)  <br>
=> (0.8, 0.6)
</div>
<br>

탄환의 속도가 (0.8, 0.6) 으로 같아졌다.  

두 탄환의 속도가 같기 때문에 2배의 이동거리와 2배의 도착시간이 정확히 화면에 표현된다.  
1프레임당 이동 값인 (0.8, 0.6) 좌표가 바로 방향성이 있는 힘, Vector 다.  

```swift
protocol GameClass {
    func update(_ updateDt: TimeInterval)
}

class GameEngine {
    private var lastUpdateTime : TimeInterval = 0
    private var updateDt = Double(0)
    private var targetFrameDuration : Double = 1.0/60   // 60FPS
    private var gameClass : GameClass
    
    init(gameClass: GameClass) {
        self.gameClass = gameClass
    }
    
    func run() {
        while true {
            let currentTime = Date().timeIntervalSince1970
            
            if lastUpdateTime == 0 {
                lastUpdateTime = currentTime
                continue
            }
            updateDt += currentTime - lastUpdateTime
            
            if updateDt >= targetFrameDuration {
                gameClass.update(updateDt)
                updateDt = 0
            }
            
            lastUpdateTime = currentTime
        }
    }
}

class VectorTranslation : GameClass {
    private var coordA = CGPoint(x: 2, y: 3)
    private var coordB = CGPoint(x: 6, y: 6)

    private var coordC = CGPoint(x: 1, y: 2)
    private var coordD = CGPoint(x: 9, y: 8)
    
    private var vector1 = CGPoint(x: 0, y: 0)
    private var vector2 = CGPoint(x: 0, y: 0)
    private var projectileSpped = Double(0.3)   // 탄환종류별 속도가중치
    
    init() {
        setVectors()
    }
    func setVectors(){
        let adjacent1 = coordB.x - coordA.x // 밑변 x
        let opposite1 = coordB.y - coordA.y // 높이 y
        let hypotenuse1 = log2(sqrt(adjacent1) + sqrt(opposite1))   // 윗변 r
        vector1.x = adjacent1 / hypotenuse1
        vector1.y = opposite1 / hypotenuse1
        
        let adjacent2 = coordD.x - coordC.x
        let opposite2 = coordD.y - coordC.y
        let hypotenuse2 = log2(sqrt(adjacent2) + sqrt(opposite2))
        vector2.x = adjacent2 / hypotenuse2
        vector2.y = opposite2 / hypotenuse2
    }
    
    func update(_ updateDt: TimeInterval) {

        // B좌표에 도달하지 않았다면 계속 이동
        if coordA.x < coordB.x
        {
            coordA.x += vector1.x * projectileSpped
            coordA.y += vector1.y * projectileSpped
        }
        else
        {
            // 도착. 폭발.
        }

        // D좌표에 도달하지 않았다면 계속 이동
        if coordC.x < coordD.x
        {
            coordC.x += vector2.x * projectileSpped
            coordC.y += vector2.y * projectileSpped
        }
        else
        {
            // 도착. 폭발.
        }

        draw()
    }
    
    func draw(){
        // 화면을 그려줌
        //.....
    }
}

let testGame = VectorTranslation()
let engine = GameEngine(gameClass: testGame)
engine.run()
```

