---
layout: post
title: 'Typescript Handbook'
date: 2019-10-13 20:30:00 +09:00
categories: 'typescript'
published: true
---

## Type Alias와 Interface의 차이

type과 interface는 굉장히 유사하며, 많은 경우에 서로를 교차해서 사용할 수 있다. 물론, 그 둘에는 아래와 같은 차이점이 있다. (이하 type은 type alias를 나타내기 위해 사용하며, Typescript의 type은 타입으로 표기하도록 한다).

- interface는 `extend`될 수 있지만, type은 그럴 수 없다. 이를 통해, interface는 하나의 타입에서 새로운 타입을 만드는데 더 용이하다고 볼 수 있다.
- type은 선언 병합(declaration merging)을 할 수 없다. interface는 가능하다.
- interface는 객체 타입 선언에만 사용될 수 있다.

* 이와 더불어 아래 내용은 확인이 필요하다.

- interface는 에러가 발생시 에러 메시지에 원래의 형태로 나타나지만, type은 그렇지 않을 수 있다.
