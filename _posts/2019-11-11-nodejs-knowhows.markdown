---
layout: post
title: 'NodeJS: Knowhows'
date: 2019-11-11 16:00:00 +09:00
categories: 'nodejs'
published: true
---

## Post로 hidden data보내기

```html
<form action="/cart" method="post">
  <button class="btn" type="submit">Add to Cart</button>
  <input type="hidden" name="productId" value="<%= product.id %>" />
</form>
```

위와 같은 형식으로 구성하면, 해당 페이지에서 POST request를 보낼 때 숨겨져있는 value `productId`를 추가해서 보낼 수 있다.

## 순수 MongoDB로 MVC 구성하기

ODM을 안쓰고 Mongo를 쓸 일이 있을까 싶지만, 항상 Base를 아는 것이 중요하니 남긴다.

```javascript
const mongodb = require('mongodb');
const MongoClient = mongodb.MongoClient;

let _db;

const mongoConnect = callback => {
  MongoClient.connect('mongodb://localhost:27017/shop')
    .then(client => {
      console.log('Connected!');
      _db = client.db();
      callback();
    })
    .catch(err => {
      console.log(err);
      throw err;
    });
};

const getDb = () => {
  if (_db) {
    return _db;
  }
  throw 'No database found!';
};

exports.mongoConnect = mongoConnect;
exports.getDb = getDb;
```

이 방법을 통해, 서버 시작시 MongoDB와의 Connection을 시작하고, `getDb`를 통해 이후 사용시 새로운 Connection을 만들지 않고 사용.
Connection Pool 관리는 이후에 다룬다.

### Save Procedure

```javascript
class Product {
  /** Constructor **/

  save() {
    const db = getDb();
    let dbOp;
    if (this._id) {
      // Update the product
      dbOp = db
        .collection('products')
        .updateOne({ _id: this._id }, { $set: this });
    } else {
      dbOp = db.collection('products').insertOne(this);
    }
    return dbOp
      .then(result => {
        console.log(result);
      })
      .catch(err => {
        console.log(err);
      });
  }
}
```

현재 id가 배정되어 있으면 `update`를 진행 / 없으면 `insert`를 진행
