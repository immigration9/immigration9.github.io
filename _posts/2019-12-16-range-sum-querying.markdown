---
layout: post
title: 'Range Sum Querying: Dynamic Problem'
date: 2019-12-16 23:00:00 +09:00
categories: 'algorithms'
published: true
---

Reference: [Range Sum Querying - Prevent Repeated Work with Chaching](https://youtu.be/ZMOFmHBVEcg)

## Range Sum Querying

캐싱을 통해 범위 동안의 Sum 값을 구한다.

이 방법이 가진 가장 큰 장점은, 캐싱 작업을 통해 한 번 진행하였던 계산은 이후에 다시 진행하지 않아도 된다는 점에 있다.

간단한 코드로 살펴보자.

```javascript
function solution(arr) {
  let result = new Array(arr.length + 1).fill(0);

  for (let i = 1; i < result.length; i++) {
    result[i] = arr[i - 1] + result[i - 1];
  }

  // rest of the code
}
```

기본적으로 식은 `cache[i] = cache[i - 1] + array[i - 1]`을 계산한다고 생각하면 된다.

이 방식으로 계산을 하면, 이후부터는 기존의 `array`가 아닌, `cache` 배열만으로 계산을 진행할 수 있다.

예를들어, 아래와 같이 계산한다고 가정해보자.

```javascript
function solution(arr, start, end) {
  let result = new Array(arr.length + 1).fill(0);

  for (let i = 1; i < result.length; i++) {
    result[i] = arr[i - 1] + result[i - 1];
  }

  return result[end] - result[start];
}
```
