---
layout: post
title:  "1. 공통 라이브러리: CSS 전처리기 SASS"
date:   2018-11-14 01:30:00 +0900
categories: "styling"
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
  width: 5em; }
```

변수는 해당 변수가 정의된 네스트된 선택자의 단계 내에서만 사용할 수 있다. 만약 내부에서 선언되었지만, 외부에서도 사용되고 싶다면 해당 변수 뒤에 `!global` 플래그를 붙혀주면 된다. (물론 딱히 쓰는 것이 권장되는 것이 아니다)

* 변수명 같은 경우, 하이픈(-)과 언더스코어(_)가 상호교환 가능하도록 설계되어있다. `$container-height`와 같은 변수의 경우, `$container_height`로도 쓸 수 있다. (반대도 가능하다)

## Data Types
SassScript는 아래와 같은 8가지 종류의 데이터 타입들을 지원한다.
**numbers: 숫자 (1, 2, 3, 4, 10px...)**

**strings of text: 따옴표를 포함하거나 포함하지 않는 글자 ("foo", 'bar', foobar)**
`margin: 10px 15px 0 0` 혹은 `font-face: Helvetica, Arial` 등 n개의 값이 선언되어 있으며 띄어쓰기나 콤마로 구분되어 있다. 한개의 값만 들어가는 경우에도 길이가 1인 list라고 볼 수 있다.
List는 한 공간에 n개 만큼 넣을 수도 있는데, 예를들어 `1px 2px, 5px 6px`과 같이 `1px 2px`짜리 리스트와 `5px 6px`짜리 리스트를 동시에 사용할 수 있다. 다만 이 경우에는 양옆에 소괄호를 사용하여 시작과 끝을 명시해주는 것이 좋다. `(1px 2px) (5px 6px)`과 같이. (결과물의 차이는 없다)
List는 아무런 값도 갖지 않을 수 있는데, CSS로 출력은 되지 않는다. `font-family: ()`와 같이 선언할 경우, Sass가 에러를 출력하게 된다. (빈 리스트와 null 값은 CSS로 변환 전에 제거 된다)
이 외에도 대괄호로도 List를 작성할 수 있고, 콤마를 마지막에만 넣음으로써 값이 하나만 있는 List를 나타낼 수도 있다.

**colors: 색상값 (blue, #ffffff, rgba(255, 255, 255, 0.1))**

**booleans: 불린값 (true, false)**

**nulls: 널값 (null)**

**lists of values, separated by spaces or commas: 띄어쓰기나 콤마로 나뉘어져있는 행렬 (1.5em 1em 0 2em, Helvetica, Arial, sans-serif)**

**maps from one value to another: 하나의 값에서 다른 값으로 연결해주는 Map ((key1: value1, key2: value2))**

**function references: 함수 참고**
