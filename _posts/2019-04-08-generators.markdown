---
layout: post
title:  "ES6: Generators prior to understanding Sagas"
date:   2019-04-08 01:00:00 +09:00
categories: "javascript"
published: true
---

## for ... of loop

```javascript
const colors = ['red', 'green', 'blue'];

for (let color of colors) {
  console.log(color);
}

const numbers = [1, 2, 3, 4];
let total = 0;
for (let number of numbers) {
  total += number;
}
console.log(total);
```

## Generators
여러번 출입할 수 있는 함수를 뜻한다. 과연 이게 의미하는 바가 무엇일까?

```javascript
function* numbers() {
  yield;
}
const gen = numbers();
gen.next(); // {"done":false} 
gen.next(); // {"done":true}
```

```javascript
function* shopping() {
  // stuff on the sidewalk

  // walking down the sidewalk

  // go into the store with cash
  const stuffFromStore = yield 'cash';

  // walking back home
  return stuffFromStore;
}

// stuff in the store
const gen = shopping();
gen.next(); // leaving our house
gen.next('groceries') ; // leaving the store with groceries
```


```javascript
const testingTeam = {
  lead: 'Amanda',
  tester: 'Bill'
}

const engineeringTeam = {
  testingTeam,
  size: 3,
  department: 'Engineering',
  lead: 'Jill',
  manager: 'Alex',
  engineer: 'Dave'
}

function* TeamIterator(team) {
  yield team.lead;
  yield team.manager;
  yield team.engineer;
}

function* TestingTeamIterator(team) {
  yield team.lead;
  yield team.tester;
}

const names = [];
for (let name of Teamiterator(engineeringTeam)) {
  names.push(name);
}
names;
```