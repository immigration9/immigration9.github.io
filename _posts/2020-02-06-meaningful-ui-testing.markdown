---
layout: post
title: '의미있는 UI Testing 환경 구축기'
date: 2020-02-06 23:15:00 +09:00
categories: 'react,testing'
published: false
---

## 동기

프론트엔드에서 무언가를 테스팅하는 것은 참으로 어려운 일이다. 서버 사이드에 비해 기술적인 한계점도 많이 존재하지만, 동시에 어려운 점은 레퍼런스가 절대적으로 적다는 것이다. 물론, 오픈소스나 UI 라이브러리를 찾아보면 거의 모든 프로젝트들이 테스트 코드를 메인테이닝 하고 있다. 하지만 우리가 여기서 얘기하는 것은 오픈소스 프로젝트가 아니다. 실제 프로덕트로 나가는 코드에 추가하는 테스트 코드다.

솔직하게 밝히자면, UI Testing을 도입하기 위한 도전은 이번이 처음이 아니었다. 프론트엔드 개발자라면 누구나 최소 살면서 두 번 이상은 실패한다는 UI Testing 도입은 작성자 본인도 여러번의 실패와 시행착오를 거쳐왔다.

결론부터 말하자면 선정된 라이브러리들은 다음과 같다. 그러니 만약 글을 읽고 싶지는 않고, 대신 정보는 가져가고 싶다면, 아래 추천 라이브러리 목록을 설치한 다음 작성하기 시작하면 된다.

```
RosieJS - JS 객체 생성 팩토리 라이브러리
axios-mock-adapter
Jest
@testing-library/react
Faker
```

## 의미 부여하기

테스트 코드를 쓰는 것은 그것이 업무에 차지하는 비중이 적다고 하더라도 일부를 담당하게 된다. 결론적으로 테스트 코드를 작성할 때는 아래와 같은 요소들을 고려해야 한다.

1. 작성하는데 지나치게 많은 시간을 필요로 하지는 않는가?
2. 시간이 지나서 걸림돌이 되는 현상을 방지할 수 있는가?
3. 제품의 완성도에 긍정적인 영향력을 가질 수 있는가?

1, 2, 3번 모두 당연하지만 다소 놓치기 쉬운 항목들이라는 점을 발견할 수 있었다. UI 개발을 시작하면, 기존에 존재하던 테스트코드가 굉장히 필요 없고, 오히려 개발에 걸림돌이 되는 경우들이 있다. 분명 그 당시에는 프로덕션에 도움을 주기 위해 도입이 되었을텐데, 왜 그런 것일까?

결국 테스트도 코드의 일부다. 코드의 발전에

<!-- @todo -->

