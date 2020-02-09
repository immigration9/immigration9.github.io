---
layout: post
title: 'Learning React Native'
date: 2020-01-27 13:15:00 +09:00
categories: 'reactnative'
published: false
---

## Components to Remember

## Platform Specific

1. Platform Module을 import하여 사용한다.

```javascript
import { Platform } from 'react-native';

const isPlatformIOS = Platform.OS === 'ios';
```

iOS의 경우 `ios`가 반환되고, Android의 경우 `android`가 반환된다.

아예 내부적으로 선택지를 key / value값으로 가져가고 싶다면, `Platform.select`를 사용하면 된다.

```javascript
import { Platform, StyleSheet } from 'react-native';

const viewStyle = StyleSheet.create({
  container: {
    flex: 1,
    ...Platform.select({
      ios: {
        backgroundColor: 'red'
      },
      android: {
        backgroundColor: 'blue'
      }
    })
  }
});
```

놀랍게도 아무거나 다 가능하기 때문에, Platform에 해당하는 컴포넌트도 불러올 수 있다. (아래 코드는 ES6형식으로 고칠 수 있을 것 같다)

```javascript
const Component = Platform.select({
  ios: () => require('ComponentIOS'),
  android: () => require('ComponentAndroid')
})();

<Component />;
```

2. 특정 플랫폼에 해당하는 확장자 사용하기

```
BigButton.ios.js
BigButton.android.js
```

## Navigation

`react-navigator`를 이용한 스크린 transition
