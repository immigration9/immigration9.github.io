---
layout: post
title: 'Studying D3JS'
date: 2019-06-16 17:00:00 +09:00
categories: 'javascript'
published: false
---


## D3js 개요
* D3js (이하 D3)는 셀렉션과 바인딩이다.
1. 셀렉션: 선택된 묶음을 단위로 이동, 색상 변경, 데이터값 변경 등을 수행

```javascript
d3.selectAll("circle.a")
    .style("fill", "red")
    .attr("cx", 100);
```
해석: (1) a 클래스에 속한 원을 모두 선택해 (2) 빨간색으로 채우고, (3) 원의 중점을 `<svg>` 영역의 왼쪽에서 100px 떨어진 곳에 위치시킨다.

2. 데이터 바인딩: 데이터를 요소에 연결하는 것

```javascript
d3.selectAll("div.market").data([1,5,11,3])
```
해석: market 클래스에 속한 모든 항목들을 선택해 데이터를 바인딩 시킨다.

* Style / Attribute / Property는 각 요소를 특정짓는다.
Style: CSS style
Attribute: id명 설정 / SVG 요소의 경우 위치, 크기 등
Property: Checkbox의 경우 체크 유무

### SVG 요소
1. 모든 그림을 넣을 수 있다.
2. 왼쪽 꼭대기가 0,0 이다 (Canvas와 동일)
3. CSS로 테두리와 배경을 다르게 설정 가능
4. viewBox 속성을 통해 동적으로 크기 변경 가능

* `<circle>, <recr>, <line>, <polygon>, <ellipse>` 요소
* `<text>` 요소: 포멧팅을 할 경우 요소 안에 `<tspan>` 요소를 넣어 구현
* `<g>` 요소: 논리적 그룹핑 -> 영역 안에서 요소를 움직이려면 `<g>` 요소의 `transform` 속성만 조정하면 된다.
* `<d>` 요소: d 속성이 결정하는 도형의 영역

### CSS 사용
```css
.active {
  stroke: black;
  stroke-width: 4px;
  stroke-dasharray: 1;
}
circle {
  fill: red;
}
```

### Javascript 사용
* 메서드 체이닝: 연속적으로 메서드 호출 + 각 메서드 실행 후 객체 자신을 반환해야함.


## 데이터 정보화

### 정규화: 스케일과 규모 변경
`d3.scale()`을 통해 수치형 데이터가 화면에 표기될 수 있도록 정규화

수치형 값을 색상 범위로 매핑시
```javascript
var newRange = d3.scale.linear().domain([50000, 100000]).range(["blue", "red"]);
newRange(100000) // red color를 리턴한다.
newRange(60000) // blue color에 가까운 값이 리턴된다.
```

`d3.scale.quantile()` 사용시 변윗값을 사용할 수도 있다.


### 최소 / 최댓값
```javascript
d3.min() // 최소값
d3.max() // 최대값
d3.mean() // 평균값
d3.extent() // 최소값 - 최대값 동시에 반환

d3.csv("cities.csv", function(data) {
  d3.min(data, function(el) { return +el.population});
});
```

셀렉션과 바인딩 동시에
```javascript
d3.csv("cities.csv", function(error, data) { dataViz(data); });

function dataViz(incomingData) {
  d3.select("body").selectAll("div.cities")
    .data(incomingData)
    .enter()
    .append("div")
    .attr("class", "cities")
    .html(function(d, i) { return d.label; });
}
```

### 요소들
`d3.select() / d3.selectAll()`: DOM element selector
`data()`: 선택한 DOM Element와 Data 배열을 연결시킨다.
`enter() / exit()`: 데이터 바인딩시 데이터 개수가 DOM 요소보다 많을 경우 enter로 처리 / 적을 경우 exit로 처리
`append() / insert()`: 요소 추가시 사용, insert는 추가할 위치 지정 가능
`attr()`: class, id 등 DOM element의 속성 변경시 사용
* 주의: select로 특정 class를 지정했다고 하더라도, append로 추가 생성된 분에 대해서는 attr을 이용하여 별도로 지정해줘야 한다.
`html()`: 전통적인 DOM element 콘텐츠 접근

### 이외
`d3.nest()`: 데이터 배열을 key로 구분하여 정리할 수 있다.

```javascript
/**
 * 위에 incomingData로 데이터를 받았다고 가정한다. incomingData는 배열로, 각각 user 항목을 가지고 있다.
 * 아래와 같이 정리하면, nestedTweets는 유저 별로 데이터 값이 정리된다.
 */
var nestedTweets = d3.nest()
                      .key(function (el) { return el.user; })
                      .entries(incomingData);
```