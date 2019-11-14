---
layout: post
title: 'Pseudo-Class / Attribute Selector'
date: 2019-02-01 01:00:00 +09:00
categories: 'sass'
published: true
---

## :not pseudo-class

```scss
.row {
  max-width: 114rem;
  background-color: #eee;
  // Center block element
  margin: 0 auto;

  &:not(:last-child) {
    margin-bottom: 8rem;
  }
```

`not` pseudo-class를 통해, () 안에 있는 class를 제외한 나머지에 아래 내용을 적용할 수가 있다.

## calc()

`.col-*-of-*`의 경우, 전체 `.row`의 길이 중에 계산을 해야한다. 일반적으로 `calc()`를 사용하면 되는데, SCSS의 경우, `calc()` 내부에서 변수를 사용할 경우 앞에 `#{}`를 붙혀야한다.

```scss
$gutter-horizontal = 8rem;

.col-1-of-2 {
  width: calc((100% - #{$gutter-horizontal}) / 2);
  background-color: red;
  float: left;
}
```

## mixin 적용

완료된 뒤, 확인해보면 `float`로 인하여 height값이 0으로 계산되어 있는 것을 확인할 수 있다. 이러한 버그를 해결하기 위해, Clearfix를 넣어주면 되는데, 자주 사용하는 항목이니 `mixin`을 사용하도록 하자.

```scss
@mixin clearfix {
  &::after {
    content: '';
    display: table;
    clear: both;
  }
}
```

포함은 중간에 넣어주면 된다.

```scss
.row {
  ...

  @include clearfix;
}
```

## Attribute selector

먼저 Column을 만들어보자.

```scss
.col-1-of-2 {
  width: calc((100% - #{$gutter-horizontal}) / 2);
  background-color: orangered;
  float: left;

  &:not(:last-child) {
    margin-right: $gutter-horizontal;
  }
}
```

위와 같은 방식으로 1/2에 해당하는 Column을 생성하였다.
하지만, 동일한 방식으로 Column을 여러개 만든다면 굳이 `background-color` 밑에 항목들은 반복해서 작성해야 하는 것일까?

그런 경우를 위해 Attribute Selector를 사용할 수 있다.

```scss
[class^='col-'] {
  background-color: orangered;
  float: left;

  &:not(:last-child) {
    margin-right: $gutter-horizontal;
  }
}
```

말그대로, 특정 속성에 대하여 검색하여 일치하는 결과들에 동일한 값을 적용할 수 있다. 위에 써있는 `class^="col-"`은 `col-`로 시작하는 항목을, `class*="col-"`이라고 작성하면 `col-`을 포함하는 항목을 뜻한다. 마지막으로, `$=`으로 쓰면 끝나는 경우를 뜻하는데, 여기서는 가장 적당한 윗 예제로 작성한다. 이제 나머지는 아래와 같이 계산만 해주면 된다.

```scss
.col-1-of-2 {
  width: calc((100% - #{$gutter-horizontal}) / 2);
}

.col-1-of-3 {
  width: calc((100% - 2 * #{$gutter-horizontal}) / 3);
}
```
