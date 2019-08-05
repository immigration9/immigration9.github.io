### Column Drop Pattern

```html
<div class="container">
  <div class="box dark_blue"></div>
  <div class="box light_blue"></div>
  <div class="box green"></div>
</div>
```

```css
/**
 * `flex-wrap`: 항목들이 여러줄에 배치되게 할 것인지 결정. nowrap 옵션이 default (한줄에 배치)
 */
.container {
  display: flex;
  flex-wrap: wrap;
}
.box {
  width: 100%;
}

@media screen and (min-width: 450px) {
  .dark_blue {
    width: 25%;
  }
  .light_blue {
    width: 75%;
  }
}

@media screen and (min-width: 550px) {
  .dark_blue, .green {
    width: 25%;
  }
  .light_blue {
    width: 50%;
  }
}
```

### Mostly Fluid Pattern

```html
<div class="container">
  <div class="box dark_blue"></div>
  <div class="box light_blue"></div>
  <div class="box green"></div>
  <div class="box red"></div>
  <div class="box orange"></div>
</div>
```


```css
.container {
  display: flex;
  flex-wrap: wrap;
}
.box {
  width: 100%;
}

@media screen and (min-width: 450px) {
  .light_blue, .green {
    width: 50%;
  }
}

@media screen and (min-width: 550px) {
  .dark_blue, .light_blue {
    width: 50%;
  }
  .green, .red, .orange {
    width: 33.333333%;
  }
}

/**
 * 최대 700px을 넘어가지 않도록 구성
 */
@media screen and (min-width: 700px) {
  .containre {
    width: 700px;
    margin-left: auto;
    margin-right: auto;
  }
}
```

### Layout Shifter Pattern

```html
<div class="container">
  <div class="box dark_blue"></div>
  <div class="container" id="container2">
    <div class="box light_blue"></div>
    <div class="box green"></div>
  </div>
  <div class="box red"></div>
</div>
```

```css
.container {
  width: 100%;
  display: flex;
  flex-wrap: wrap;
}

.box {
  width: 100%;
}

@media screen and (min-width: 500px) {
  .dark_blue {
    width: 50%;
  }
  #container2 {
    width: 50%;
  }
}

@media screen and (min-width: 600px) {
  .dark_blue {
    width: 25%;
    order: 1; /** shows up last */
  }
  #container2 {
    width: 50%;
  }
  .red {
    width: 25%;
    order: -1; /** shows up first */
  }
}
```

### Off Canvas
