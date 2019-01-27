---
layout: post
title:  "CSS How it works"
date:   2019-01-27 17:00:00 +09:00
categories: "css"
published: true
---

## CSS의 작동 방식
HTML 로드 -> HTML 파싱 -> Document Object Model (DOM)
              -> CSS 로드 -> CSS 파싱 -> CSS Object Model (CSSOM)
두 단계가 합쳐져서 Render Tree -> Website Rendering (Visual Formatting Model) -> 최종 렌더링

### CSS 파싱
CSS는 다음과 같이 구성되어 있다.

```css
.some-class {
  color: black;
  text-align: center;
}
```

`.some-class`: Selector
`{}`: Declaration Block`
`text-align: center`: Declaration
`text-align`: Property
`center`: Declared value

* 파싱 과정
1. 캐스캐이딩(Cascade): 여러개의 Stylesheets 들을 합치고, 서로 다른 CSS 규칙들과 선언문이 중복될 때 conflict를 해결하는 과정.
- Declaration의 종류: Author(개발자 작성) / User(사용자 수정) / Browser (User agent, 브라우저 기본값)

* 중요도 선정 기준 (높을 수록 우선권)
1. 사용자가 정의한 `!important` 선언
2. 개발자가 정의한 `!important` 선언
3. 개발자가 정의한 선언
4. 사용자가 정의한 선언
5. 기본 브라우저 선언

* 동일한 중요도를 가질 경우 선정 기준 (높을 수록 우선권)
1. Inline styles
2. IDs
3. Classes, pseudo-classes, attribute
4. Elements, pseudo-elements (like div, nav)

(Inline / IDs / Classes / Elements) 이 과정을 왼쪽에서 오른쪽으로 숫자를 비교한다.

* 여기서 중요한점은, pseudo-class 역시 class와 동일하게 취급되기 때문에, 코드를 쓰다보면 갑자기 `:hover`와 같은 경우가 적용되지 않을 수 있다. 이것은 중요도가 낮기 때문이다.

* 결론적으로, ID를 한개 가지고 있는 Selector가 1000개 이상의 Class를 가지고 있는 Selector보다 높은 중요도를 갖는다.

만약에 모든 중요도와 선정 기준이 같을 경우, 마지막에 작성된 것이 반영된다.

왠만하면 `!important`는 쓰지 말자. maintainability를 해친다.