[Write tests. Not too many. Mostly integration.](https://kentcdodds.com/blog/write-tests)

## Jest와 Testing Library를 통한 테스트 코드 작성

[]()
Testing Library는

## RosieJS + Faker를 통해 API 결과값 생성하기

테스트 도입시에 가장 큰 어려움 중 하나는 무엇보다 Mock 데이터를 생성하는 것이었다. 초기에는 이 항목을 자동화하기 위해 데이터 생성 함수도 별도로 작성해보고 그랬지만, 프로젝트 규모가 작지 않고, 업데이트가 주기적으로 되다보니 두 가지 문제점에 봉착하게 되었다.

- API가 변경되었을 때 유기적으로 대응하기 어렵다
- 서로 연관관계가 있는 데이터를 생성하기가 어렵다

특히 유기적인 대응이 어렵다는 점이 굉장히 큰 이슈였는데, 아무래도 제품이 초기단계다 보니 새로운 업데이트마다 굉장히 많은 내용이 변경되었고, 이로인해 테스트코드가 금방 죽은 코드가 되어버리는 문제들이 있었다. 다행히도 사내 서버 개발자 분들이 루비 + 테스트코드 전문가 분들이다 보니, 좋은 라이브러리를 추천 받아 사용할 수 있었다.

RosieJS는 factory_girl (현재 factory_bot)에서 영감을 얻은 라이브러리로, 자바스크립트 객체 생성 팩토리 역할을 한다.
[RosieJS](https://github.com/rosiejs/rosie)

Factory를 define 하는 것의 가장 큰 이점은, 무엇보다 의존성을 갖는 attribute나 sequence 처리를 자동으로 해결할 수 있다는 점이다. Test data 생성시에 가장 큰 어려움 중 하나는 데이터를 만드는 것도 까다롭지만, 향후 API 스펙이 변경되거나 리팩토링이 필요할 때 변경 사항에 유기적으로 대응하기 어렵다는 점이다. 공식 Document에 나와있는 가장 기본적인 RosieJS의 예제를 한 번 살펴보자.

```javascript
Factory.define('game')
  .sequence('id')
  .attr('is_over', false)
  .attr('created_at', () => new Date())
  .attr('random_seed', () => Math.random())

  // Default to two players. If players were given, fill in
  // whatever attributes might be missing.
  .attr('players', ['players'], players => {
    if (!players) {
      players = [{}, {}];
    }
    return players.map(data => Factory.attributes('player', data));
  });

Factory.define('player')
  .sequence('id')
  .sequence('name', function(i) {
    return 'player' + i;
  })

  // Define `position` to depend on `id`.
  .attr('position', ['id'], id => {
    const positions = ['pitcher', '1st base', '2nd base', '3rd base'];
    return positions[id % positions.length];
  });
```

`define` 뒤에 호출되는 `attr` / `sequence`의 경우 생성될 객체의 내부에 속성을 정의해준다.
`players` 속성은 중간에 `players`가 전달되는 배열이 존재하는데, 이는 참고할 속성 명단을 전달한다.
만약 여기서 다른 속성을 참고하고 싶다면 아래와 같이 추가해주면 된다.

```javascript
Factory.define('game')
  .sequence('id')
  .attr('players', ['id', 'players'], (id, players) => {
    // id와 players를 모두 참고
  });
```

위 예제에 따라 game 객체를 만들면, 사전에 정의되어 있던 속성들이 들어가게 된다.

```javascript
const game = Factory.build('game', { is_over: true });
// Built object (note scores are random):
//{
//    id:           1,
//    is_over:      true,   // overriden when building
//    created_at:   Fri Apr 15 2011 12:02:25 GMT-0400 (EDT),
//    random_seed:  0.8999513240996748,
//    players: [
//                {id: 1, name:'Player 1'},
//                {id: 2, name:'Player 2'}
//    ]
//}
```

전달 받는 데이터는 faker.js 라이브러리를 사용하였다. 워낙 유명한 라이브러리다보니 별도로 설명이 필요하진 않을 것 같다.
[faker.js](https://github.com/marak/Faker.js/)
라이브러리 사용에 있어 이 부분은 사용하는 항목들을 공통화 시켜 별도의 파일에 빼놓고 필요할 때 호출하는 방식을 사용하였다.

```javascript
export const randomDate = (pastMonth: number = 1) => {
  return faker.date.past(pastMonth).toISOString();
};

export const getCount = (max: number, isFixed: boolean = false) => {
  return isFixed ? max : randomNumber(max);
};

export const randomNumber = (max?: number) => {
  const randNum = faker.random.number(max);
  return randNum > 0 ? randNum : 1;
};

export const randomRange = (min: number, max: number) => {
  return faker.random.number({ min, max });
};

export const randomWords = (count: number = 1) => {
  return count === 1 ? faker.lorem.word() : faker.lorem.words(count);
};

export const randomSentence = (count: number = 1) => {
  return count === 1 ? faker.lorem.sentence() : faker.lorem.sentences(count);
};
```

위의 두 가지를 조합하였을 때, `post`와 `comment`에 대한 Factory를 정의한다고 가정하면 아래와 같이 생성할 수 있다.

```javascript
Factory.define('post')
  .sequence('id')
  .attr('title', () => randomSentence())
  .attr('content', () => randomSentence(3))
  .attr('comments', ['id'], id => {
    return Factory.buildList('comment', randomNumber(5), { postId: id });
  });

Factory.define('comment')
  .sequence('id')
  .attr('content', () => randomSentence(3))
  .attr('postId', 0);
```

정의된 Factory를 기준으로 `post`를 생성하면, 자체적으로 `post`의 id값을 참고하는 `comment`들을 여러개 생성할 수 있다.

## axios-mock-adapter를 통해 API 호출 해결하기
