---
title: 블록스
titleEn: The Blocks
author: BabyK
date: 2025-07-25
category: GameDev
layout: post
tags: [GameDev]
published: true
---

#### _This is a personal project developed for learning purposes only, inspired by retro-style block puzzle games._ ####

#### 키조작 (INPUT - PC ONLY) 

<table style="width: 300px; height: 200px;font-size: 0.9em;">
  <tr>
    <td>UP</td>
    <td>오른쪽 회전</td>
  </tr>
  <tr>
    <td>LEFT, RIGHT</td>
    <td>좌우 이동</td>
  </tr>
  <tr>
    <td>DOWN</td>
    <td>소프트 드롭</td>
  </tr>
  <tr>
    <td>SPACE BAR</td>
    <td>하드 드롭</td>
  </tr>

  <tr>
    <td>Z</td>
    <td>왼쪽 회전</td>
  </tr>
  <tr>
    <td>X</td>
    <td>오른쪽 회전</td>
  </tr>
</table>

<div style="display: flex; gap: 20px;">
    <a href="/source/blocks.html" target="_blank">
    <button>PLAY</button>
    </a>
    <a>
    <button onclick="showHtmlInNewTab()">Source코드</button>
    </a>
</div>


<script>
    function showHtmlInNewTab() {
      fetch('/source/blocks.html')
        .then(res => res.text())
        .then(data => {
          const encodedHtml = data
            .replace(/</g, '&lt;')  
            .replace(/>/g, '&gt;');

          const newWindow = window.open();
          newWindow.document.write(`<pre style="white-space:pre-wrap; word-wrap:break-word;">${encodedHtml}</pre>`);
          newWindow.document.close();
        });
    }
</script>


<br>
전세계 사람들이 즐기는 전설적인 그 게임을 만들어보자.  
이 게임은 저작권이 걸려있어 관련 내용을 오리지널 그대로 포스팅 하는경우 운이 좋지 않다면 홈페이지가 폐쇄되는 상황을 초래할 수도 있다.  
업계에 첫발을 내딛기 위해 제작했던 포트폴리오가 도스용으로 제작한 바로 이 게임이라 개인적으로 애착을 갖고있고 
충돌감지, 키조작, 좌표 및 화면출력과 같은 모든 게임의 기본이 되는 기능들이 포함하고 있어 게임코드의 구조를 익히기에도 좋다.  
 
게임 업데이트 로직은 [requestAnimationFrame][2]{:target="_blank"} 메소드를 사용한다.  
자바스크립트는 기본적으로 **싱글스레드 + 이벤트 루프기반 비동기처리** 방식이라 JAVA 나 C 와 똑같이 while 문을 사용하면 사용자 입력이나 화면 처리가 불가능해지고 [setInterval][3]{:target="_blank"} 함수를 사용하면 딜레이가 생길 수도 있어 정확한 타이머 동작이 요구되는 게임에는 권장되지 않는다. ( setInterval 은 이벤트 루프상에서 우선순위가 낮아 처리가 뒤쪽으로 밀리게 될 가능성이 있다)

### 1. 게임로직
```javascript
// 게임루프 (업데이트 + 화면 렌더링)
function runGameLoop() {
    update(performance.now());
    renderGame();
    requestAnimationId = requestAnimationFrame(runGameLoop);
}

// 업데이트
function update(currentTime) {
    deltaTime += (currentTime - lastTime) / 1000;
    if (deltaTime > 1) {
        dropShape();
        deltaTime = 0;
    }
    lastTime = currentTime;
}

// 화면 렌더링
function renderGame() {
    drawGameScreen();
    drawCurrentShape();
}
```
<br>

