---
layout: post
title: 'NodeJS Filestream'
date: 2018-11-11 16:41:00 +0900
categories: 'nodejs'
---

### Reference

[(재인용) Node.js Stream 당신이 알아야할 모든 것][reference-01]

[(원글) Node.js Streams: Everything you need to know][reference-02]

[Node js Streams Tutorial: Filestream. Pipes][reference-03]

[Creating duplex streams with Node.js][reference-04]

## 스트림이란?

스트림: 배열이나 문자열 같은 데이터 컬랙션

- 많은 양의 데이터를 한 번에 한 청크(chunk)씩 가져올 수 있다.
- 리눅스 piping과 같이 코드 조합도 가능하게 해준다.
- 다양한 모듈들이 스트림 인터페이스의 구현체이다. 예를들어, HTTP Response는 클라이언트 사이드에서는 읽기 스트림인 반면, 서버 사이드에서는 쓰기 스트림이다.

## 일반 Read File과 File Stream 비교

비교를 위해 우선 400mb 정도 되는 파일을 Write Stream을 활용하여 생성한다.
{% highlight javascript %}
const fs = require('fs');
const file = fs.createWriteStream('./big.file');

for(let i=0; i<= 1e6; i++) {
file.write('Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.\n');
}

file.end();
{% endhighlight %}

`fs` 모듈의 Write Stream 을 토대로 약 400mb 정도 크기의 파일을 생성하였다.

웹서버를 통해 아래 파일을 읽는 작업을 해보자.

```javascript
const fs = require('fs');
const server = require('http').createServer();

server.on('request', (req, res) => {
  fs.readFile('./big.file', (err, data) => {
    if (err) throw err;

    res.end(data);
  });
});
server.listen(8000);
```

실행 직후 노드는 9168K (9mb) 정도의 사용량을 보였다.
이제 서버에 접속해보면 노드 프로세스의 사용량이 400mb를 넘게 되는 것을 확인할 수 있다.
이는 readFile의 경우 파일 내용을 전부 메모리에 올려놓기 때문이다.

이 문제를 해결하기 위해 Read Stream을 이용하여 HTTP Response에 Stream을 내려주도록 한다

```javascript
const fs = require('fs');
const server = require('http').createServer();

server.on('request', (req, res) => {
  const src = fs.createReadStream('./big.file');
  src.pipe(res);
});
server.listen(8000);
```

이 방식대로 진행하면 메모리에 모든 내용을 버퍼로 잡지 않기 때문에 훨씬적은 메모리를 기반으로 데이터를 전달할 수 있다.

## 다양한 스트림 타입

1. Read Stream: ex. `fs.createReadStream`
2. Write Stream: ex. `fs.createWriteStream`
3. Duplex: 읽기 쓰기가 모두 가능함. ex. TCP Socket.
4. Transform: 기본적으로 Duplex 스트림이며, 데이터를 읽거나 기록시 수정/변환될 수 있는 데이터를 나타낸다. ex. `zlib.createGzip` gzip을 이용해 데이터 압축

- 모든 스트림은 `EventEmitter`의 인스턴스이며 데이터 읽기/쓰기시 사용할 이벤트 방출(emit)하는 구조.
- 스트림 데이터 사용을 위해 `pipe` 메소드와 함께 사용시 더 편리하다.

### Read Stream 실제로 사용해보기

`createReadStream`을 실제로 사용하는 예제를 작성해보자.

```javascript
const fs = require('fs');
const src = fs.createReadStream('./test1');
src.on('readable', () => {
  let data;

  while ((data = src.read())) {
    console.log(data.toString());
  }
});

src.on('end', () => {
  console.log('File has ended');
});
```

`readable` 이벤트는 stream으로부터 읽을 수 있는 데이터가 있을 때 실행된다. 해당 이벤트는 새로운 데이터가 존재하거나, 스트림의 마지막에 도착하였음을 나타낸다. 전자의 경우, `read()`가 활용 가능한 데이터를 반환할 것이고, 후자의 경우 `null`을 반환할 것이다.

- `read`의 EOF는 null, 혹은 `destroy()` 호출을 통해 시행할 수 있다.

### Pipe 이벤트

`pipe` 이벤트는 `stream.pipe()` 메소드가 readable stream에 호출되었을 때 방출되며, 해당 writable을 목적지에 추가한다.

