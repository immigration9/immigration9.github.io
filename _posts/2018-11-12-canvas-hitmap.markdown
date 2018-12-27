---
layout: post
title:  "Canvas 그리기"
date:   2018-11-12 10:00:00 +0900
categories: "canvas"
---
## Reference
[HTML5 Canvas Clipping Region Tutorial][reference-01]

[HTML Canvas 2D Context][reference-02]




## CanvasRenderingContext2D.save() / restore() 사용
Canvas 2D API의 `CanvasRenderingContext2D.save()` 메소드는 현재 상태를 스택(Stack)에 넣는 것으로 전체 상태를 저장한다.

* 현 transformation matrix 값
* 현 clipping region 값 ([링크 참조][reference-01])
* 현 dash list 값 ([링크 참조][reference-02])
* `strokeStyle`, `fillStyle`, `lineWidth`, `font`, `textAlign` 등 속성들

`CanvasRenderingContext2D.restore()` 메소드는 drawing state 스택의 최상단으로부터 가장 최근에 저장된 canvas state 값을 복구한다. (없을 경우 아무일도 일어나지 않는다).

```javascript
var canvas = document.getElementById('test_canvas');
var ctx = canvas.getContext('2d');
// 1. Default State (Black)을 저장하였다.
ctx.save(); 

// 2. Red color를 저장한다
ctx.fillStyle = "red";
ctx.save();

// 3. Green color를 바로 사용한다.d
ctx.fillStyle = 'green';
ctx.fillRect(10, 10, 100, 100);

// 2. Stack 상단에 있는 Red를 가져온다.
ctx.restore(); 
ctx.fillRect(150, 75, 100, 100);

// 1. Stack에 남은 Black을 가져온다.
ctx.restore();
ctx.fillRect(300, 150, 100, 100);
```

위의 예제를 보면, `restore()`가 호출 된 이후에는 스택에서 저장된 state값이 제거되었음을 알 수 있다.

## Clipping
`CanvasRenderingContext2D.clip()` 메소드는 이후 들어오는 `path` 값을 현재 장소로 국한시킨다.


[reference-01]:https://www.html5canvastutorials.com/advanced/html5-canvas-clipping-region-tutorial/

[reference-02]:https://www.w3.org/TR/2dcontext/#canvasdrawingstyles