### 2. 좌표계산
C 언어에서 2차원 배열은 메모리상에 연속적으로 할당되어 포인터연산으로 순차적으로 접근할 수 있는 반면 JAVA 나 자바스크립트의 경우 비연속 구조로 배열 안에 포인터를 참조하여 접근하기 때문에 그보다는 효율성이 떨어진다.
화면배열은 1차원으로 생성하고 접근시에 X, Y 좌표를 계산하여 조작하는 것이 성능상 조금이라도 더 유리하다.
현재 도형의 1차원좌표가 10 이라면 2차원 좌표는 (0,1) 이다. 도형의 위치가 배열의 시작지점이 되고 도형의 size 값을 사용해서 화면상의 도형배열이 위치한 범위를 탐색한다. 


```javascript
// 도형이 위치한 게임화면의 1차원 좌표 -> 2차원 좌표 변환
let xStart = currentShape.location % 10;
let yStart = Math.floor(currentShape.location / 10);

// 도형이 위치한 게임화면 내의 블록들을 하나하나 체크
// size : 도형별 크기. (2X2, 3X3, 4X4)
for (let i = 0; i < size; i++) {
    for (let j = 0; j < size; j++) {
        // 2차원 좌표 -> 1차원 좌표 변환
        let currentLocation = (yStart + i) * 10 + (xStart + j);
        // ...
    }
}
```

<br>

### 3. 도형생성
7개의 도형을 한꺼번에 생성하여 큐에 넣고 셔플을 돌려 순서를 섞는다.  
놀랍게도 각 도형은 순서만 달라질뿐 각 사이클당 한 번씩 정확하게 스폰되니 혹시라도 I 쉐입이 잘 안나오는 것 같이 느껴진다면 그것은 단지 기분탓.  

```javascript
// 도형 선언
const SHAPES = { 
                I: {
                    type: 'I',
                    matrix: [
                        0, 0, 0, 0,
                        1, 1, 1, 1,
                        0, 0, 0, 0,
                        0, 0, 0, 0
                    ],
                    size: 4,
                    color: colors[0]
                },
                // ...
}

// 도형생성
function generateShape() {
    if (!isInputValid) {
        return;
    }

    if (shapeQueue.length <= 1) {       // 큐가 비었다면
        for (const key in SHAPES) {     // 모든 유형의 도형을 하나씩 생성해 큐에 복사
            const shape = SHAPES[key];
            shapeQueue.push({
                type: shape.type,
                matrix: [...shape.matrix],
                size: shape.size,
                color: shape.color,
                rotationNum: 0,
                location:   // 게임화면상 스폰될 위치 좌표
                shape.type === 'I' ? 3 : 
                shape.type === 'O' ? 4 :
                shape.type === 'T' ? 4 :
                shape.type === 'S' ? 4 :
                shape.type === 'Z' ? 4 :
                shape.type === 'J' ? 4 :
                shape.type === 'L' ? 4 : 4
            });
        }
        // 큐에 셔플을 돌려 순서를 섞는다.
        for (let i = shapeQueue.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [shapeQueue[i], shapeQueue[j]] = [shapeQueue[j], shapeQueue[i]];
        }
    }
    
    // 도형의 스폰위치에 축적된 블록이 존재하는가. 게임오버 체크
    let collisionCheckShape = shapeQueue.shift();
    const tempShape = {collisionCheckShape, matrix : [collisionCheckShape.matrix], location: collisionCheckShape.location+10};
    const resultCollision = checkCollision(tempShape);
    if (resultCollision === CollisionType.BLOCK)
    {   // 충돌감지. 게임오버
        handleGameOver();
    }
    else
    {   // 충돌없음. 도형 스폰. 미리보기창 업데이트
        currentShape = collisionCheckShape;
        drawNextShape();   
    }
}


// 낙하중인 도형 그리기
function drawCurrentShape() {
    let size = currentShape.size;

    for(let i = 0; i < size; i++) {
        for (let j = 0; j < size; j++) {
            if(currentShape.matrix[i*size + j] === 1) { // 도형배열 내에 블록이 있는 좌표만 필터
                let currentLocation = currentShape.location + j + i * 10;   // 도형이 위치한 게임화면내의 좌표 계산
                let x = currentLocation % 10;               // x 좌표
                let y = Math.floor(currentLocation / 10);   // y 좌표
                let rectColor = currentShape.matrix[(i*size) + j] === 1 ? currentShape.color : defaultRectColor;
                drawRectangle(x, y, rectColor); //블록 그리기
            }
        }
    }
}

// 사각형 블록 그리기 함수
function drawRectangle(x, y, color) {
    ctx.beginPath();
    ctx.fillStyle = color;
    ctx.fillRect(x * rectSize, y * rectSize, rectSize, rectSize);
    ctx.strokeStyle = (color === defaultRectColor || color === hiddenRectColor) ? rectBorderColor2 : rectBorderColor1;
    ctx.strokeRect(x * rectSize, y * rectSize, rectSize, rectSize);
}
```

