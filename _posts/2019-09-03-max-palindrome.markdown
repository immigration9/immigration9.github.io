---
layout: post
title: 'LeetCode 문제풀이 - 가장 긴 Palindrome 찾기'
date: 2019-09-03 14:05:00 +09:00
categories: 'javascript,algorithms'
published: false
---

```javascript
/**
 * @param {string} s
 * @return {string}
 */
var longestPalindrome = function(s) {
  let maxPalindrome = '';

  s.split('').forEach((chr, idx) => {
    let palindrome = findPalindrome(s, idx, idx, true, chr);

    if (palindrome.length > maxPalindrome.length) {
      maxPalindrome = palindrome;
    }
  });

  return maxPalindrome;
};

function findPalindrome(str, l, r, dir, v) {
  if (l < 0 || r > str.length) return v;

  let isPalindrome = true;
  let subset = str.substring(l, r + 1);
  for (let i = 0; i < subset.length / 2; i++) {
    if (subset[i] !== subset[subset.length - 1 - i]) {
      if (dir === true) {
        return findPalindrome(str, l, r - 1, false, v);
      } else {
        return v;
      }
    }
  }

  if (isPalindrome) {
    return findPalindrome(str, l - 1, r + 1, true, subset);
  }
}
```

- `dir` 변수는, 검색하였을 때, 결과가 발견되지 않을 경우, 롤백 후, 한쪽만 수정한 뒤 확인해보는 방법을 차용하였다.
- 이렇게 함으로써, "caad"와 같이, 짝수로 떨어지는 경우를 처리할 수 있다.
