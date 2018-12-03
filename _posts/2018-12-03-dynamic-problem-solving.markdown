---
layout: post
title:  "다이내믹 프로그래밍 - 문자열 비교"
date:   2018-12-04 00:47:00 +09:00
categories: "algorithms"
---

## Reference
본 내용은 CS Dojo 채널의 'Using Dynamic Programming to Solve a Real-World Problem!' 강의를 공부하기 위해 정리한 것.
[Using Dynamic Programming to Solve a Real-World Problem!][reference-01]
[Example Github][reference-02]

## 개요
다이내믹 프로그래밍을 이용하여 문자열을 비교하는 로직을 구성하고자 한다.
예를들어, "ABCD"와 "ACDEB" 문자열을 가지고 있다면, 두 문자열을 우선적으로 비교한다.
비교를 통해 공통 분모를 찾아낸다. 이 경우, 공통분모는 "ACD"가 된다.
이 과정을 통해 얻어야하는 것은 두 가지다.
1. 공통분모
2. 두 문자열의 차이점

## Psuedo Code
```
1   function LCS(s1, s2, i1, i2)
2     if i1 == s1.length || i2 == s2.length
3       return ""
4     if s1[i1] == s2[i2]
5       return s1[i1] + LCS(s1, s2, i1 + 1, i2 + 1)
6     resultA = LCS(s1, s2, i1 + 1, i2)
7     resultB = LCS(s1, s2, i1, i2 + 1)
8     if resultA.length > resultB.length
9       return resultA
10    else
11      return resultB
```
LCS(Longest Common Sequence) 함수는 문자열 2개와 각각에 대한 인덱스 값(여기선 0)을 기준으로 시작한다.
2번째 줄에서 인덱스 값이 각각 문자열의 길이를 초과할 경우 함수를 종료한다.
4번째 줄에서 비교시 두 값이 같으면, 현재 비교 값과 인덱스 값을 증가한 LCS 함수의 재귀호출의 합을 반환한다.
만약, 둘이 같지 않다면, s1과 s2의 인덱스 값을 각각 하나씩 증가한 뒤 LCS 함수를 재귀호출한다.
여기서 나온 값 중 더 큰 값을 반환한다.

다 좋은데, 문제점이 하나 있다. 시간 복잡도가 너무 높다. O(2의 m + l승), 여기서 m, l은 각각 s1과 s2의 길이를 뜻한다. 정도의 시간 복잡도를 갖는다. (최악의 경우, 각 항목마다 2번씩의 연산을 진행하는데, s1과 s2만큼의 길이가 있기 때문에, O(2의 m + l승)이 된다.)

## Pseudo Code Refactored
이를 해결하기 위해, cache해줄 항목을 추가하면 아래와 같은 코드가 구성된다.
```
0   init memo[s1.length][s2.length]
1   function LCS(s1, s2, i1, i2, memo)
2     if i1 == s1.length || i2 == s2.length
3       return ""
4     if memo[i1][i2] != undefined
5       return memo[i1][i2]
6     if s1[i1] == s2[i2]
7       memo[i1][i2] = s1[i1] + LCS(s1, s2, i1 + 1, i2 + 1, memo)
8       return memo[i1][i2]
9     resultA = LCS(s1, s2, i1 + 1, i2)
10    resultB = LCS(s1, s2, i1, i2 + 1)
11    if resultA.length > resultB.length
12      return resultA
13    else
14      return resultB
```
0번째 줄에는 추가로 s1과 s2만큼의 길이를 배열의 크기로 갖는 이차원 배열 `memo`가 초기화 되어있다.
이제 값이 저장되는 경우, 바로 반환되는 것이 아니라, `memo`에 저장된 다음, 저장된 값을 반환한다.

이 경우, 한 번 수행한 값은 다시 재귀 연산하지 않고, `memo`값을 반환하기 때문에 O(m * l)의 시간 복잡도를 갖는다.
10개의 문자열 2개를 비교한다면, 최악의 경우에는 `memo`를 추가하지 않은 경우에는 2의 20승 만큼의 시간(1,048,576)이 걸리고, 추가한 경우에는 100 만큼의 시간이 소요된다. (최초(100)에 비해 0.009 정도의 시간 밖에 사용되지 않는다)

## Real Code Example
```javascript
function LCS(s1, s2) {
  let memo = [...Array(s1.length)].map(e => Array(s2.length));
  return helper(s1, s2, 0, 0, memo);
}

function helper(s1, s2, i1, i2, memo) {
  if (i1 === s1.length || i2 === s2.length) {
    return '';
  }
  if (typeof memo[i1][i2] !== 'undefined') {
    return memo[i1][i2];
  }
  if (s1[i1] === s2[i2]) {
    memo[i1][i2] = s1[i1] + helper(s1, s2, i1 + 1, i2 + 1, memo);
    return memo[i1][i2];
  }
  let result;
  let resultA = helper(s1, s2, i1 + 1, i2, memo);
  let resultB = helper(s1, s2, i1, i2 + 1, memo);
  if (resultA.length > resultB.length) {
    result = resultA;
  } else {
    result = resultB;
  }
  memo[i1][i2] = result;
  return memo[i1][i2];
}
```

[reference-01][https://www.youtube.com/watch?v=4SP_AY7GGxw]
[reference-02][https://github.com/ykdojo/text_difference_finder/blob/master/text_difference.js]