<br>

### 4. 도형회전 (Shape Rotation)
도형은 총 7개, 각각 회전된 모양을 하드코딩해서 사용하거나 회전키를 누를때마다 계산하는 방법이 있다.  
이번에는 계산하는 방식을 사용하자.  


```javascript
// 도형 선언. 1차원배열
const SHAPES = { 
    I: {
        type: 'I',
        matrix: [
            0, 0, 0, 0,
            1, 1, 1, 1,
            0, 0, 0, 0,
            0, 0, 0, 0
        ],
        size: 4,
        color: colors[0]
    },
    // ...
};

// 도형 회전 
function tryRotation(direction) {
    let size = currentShape.size;
    let newMatrix = new Array(size * size);
    
    // 4회전
    // >> 회전 후 위치 => [행번호 -> 행번호의 역순으로 열에 대입, 열번호 -> 행번호]
    // ex 우측회전일때 (1,1) -> (1,3)
    for (let i = 0; i < size; i++) {
        for (let j = 0; j < size; j++) {
            let value = currentShape.matrix[(i * size) + j];
            if (direction === 'Right')
            {
                newMatrix[j * size + (size - 1 - i)] = value;
            }
            else if (direction === 'Left')
            {
                newMatrix[(size - 1 - j) * size + i] = value;
            }
        }
    }
    let newRotationNum = direction === 'Right'
                ? (currentShape.rotationNum + 1) % 4
                : (currentShape.rotationNum + 3) % 4;
    const shape_rotation = {...currentShape, matrix: [...newMatrix], rotationNum: newRotationNum};
    //  충돌감지
    const resultCollision = checkCollision(shape_rotation);
    if (resultCollision === CollisionType.OK) {
        currentShape = {...shape_rotation};
    }
}
```
<br>

### 5. 충돌감지 (Collision Detection)
현재 조작중인 도형이 위치하고 있는 영역안에 벽, 누적된 블록이 있는지를 순회하며 검사한다.  
충돌감지는 모든 게임에서 핵심적인 영역으로 3D게임뿐만 아니라 왠만한 2D게임의 경우에도 픽셀과 픽셀이 맞닿는 영역을 검사하는 루프문은 상당한 리소스를 잡아먹을 수 있어 상용게임의 경우 정확성과 동시에 최적화에 공을 들여야 하는 부분이다.  

