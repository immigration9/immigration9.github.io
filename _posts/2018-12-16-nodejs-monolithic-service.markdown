---
layout: post
title:  "NodeJS 공부하며 궁금했던 점들 정리"
date:   2018-12-16 14:00:00 +09:00
categories: "nodejs"
published: false
---

## Reference
[Node.js 마이크로 서비스 코딩공작소][reference-01]


## Request값에 body가 들어올 경우 (POST, PUT)
비동기 형식으로 동작하기 때문에, body가 전송될 경우, `data`와 `end`로 나눠서 작업하도록 한다.

```javascript
let server = http.createServer((req, res) => {

  /** 상략 **/
  let body = "";

  req.on('data', (data) => {
    body += data;
  });

  req.on('end', () => {
    let params;
    /**
      * Request의 header값이 json일 경우, 해당 방식으로 파싱을 진행
      * 이외에는 querystring의 파싱을 사용 
      * (여기서는 보여주기 위함으로 querystring을 코드 중간에 집어넣었다)
      */
    if (req.headers['content-type'] === "application/json") {
      params = JSON.parse(body);
    } else {
      params = require('querystring').parse(body);
    }
    
    cb(res, method, pathname, params);
  });
}).listen(8000);
```

## Event Loop



## process.nextTick()








[reference-01]:https://book.naver.com/bookdb/book_detail.nhn?bid=13347358