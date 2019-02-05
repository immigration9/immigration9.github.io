---
layout: post
title:  "Font Sizing: Pixel & REM"
date:   2019-02-05 08:00:00 +09:00
categories: "css"
published: true
---


## 웹사이트 Font Size는 어떻게 결정해야할까?
모든 경우에 폰트 크기를 정하는 것은 어려울 뿐만 아니라, 문제점도 다량으로 수반할 수 있다. 첫째로는 픽셀이 매번 Pixel 절대값으로 들어가기 때문에 코드 수정시 매우 번거로울 수 있고, 둘째로는 Pixel값은 브라우저에서 설정 가능한 접근성 등의 변화로 제공되는 폰트 크기를 임의로 오버라이딩하기 때문에, 유저에게도 불편한 UX를 제공하게 될 수 있다.

이러한 문제를 해결하기 위해 다른 DOM element들이 별도의 Pixel값이 아닌, 폰트 크기를 기반으로 계산되도록 할 수 있다. 바로 `rem` 유닛을 사용하는 것이다. 

[CSS Length]:https://developer.mozilla.org/en-US/docs/Web/CSS/length

놀랍게도, CSS에서 길이값을 설정하는 지표들은 굉장히 많다. 다만 여기서는 `rem` 을 기준으로 설명한다.

```
rem

Represents the font-size of the root element (typically <html>). When used within the root element font-size, it represents its initial value (a common browser default is 16px, but user-defined preferences may modify this).
```

일반적으로 `<html>`에 해당되는 최상단 element의 font-size를 기준으로 계산된다. 일반적인 브라우저에서는 16px값이 기본값으로 지정되어 있다.


## 실사용
`rem` 값은 아래와 같이 `<html>` 태그 아래에 배치하는 것이 권장된다.

```scss
html {
  font-size: 10px;
}
```

여기서 10px로 지정한 이유는, 향후에 계산하기에 용이하기 때문이다. 이제 다른 요소들을 선언할 때 rem을 이용하여 크기에 일관성을 가져갈 수 있다.

```scss
body {
  font-family: "Lato", sans-serif;
  font-weight: 400;
  font-size: 1.2rem;
  line-height: 1.7;
  color: $color-grey-dark;
  padding: 3rem;
}
```
이제 `<body>` 안에 있는 child element들은 폰트 크기 12px과 padding 30px을 가지게 된다.
이 방법의 갖아 큰 장점은 무엇보다, 향후 코드 수정시에 가장 상단에 있는 `font-size`만 수정해주면 된다는 점이다.


## 좀 더 나은 방법
Pixel로 선언해도 충분해 보일 수 있지만, 이는 가장 큰 단점 중 하나를 해결하지 못한다.
바로, 사용자가 사용하고자 하는 다른 Pixel 사이즈를 임의로 오버라이딩 해버린다는 점이다. 이는 접근성 항목을 심각히 해친다는 문제를 가지고 있다. (즉, 사용자가 16px로 보겠다고 해도, 10px로 나오는 치명적인 문제점이 있다)

이와 같은 문제를 해결하기 위해서는 아래와 같이 Pixel 단위가 아닌, % 단위로 표현을 해준다.

```scss
html {
  font-size: 62.5%; // 1rem = 10px
}
```
사실상 위와 같은 방식이며, 기본 결과값은 동일한 10px이 된다. 다만, 차이점이 있다면 유저가 만약 브라우더의 기본값인 16px이 아닌, 18px과 같은 커스텀 사이즈를 사용한다면, 10px이 아닌 18px * 62.5%인, 11.25px을 갖게된다.


## 반응형 적용
마지막으로 font-size를 반응형으로 편리하게 적용해보자.

```scss
html {
  font-size: 62.5%;

  @media (max-width: 75em) {
    font-size: 56.25%;  // 1rem = 9px;
  }
  @media (max-width: 56.25em) {
    font-size: 50%;     // 1rem = 8px;
  }
  @media (max-width: 37.5em) {
    font-size: 37.5%;   // 1rem = 6px;
  }
  @media (min-width: 112.5em) {
    font-size: 75%;     // 1rem = 12px;
  }
}
```

* media-query에 `em` 유닛을 사용한 이유는 아래 링크로 대체한다. 브라우저 테스트상에서 `em` 유닛이 가장 일관성 있는 결과물을 나타냈다.
[PX, EM or REM Media Queries?]:https://zellwk.com/blog/media-query-units/


## rem 유닛 사용의 단점
사소하지만 중요한 부분에서 짚고 넘어가야하는 단점들이 두 가지 있다. CSS 스타일링을 처음 접할 때는 전혀 이슈가 되지 않지만, 전문적인 부분으로 파고 들어갈 때 문제가 생길 수 있음으로 남겨놓는다.

1. IE9 미만에서는 지원이 되지 않는다.
어차피 IE9 미만에서는 React도 동작하지 않기 때문에 큰 의미는 없다. 이제 놓아주자.

2. media query 사용시 버그가 있다.
아래 링크로 대체한다.
[EM vs REM vs PX – Why you shouldn't “just use pixels”]:https://engageinteractive.co.uk/blog/em-vs-rem-vs-px