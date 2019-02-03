

## max-width (desktop first) / min-width (mobile first)
media query는 selector에 중요도를 지정하지 않기 때문에 코드의 순서가 중요하다. 주로 마지막에 쓴다.
예를들어 아래와 같이 작동한다.

```scss
@media (max-width: 600px) {
  font-size: 30%;
}

@media (max-width: 900px) {
  font-size: 60%;
}

@media (max-width: 1200px) {
  font-size: 90%;
}
```

media query는 순서대로 읽히기 때문에, 이 과정에서 제일 처음 읽히는 것은 600px부터 읽히게 된다. 문제는, 작은 화면의 경우 (ex. 500px), 600px, 900px, 1200px에 모두 해당되기 때문에, 결론적으로 1200px에 해당하는 `font-size: 90%`를 갖게 된다. 그렇기 때문에 방향을 반대로 가져가야 한다.
```scss

@media (max-width: 1200px) {
  font-size: 90%;
}

@media (max-width: 900px) {
  font-size: 60%;
}
@media (max-width: 600px) {
  font-size: 30%;
}
```
이렇게 될 경우, 순서대로 봤을 때 가장 마지막 media query가 실행되기 때문에 정상적으로 실행된다.


## mixin을 사용하여 media query 사용
mixin을 사용하면, media query를 좀 더 손쉽게 사용할 수 있다. `content-selector`를 사용하는 방식인데, `@content`로 지정을 해두면, 해당 mixin을 사용할 때 안에 내용을 별도로 채워줄 수 있다.

우선 mixin에 아래와 같이 정의한다.
```scss
@mixin respond-phone {
  @media (max-width: 600px) { @content };
}
```

정의된 mixin을 이제 가져와서 사용하면 된다.
```scss
@include respond-phone {
  font-size: 50%;
}
```
해당 위에 나와 있는 `font-size: 50%;`가 `@content`에 삽입되어, 향후에 media query 사용시에 불필요하게 `@media`를 선언할 필요가 없게 된다.


## $breakpoint와 @if를 사용하여 좀 더 편하게 media query 사용
mixin에는 argument를 전달할 수 있다. 이 때 전달되는 `$breakpoint`와 SCSS의 `@if` 문법을 조합하면, 동일한 코드를 반복하지 않고 반응형을 구현할 수 있다.

```scss
@mixin respond($breakpoint) {
  @if $breakpoint == phone {
    @media (max-width: 37.5em) { @content }; // 600px
  }
  @if $breakpoint == tab-port {
    @media (max-width: 56.25em) { @content };
  }
  @if $breakpoint == tab-land {
    @media (max-width: 75em) { @content };
  }
  @if $breakpoint == tab-destop {
    @media (min-width: 112.5em) { @content };
  }
}
```

mixin 항목을 살펴보면, `$breakpoint` argument를 전달 받은 뒤, `@if`로 분기하고 있다.
이 때, 분기된 항목은 비교를 하여 값을 할당하게 된다.

```scss

html {
  font-size: 62.5%; // 1rem = 10px

  /** always use the larger one first **/
  @include respond(tab-land) {
    font-size: 56.25%; // 1rem = 9px, 9 / 16 = 50%
  }
  @include respond(tab-port) {
    font-size: 50%; // 1rem = 8px, 8 / 16 = 50%
  }
  @include respond(phone) {
    font-size: 50%;
  }
  @include respond(big-desktop) {
    font-size: 75%; // 1rem = 12px, 12 / 16
  }
}
```

이 방식대로 하면, 반응형에 들어가는 공수를 최소화할 수 있으며, 변경사항이 있을 때 mixin 하나만 바꾸면 전체에 공통적으로 적용할 수 있는 편리함이 있다.