---
layout: post
title:  "Mediator Pattern (중재자 패턴)"
date:   2018-11-24 18:00
categories: "javascript"
---
## References
[자바스크립트 코딩 기법과 핵심 패턴][reference-01]

## Basics
객체간의 통신은 하나의 객체가 다른 객체에 직접적으로 영향을 주지 않는 방향이 안전하며 유지보수 용이하다. 객체가 서로에 대한 정보를 아는 상태로 통신하게 되면 결합도가 상승하고, 이는 바람직하지 않다. 
중재자 패턴은 결합도를 낮추는데에 포커싱을 한다. 각각의 객체들은 직접 통신이 아닌, 중재자 객체를 통해 서로 영향을 끼친다. 객체의 상태가 변경되면 중재자에게 알리고, 중재자는 변경 사항을 알고 있어야하는 다른 객체들에게 알린다.

## 중재자 패턴 예제
2명의 플레이어가 플레이하는 보드게임을 구성한다. 1번 플레이어는 숫자 1을, 2번 플레이어는 숫자 2를 눌러서 플레이한다. 별도의 점수판이 점수를 보관한다.
중재자는 모든 객체에 대해 알고 있으며, 이벤트 처리와 플레이어 차례를 알려준다. 전달된 점수는 점수판에 표시된다.

* 아래 코드는 모두 ES6로 작성된 뒤, Webpack으로 번들링되었다. 
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8" />
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <title>Mediator Pattern</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<body>
  <div id="results"></div>
</body>
<script src="mediator.js"></script> <!-- Bundle filename. DOM이 인식되기 위해선 Script를 가장 아래에 둬야한다.-->
</html>
```

```javascript
import mediator from './Mediator';
class Player {
  constructor(name) {
    this.points = 0;
    this.name = name;
  }
  play = () => {
    this.points += 1;
    mediator.played();
  }
}
export default Player;
```

Scoreboard와 Mediator는 한 개씩 존재하기 때문에 Singleton으로 구성되어 있다. 다용도로 사용될 가능성이 있을 경우, index값을 받아서 별도로 생성을 해주거나, `unsubscribe`와 같은 방법으로 처리를 해줘야 한다.
```javascript
class Scoreboard {
  constructor() {
    this.element = document.getElementById('results');
  }
  update = (score) => {
    let msg = '';
    for (let i in score) {
      if (score.hasOwnProperty(i)) {
        msg += '<p><strong>' + i + '</strong>: ';
        msg += score[i];
        msg += '</p>';
      }
    }
    this.element.innerHTML = msg;
  }
}
let scoreboard = new Scoreboard();
export default scoreboard;
```

```javascript
import scoreboard from './Scoreboard';
class Mediator {
  constructor() {
    this.players = [];

  }
  subscribe = (player) => {
    this.players.push(player);
  }
  played = () => {
    let players = this.players;
    let score = {};
    players.map((pl) => {
      score[pl.name] = pl.points;
    })
    scoreboard.update(score);
  }
  keypress = (e) => {
    e = e || window.event;
    const base = 49;
    let key = e.which - base; // 49 === Number(1);

    if (key < this.players.length) {
      let player = this.players[key];
      player.play();
    } else {
      alert("Key is not assigned");
    }
  }
}
let mediator = new Mediator();
export default mediator;
```

일시적으로 `window.onkeypress` 이벤트에 중재자 객체의 `keypress` 함수를 등록하였다. 아래의 코드에선 30초 타임아웃 이후에 값을 제거한다.
```javascript
import mediator from './Mediator';
import Player from './Player';

let player1 = new Player("immigration9");
let player2 = new Player("helloworld");
let player3 = new Player("homer");

mediator.subscribe(player1);
mediator.subscribe(player2);
mediator.subscribe(player3);
window.onkeypress = mediator.keypress;

setTimeout(() => {
  window.onkeypress = null;
  alert('Game finished');
}, 30000)
```

[reference-01]:https://book.naver.com/bookdb/book_detail.nhn?bid=6763510