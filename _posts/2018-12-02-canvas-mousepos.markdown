---
layout: post
title: 'Canvas Mouse Position'
date: 2018-12-02 16:10:00 +09:00
categories: 'canvas'
---

## Glossary

- document.documentElement.scrollLeft / scrollTop
- element.offsetWidth / offsetHeight
- event.clientX / clientY

### document documentElement

[MDN documentElement][reference-01]
document 아래에 있는 루트 요소를 얻기 위한 요소이다. 일반적인 HTML 문서라면, 가장 최상단 자식 노드인 `<html>` 항목의 요소를 반환한다.

### HTMLElement offsetWidth / offsetHeight

`HTMLElement.offsetWidth`는 HTML 요소의 레이아웃 너비를 integer 단위로 반환하는 읽기 전용 속성이다.
일반적으로, `offsetWidth`는 border, padding, 그리고 렌더되었을 경우 vertical scrollbar를 포함한 요소의 CSS 너비값을 나타낸다. 가짜 요소인 `::before`나 `::after`는 포함하지 않는다. (margin도 포함되지 않는다)
해당하는 요소가 숨김 상태로 되어 있을 경우, 0값이 반환된다.

### MouseEvent clientX / clientY

[Difference between coordinate values][reference-02]
`MouseEvent.clientX`는 `MouseEvent` 인터페이스의 읽기 전용 속성으로써, 이벤트가 발생한 애플리케이션의 클라이언트 공간의 가로 값을 제공한다. (clientY는 세로 값). 예를들어, 클라이언트 공간의 왼쪽 상단을 클릭할 경우 `clientX`는 스크롤 여부와 상관없이 항상 0 값을 반환할 것이다. 기존에는 `long` integer로 설계되었었는데, 현재는 `double` float 값으로 나타난다.

[reference-01]: https://developer.mozilla.org/ko/docs/Web/API/Document/documentElement
[reference-02]: https://stackoverflow.com/questions/6073505/what-is-the-difference-between-screenx-y-clientx-y-and-pagex-y
