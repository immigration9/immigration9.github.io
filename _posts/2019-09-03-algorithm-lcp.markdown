---
layout: post
title: 'LeetCode 문제풀이 - 최장 공통 접두사'
date: 2019-09-03 14:05:00 +09:00
categories: 'javascript,algorithms'
published: false
---

## 문제

LeetCode

문자 배열을 입력 받았을 때, 그 중에서 가장 길게 공통되는 접두사를 찾는 문제.
공통되는 접두사가 없을 경우, "" 를 반환한다.

```
["flower","flow","flight"] 를 받으면, "fl"을 반환한다.
```

## 풀이 1번

Horizontal 풀이법. LCP(S1... Sn)을 두개씩 연이어서 비교.
`LCP(S1, LCP(S2, LCP(S3, ...)))`
여기서 나온 결과값 도출

```javascript
var longestCommonPrefix = function(strs) {
  if (strs.length === 0) return '';

  let prefix = strs[0];

  for (let i = 1; i < strs.length; i++) {
    while (strs[i].indexOf(prefix) !== 0) {
      prefix = prefix.substring(0, prefix.length - 1);

      if (prefix.length === 0) return '';
    }
  }

  return prefix;
};
```

```javascript
/**
 * @param {string[]} strs
 * @return {string}
 */
var longestCommonPrefix = function(strs) {
  if (!strs || strs.length === 0) return '';
  return findCommonPrefix(strs, 0, strs.length - 1);
};

function findCommonPrefix(strs, l, r) {
  if (l === r) {
    return strs[l];
  } else {
    let mid = parseInt((l + r) / 2);
    /**
     * Divide
     */
    let lcpLeft = findCommonPrefix(strs, l, mid);
    let lcpRight = findCommonPrefix(strs, mid + 1, r);

    /**
     * Conquer
     */
    return commonPrefix(lcpLeft, lcpRight);
  }
}

function commonPrefix(left, right) {
  let min = Math.min(left.length, right.length);
  for (let i = 0; i < min; i++) {
    if (left.charAt(i) !== right.charAt(i)) {
      return left.substring(0, i);
    }
  }
  return left.substring(0, min);
}
```
