---
layout: post
title:  "직접 Virtual DOM 작성해보기"
date:   2019-08-01 15:50:00 +09:00
categories: "javascript,react"
published: true
---

## Reference
[https://medium.com/@deathmood/how-to-write-your-own-virtual-dom-ee74acc13060](How to write your own Virtual DOM)


## Virtual DOM이란?
React 공식 사이트에서는 Virtual DOM (혹은 가상DOM)을 아래와 같이 규정한다.
> Virtual DOM, 혹은 VDOM은 UI의 상징적인 (혹은 가상의) 항목이 메모리에 저장되어 실제 DOM과 동기화되는 프로그래밍적 개념을 뜻한다. 이에 사용되는 라이브러리 중에는 ReactDOM이 있다. 이 과정은 재조정(reconciliation) 과정이라 불린다.

Virtual DOM은 실제 DOM의 형태 중 하나로 생각하면 될듯하다.

## DOM 트리를 메모리에 저장하기
Virtual DOM의 작동 원리는 메모리에 저장되어 실제 DOM과 동기화되는 방식이다. 예를들어 아래와 같은 DOM 트리가 있다고 가정하자.
```html
<ul class="tree">
  <li class="leaf">list 1</li>
  <li class="leaf">list 2</li>
  <li class="leaf">list 3</li>
</ul>
```

위와 같은 DOM 트리는 자바스크립트 객체로 나타낼 수 있다.
```javascript
{ type: 'ul', props: { 'class': 'tree' }, children: [
  { type: 'li', props: { 'class': 'leaf' }, children: ['list 1'] },
  { type: 'li', props: { 'class': 'leaf' }, children: ['list 2'] },
  { type: 'li', props: { 'class': 'leaf' }, children: ['list 3'] },
]}
```

### 그러면 React에서는 

