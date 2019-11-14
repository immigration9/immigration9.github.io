---
layout: post
title: 'LeetCode 문제풀이 - 가장 큰 Container 크기 찾기'
date: 2019-09-03 14:05:00 +09:00
categories: 'javascript,algorithms'
published: false
---

```javascript
/**
 * @param {number[]} height
 * @return {number}
 */
var maxArea = function(height) {
  let largestContainer = 0;
  let l = 0;
  let r = height.length - 1;

  while (l < r) {
    largestContainer = Math.max(
      largestContainer,
      (r - l) * Math.min(height[r], height[l])
    );

    if (height[l] > height[r]) {
      r--;
    } else {
      l++;
    }
  }
  return largestContainer;
};
```

- 이 문제의 핵심은, 항상 더 작은 선을 기준으로 높이가 정해지기 때문에, 높은 선은 놔두고, 더 작은 선을 움직인다는 점에 있다.
- 이 방법을 차용할 경우, 기존의 Brute Force 대비하여 (for 문을 두 번 사용) 약 10배 정도 속도가 빨라진다.