```javascript
const fs = require('fs');
const assert = require('assert');
const writer = fs.createWriteStream(__dirname + '/test2');
const reader = fs.createReadStream(__dirname + '/test1');

writer.on('pipe', src => {
  console.log('Asserting if it equals');
  assert.equal(src, reader);
});

reader.pipe(writer);
```

위 과정에서 `createReadStream`은 `test1` 파일을 읽은 뒤 pipe로 해당 내용을 전달한다. `pipe` 이벤트는 해당 writable을 목적지에 전달한다.
`createWriteStream`의 `test2` 파일은 `pipe` 이벤트를 대기하고, 이벤트가 접수되면 데이터를 받은 후 `assert` 함수를 진행 한 후, 완료되면 정상적으로 파일에 데이터를 추가한다.

### 스트림 이벤트

읽고 쓰는 작업 이외에도 `pipe` 메소드는 자동으로 몇 가지 작업들을 관리한다. (에러 처리 / EOF 처리 / 스트림 우선순위 등))
하지만 아래와 같이 직접 이벤트를 트리거 하여 사용할 수도 있다.

- Readable Streams: data / end / error / close / readable
  - `data` 이벤트: 스트림이 소비자에게 데이터 청크 전송시 발생
  - `end` 이벤트: 더 이상 전송할 데이터 없을시 발생
- Writable Streams: drain / finish / error / close / pipe-unpipe
  - `drain` 이벤트: 쓰기 가능한 스트림이 더 많은 데이터를 수신할 수 있다는 신호
  - `finish` 이벤트: 모든 데이터가 시스템으로 플러시 될 때 생성

### Readable Stream 일시정지 모드 / 흐름 모드

모든 readable stream은 일시정지 모드에서 시작하지만, 필요에 따라 흐름모드(flowing mode)로 변경되거나 일시 정지 모드(pause mode)로 되돌아갈 수 있음. (자동 스위칭도 발생)

Readable stream이 일시정지 모드일 때 스트림을 읽기 위해 `read()` 메소드를 호출할 수 있다. 반대로 flowing mode일 경우에는 데이터가 연속적으로 흐르고 있으며 대기를 해야한다.

- 주의: flowing mode시에, 데이터 수신자가 없을 경우 데이터는 소실된다. 그렇기 때문에 flowing mode에서 readable stream은 `data event handler`를 필요로 한다. `data event handler`의 존재는 pause -> flowing mode 변경이며, 제거하는 것은 다시 flowing -> pause mode로의 변경이다. `pipe` 사용시에는 자동으로 관리되기 때문에 신경쓸 필요 없다.

## 스트림 구현

### 쓰기 스트림 만들기

```javascript
const { Writable } = require('stream');

class myWritableStream extends Writable {
  _write(chunk, encoding, cb) {
    console.log(chunk.toString());
    cb();
  }
}

const outStream = new myWritableStream();

process.stdin.pipe(outStream);
```

스트림 모듈의 `Writable` 클래스를 이용하여 구현을 진행하였다. (ES6 형식으로 작성할 경우 `_write` 메소드를 사용하자)

### 읽기 스트림 만들기

```javascript
const { Readable } = require('stream');

class myReadableStream extends Readable {
  constructor() {
    super();

    this.currentCharCode = 65;
  }

  _read(size) {
    this.push(String.fromCharCode(this.currentCharCode++));
    if (this.currentCharCode > 90) {
      this.destroy();
    }
  }
}

const inStream = new myReadableStream();

inStream.pipe(process.stdout);
```

읽기 스트림을 `_read` 메소드에서 호출시 일부 데이터를 Queue에 푸시하게 된다. 마지막으로 90이 넘었을 때 (알파벳 Z) `this.destroy()` 를 호출함으로써 `_read`를 종료시킨다.

### Duplex + Transform

TODO

[reference-01]: https://github.com/FEDevelopers/tech.description/wiki/Node.js-Stream-%EB%8B%B9%EC%8B%A0%EC%9D%B4-%EC%95%8C%EC%95%84%EC%95%BC%ED%95%A0-%EB%AA%A8%EB%93%A0-%EA%B2%83
[reference-02]: https://medium.freecodecamp.org/node-js-streams-everything-you-need-to-know-c9141306be93
[reference-03]: https://www.guru99.com/node-js-streams-filestream-pipes.html
[reference-04]: http://codewinds.com/blog/2013-08-31-nodejs-duplex-streams.html