```javascript

const CollisionType = {
    OK: 0,      // no collision
    WALL: 1,    // left right screen edge
    BLOCK: 2,   // stacked block
};

 // 충돌감지
function checkCollision(shape) {
    let xStart = shape.location % 10;
    let yStart = Math.floor(shape.location / 10);

    for (let i = 0; i < shape.size; i++) {
        for (let j = 0; j < shape.size; j++) {
            let shapeLocation = i * shape.size + j;
            if ( shape.matrix[shapeLocation] === 1 ) {  // 블럭이 있는 메트릭스만 검사
                const screenLocation = (yStart + i) * 10 + (xStart + j);    // 화면좌표 계산
                
                // 1. 탑바텀 화면아웃 체크 (BLOCK)
                const isOutOfTopBounds = (shape.location < 0);
                const isOutOfBottomBounds = (screenLocation >= 220);
                    if (isOutOfBottomBounds || isOutOfTopBounds) { return CollisionType.BLOCK; };
                // 2. 블록충돌 검사 (BLOCK)
                const isCollision = (gameScreenArray[screenLocation]?.value === 1);
                if (isCollision) {return CollisionType.BLOCK;};
                // 3. 회전충돌 (WALL)
                const isRotating = currentShape.rotationNum !== shape.rotationNum;
                if (isRotating) {
                    let x = xStart + j;
                    let y = yStart + i;

                    // RIGHT, LEFT, BOTTOM
                    if (x > 9 || x < 0 || y >= 22 || isCollision) {   
                        return CollisionType.WALL;
                    }
                }
                // 4. 좌우 이동 벽 충돌 (WALL)
                const isLeftWallCollision = (currentShape.location - shape.location === 1 && screenLocation % 10 === 9);
                const isRightWallCollision = (currentShape.location - shape.location === -1 && screenLocation % 10 === 0); 
                if(isLeftWallCollision || isRightWallCollision) { return CollisionType.WALL; };
            }
        }
    }
    return CollisionType.OK;
}

```
<br>

### 6. 도형쌓기 (Stacking)
충돌이 감지되면 (바텀 혹은 블록) 화면배열위에 현재 도형이 위치한 지점부터 블록이 그려진 영역들만 필터링하여 복사한다. 이후 화면영역을 순회하며 블록이 꽉 채워진 행들을 찾아 삭제한다.

```javascript
function lockCurrentShape() {
    let matrix = currentShape.matrix;
    let size = currentShape.size;
    let xStart = currentShape.location % 10;
    let yStart = Math.floor(currentShape.location / 10);

     화면배열에 넣는다.  
    for (let i = 0; i < size; i++) {
        for (let j = 0; j < size; j++) {
            // 조작중인 도형이 위치한 배열에서 블록부분 ( value === 1) 을 찾는다
            if (matrix[i * size + j] === 1) {
                // 도형의 블록 위치를 화면배열상의 위치로 변환
                let currentLocation = (yStart + i) * 10 + (xStart + j);
                // 화면배열에 블록 채우기
                gameScreenArray[currentLocation].value = 1;
                gameScreenArray[currentLocation].color = currentShape.color;
            }
        }
    }
    clearLines();   // 채워진 라인 삭제
}

// 채워진 행 삭제
function clearLines() {
    if (!isInputValid) {
        return;
    }
    let row = 21;   // 총 22라인 (히든스페이스 2 라인 포함)
    while(row > -1) {
        // 현재 라인이 가득 찼는가.
        let isFull = true;
        for (let i = 0; i < 10; i ++) {
            if (gameScreenArray[row * 10 + i].value === 0) {
                isFull = false;
                break;
            }
        }
        if (isFull) 
        {   // 삭제할 라인의 윗 라인을 한칸씩 내려 덮어씌운다.
            for (let i = row; i > 0; i--) {
                for (let j = 0; j < 10; j++) {
                    gameScreenArray[i * 10 + j].value = gameScreenArray[((i - 1) * 10) + j].value;
                    gameScreenArray[i * 10 + j].color = gameScreenArray[((i - 1) * 10) + j].color;
                }
            }
        } 
        else
        {
            row--;
        }
    }
}

```
<br>


#### 개선 가능한 부분
- 월킥 (회전시 벽이나 블록을 차면서 이동)
- 네트워크 플레이
- 게임점수누적
- 랭킹보드
- 효과음
- 난이도 조절 (드롭속도, 도형스폰 확률)
- 화면디자인

<br>

다음 포스트에서는 지금의 싱글 게임을 멀티플레이가 가능한 네트워크 게임으로 바꿔보자.  

<br>

[1]: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API
[2]: https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame
[3]: https://developer.mozilla.org/en-US/docs/Web/API/Window/setInterval