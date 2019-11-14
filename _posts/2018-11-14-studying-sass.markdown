---
layout: post
title: '1. 공통 라이브러리: CSS 전처리기 SASS'
date: 2018-11-14 01:30:00 +0900
categories: 'styling'
---

## CSS 작업하던 중...

CSS로 작업하는 것은 굉장히 큰 고역이다. 무제한적인 자유를 부여하는 CSS는 마치 모든 것을 할 수 있는 것처럼 보여지지만, 애플리케이션의 크기가 조금만 커져도 굉장히 많은 문제들을 발생시킨다.

무엇보다 정말 화가나는 것은, 개발하다 CSS를 만지기 시작하면 우선 능력 저하 현상이 엄청나게 발생한다. 누군가가 만들어 놓은 코드를 유지보수하거나, 과거에 만들어놓은 CSS 파일들을 보면, 어느 곳에서 내가 수정한 것이 어떻게 다른 부분에 영향을 끼치는지를 파악할 수가 없다.

그렇기 때문에 전처리기를 사용한다. CSS계의 TypeScript라고 해야할까? 더 많은 함수들을 사용하여 이제는 조금 더 Scalable한 애플리케이션을 만들어보자.
'CSS that scales!'

## SCSS: Syntatictally Awesome CSS

## Intro

```
# node-sass를 통하여 컴파일
sudo npm install -g node-sass
```

## Variable

Sass는 기존의 CSS와는 달리, SassScript라는 것을 이용하여 다양한 작업들을 할 수 있다. SassScript는 변수 / 사칙연산 / 그리고 추가 함수 기능들을 기존의 CSS에 추가한다.

첫번째로 변수를 선언하여 재사용할 수 있다. 변수는 `$` 표시를 붙힘으로써 선언할 수 있다.

```css
$width: 5em;

#main {
  width: $width;
}

// 컴파일 된 결과
#main {
  width: 5em;
}
```

변수는 해당 변수가 정의된 네스트된 선택자의 단계 내에서만 사용할 수 있다. 만약 내부에서 선언되었지만, 외부에서도 사용되고 싶다면 해당 변수 뒤에 `!global` 플래그를 붙혀주면 된다. (물론 딱히 쓰는 것이 권장되는 것이 아니다)

- 변수명 같은 경우, 하이픈(-)과 언더스코어(\_)가 상호교환 가능하도록 설계되어있다. `$container-height`와 같은 변수의 경우, `$container_height`로도 쓸 수 있다. (반대도 가능하다)

## Data Types

SassScript는 아래와 같은 8가지 종류의 데이터 타입들을 지원한다.

- numbers: 숫자 (1, 2, 3, 4, 10px...)
- strings of text: 따옴표를 포함하거나 포함하지 않는 글자 ("foo", 'bar', foobar)
  `margin: 10px 15px 0 0` 혹은 `font-face: Helvetica, Arial` 등 n개의 값이 선언되어 있으며 띄어쓰기나 콤마로 구분되어 있다. 한개의 값만 들어가는 경우에도 길이가 1인 list라고 볼 수 있다.
  List는 한 공간에 n개 만큼 넣을 수도 있는데, 예를들어 `1px 2px, 5px 6px`과 같이 `1px 2px`짜리 리스트와 `5px 6px`짜리 리스트를 동시에 사용할 수 있다. 다만 이 경우에는 양옆에 소괄호를 사용하여 시작과 끝을 명시해주는 것이 좋다. `(1px 2px) (5px 6px)`과 같이. (결과물의 차이는 없다)
  List는 아무런 값도 갖지 않을 수 있는데, CSS로 출력은 되지 않는다. `font-family: ()`와 같이 선언할 경우, Sass가 에러를 출력하게 된다. (빈 리스트와 null 값은 CSS로 변환 전에 제거 된다)
  이 외에도 대괄호로도 List를 작성할 수 있고, 콤마를 마지막에만 넣음으로써 값이 하나만 있는 List를 나타낼 수도 있다.
- colors: 색상값 (blue, #ffffff, rgba(255, 255, 255, 0.1))
- booleans: 불린값 (true, false)
- nulls: 널값 (null)
- lists of values, separated by spaces or commas: 띄어쓰기나 콤마로 나뉘어져있는 행렬 (1.5em 1em 0 2em, Helvetica, Arial, sans-serif)
- maps from one value to another: 하나의 값에서 다른 값으로 연결해주는 Map ((key1: value1, key2: value2))
- function references: 함수 참고

## Media

미디어 쿼리는 CSS와 동일하게 작동하지만, CSS 값에 네스팅 될 수 있다는 차이점을 갖는다. 네스팅 되어 안에 들어갈 경우, 컴파일 될 때 최 상단으로 올라오며, 올라오면서 모든 선택자들을 해당 미디어 쿼리 안에 넣는다. 이렇게 함으로써 선택자를 반복하는 방식은 하지 않아도 된다.

```scss
.sidebar {
  width: 300px;
  @media screen and (orientation: landscape) {
    width: 500px;
  }
}
```

```css
.sidebar {
  width: 300px;
}
@media screen and (orientation: landscape) {
  .sidebar {
    width: 500px;
  }
}
```

SassScript의 변수명 역시 사용할 수 있다.

```scss
$media: screen;
$feature: -webkit-min-device-pixel-ratio;
$value: 1.5;

@media #{$media} and ($feature: $value) {
  .sidebar {
    width: 500px;
  }
}
```

```css
@media screen and (-webkit-min-device-pixel-ratio: 1.5) {
  .sidebar {
    width: 500px;
  }
}
```

## Mixin

Mixin은 Stylesheet 내에서 의미가 부여되지 않은 클래스(`.float-left`와 같은)를 사용하지 않고 재사용 가능한 스타일 모음을 만든다. CSS 규칙을 모두 사용할 수 있으며, 인자(argument)를 받아서 사용할 수도 있다.

```scss
@mixin large-text {
  font: {
    family: Arial;
    size: 20px;
    weight: bold;
  }
  color: #ff0000;
}
```

Mixin의 사용은 `@include`를 사용하여 포함시킬 수 있다.

```scss
.page-title {
  @include large-text;
  padding: 4px;
  margin-top: 10px;
}
```

```css
.page-title {
  font-family: Arial;
  font-size: 20px;
  font-weight: bold;
  color: #ff0000;
  padding: 4px;
  margin-top: 10px;
}
```

여기서 다소 신기할 수 있는 부분이, `font` 안에 있는 항목들이 전부 `font-*`로 컴파일 된 것을 확인할 수 있다. 기존에 CSS는 네임스페이스라 불렀는데, 위의 예제는 `font` 네임스페이스 아래에 있는 항목들이었다. 기존의 방식으로는 타이핑 하기 위해서는 전부 타이핑을 해야 했으나, Sass 문법에서는 Nested Properties 속성을 통해 네스팅을 해줌으로써 작업량을 훨씬 줄일 수 있다.

## Function

물론, function도 사용할 수 있다. function은 mixin과 굉장히 유사하게 동작하고, **반드시 `@return` 을 통해 값을 반환해야한다.**

```scss
$grid-width: 40px;
$gutter-width: 10px;

@function grid-width($n) {
  @return $n * $grid-width + ($n - 1) * $gutter-width;
}

#sidebar {
  width: grid-width(5);
}
```
